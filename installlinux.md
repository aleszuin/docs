<br>
# Install Emoncms on Ubuntu / Debian Linux

This guide should work on most debian systems including Ubuntu. For installation guide on installing emoncms on a raspberrypi see [raspberrypi from ready-to-go image](http://emoncms.org/site/docs/raspberrypiimage) or [raspberrypi build from scratch](http://emoncms.org/site/docs/raspberrypibuild).

## v6 (timestore+)

Emoncms version 6 brings in the capability of a new feed storage engine called timestore.
Timestore is time-series database designed specifically for time-series data developed by Mike Stirling.

[mikestirling.co.uk/redmine/projects/timestore](mikestirling.co.uk/redmine/projects/timestore)

Timestore's advantages:

**Faster Query speeds**
With timestore feed data query requests are about 10x faster (2700ms using mysql vs 210ms using timestore).
*Note:* initial benchmarks show timestore request time to be around 45ms need to investigate the slightly slower performance may be on the emoncms end rather than timestore.

**Reduced Disk use**
Disk use is also much smaller, A test feed stored in an indexed mysql table used 170mb, stored using timestore which does not need an index and is based on a fixed time interval the same feed used 42mb of disk space. 

**In-built averaging**
Timestore also has an additional benefit of using averaged layers which ensures that requested data is representative of the window of time each datapoint covers.

### Using MYSQL or PHPTimeSeries instead of Timestore

If your a familiar with mysql and want to use mysql to do your own queries and processing of the feed data you may want to select mysql as the default data store rather than timestore. The disadvantage of MYSQL is that it is much slower than timestore for common timeseries queries such as zooming through timeseries data.

There is also another feed engine called PHPTimeSeries which provides improved timeseries query speed than mysql but is still slower than timestore. Its main avantages is that it does not require additional installation of timestore as it uses native php file access, it also stores the data in the same data file .MYD format as mysql which means you can switch from mysql to phptimeseries by copying the .MYD mysql data files directly out of your mysql directory into the PHPTimeSeries directory without additional conversion.

To select either MYSQL or PHPTimeSeries instead of timestore as your default engine set the default engine setting in the emoncms settings.php file to:

    $default_engine = Engine::MYSQL;
    
or: 

    $default_engine = Engine::PHPTIMESERIES;

If you do not wish to use timestore you can skip to step 2 of the installation process.

## 1) Download, make and start timestore

    cd /home/yourusername
    git clone https://github.com/TrystanLea/timestore
    cd timestore
    sudo sh install
    
**Note the adminkey** at the end as you will want to paste this into the emoncms settings.php file.

If the adminkey could not be found, it may be that timestore failed to start:

To check if timestore is running type:

    sudo /etc/init.d/timestore status
    
Start, stop and restart it with:

    sudo /etc/init.d/timestore start
    sudo /etc/init.d/timestore stop
    sudo /etc/init.d/timestore restart
    
To read the adminkey manually type:

    cat /var/lib/timestore/adminkey.txt

## 2) Install Apache, Mysql and PHP (LAMP Server)
    
When installing mysql and the blue dialog appears enter a password for root user, note the password down as you will need it later.

    $ sudo apt-get install apache2
    $ sudo apt-get install mysql-server mysql-client
    $ sudo apt-get install php5 libapache2-mod-php5
    $ sudo apt-get install php5-mysql
    $ sudo apt-get install php5-curl
    
## 3) Enable mod rewrite

Emoncms uses a front controller to route requests, modrewrite needs to be configured:

    $ sudo a2enmod rewrite
    $ sudo nano /etc/apache2/sites-enabled/000-default

Change (line 7 and line 11), "AllowOverride None" to "AllowOverride All".
That is the sections <Directory /> and <Directory /var/www/>.
[Ctrl + X ] then [Y] then [Enter] to Save and exit.

Restart the lamp server:

    $ sudo /etc/init.d/apache2 restart

## 4) Install the emoncms application via git

Git is a source code management and revision control system but at this stage we use it to just download and update the emoncms application.

    $ sudo apt-get install git-core

First cd into the var directory:

    $ cd /var/

Set the permissions of the www directory to be owned by your username:

    $ sudo chown $USER www

Cd into www directory

    $ cd www

Download emoncms using git:

    $ git clone https://github.com/emoncms/emoncms.git
    
Once installed you can pull in updates with:

    git pull

Alternatively download emoncms and unzip to your server:

[https://github.com/emoncms/emoncms](https://github.com/emoncms/emoncms)

Note: Be aware that installing Emoncms to any directory other than /var/www/ will break the data import scripts. (see http://openenergymonitor.org/emon/node/1329#comment-7526).

## 5) Create a MYSQL database

    $ mysql -u root -p

Enter the mysql password that you set above.
Then enter the sql to create a database:

    mysql> CREATE DATABASE emoncms;

Exit mysql by:

    mysql> exit

## 6) Set emoncms database settings.

cd into the emoncms directory where the settings file is located

    $ cd /var/www/emoncms/

Make a copy of default.settings.php and call it settings.php

    $ cp default.settings.php settings.php

Open settings.php in an editor:

    $ nano settings.php

Enter in your database settings.

    $username = "USERNAME";
    $password = "PASSWORD";
    $server   = "localhost";
    $database = "emoncms";

If your using timestore enter the adminkey as copied in step 1 above:
    
    $timestore_adminkey = "";
    
If your not using timestore set the default engine to your selected engine:

    $default_engine = Engine::MYSQL;
    
or

    $default_engine = Engine::PHPTIMESERIES;

Save (Ctrl-X), type Y and exit

## 7) In an internet browser, load emoncms:

<div class='alert alert-info'>

<h3>Note: Browser Compatibility</h3>

<p><b>Chrome Ubuntu 23.0.1271.97</b> - developed with, works great.</p>

<p><b>Chrome Windows 25.0.1364.172</b> - quick check revealed no browser specific bugs.</p>

<p><b>Firefox Ubuntu 15.0.1</b> - no critical browser specific bugs, but movement in the dashboard editor is much less smooth than chrome.</p>

<p><b>Internet explorer 9</b> - works well with compatibility mode turned off. F12 Development tools -> browser mode: IE9. Some widgets such as the hot water cylinder do load later than the dial.</p>

<p><b>IE 8, 7</b> - not recommended, widgets and dashboard editor <b>do not work</b> due to no html5 canvas fix implemented but visualisations do work as these have a fix applied.</p>

</div>

[http://localhost/emoncms](http://localhost/emoncms)

The first time you run emoncms it will automatically setup the database and you will be taken straight to the register/login screen. 

Create an account by entering your email and password and clicking register to complete. 
<br><br>

#### PHP Suhosin module configuration (Debian 6, not required in ubuntu)

Dashboard editing needs to pass parameters through HTTP-GET mechanism and on Debian 6 the max
allowable length of a single parameter is very small (512 byte). This is a problem for designing of dashboard
and when you exceed this threshold all created dashboard are lost...

To overcome this problem modify "suhosin.get.max_value_length" in /etc/php5/conf.d/suhosin.ini" to large
value (8000, 16000 should be fine).

#### Enable Multi lingual support using gettext

Follow the guide here step 4 onwards: [http://emoncms.org/site/docs/gettext](http://emoncms.org/site/docs/gettext)
