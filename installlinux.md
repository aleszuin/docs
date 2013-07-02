## Install Emoncms on Ubuntu / Debian

This guide should work on most debian systems including Ubuntu. For installation guide on installing emoncms on a raspberrypi see [raspberrypi from ready-to-go image](http://emoncms.org/site/docs/raspberrypiimage) or [raspberrypi build from scratch](http://emoncms.org/site/docs/raspberrypibuild).

#### 1) Install mysql

    $ sudo apt-get install mysql-server mysql-client

When the blue dialog appears enter a password for root user, note the password down as you will need it later.

#### 2) Install apache2

    $ sudo apt-get install apache2

#### 3) Install php

    $ sudo apt-get install php5 libapache2-mod-php5
    $ sudo apt-get install php5-mysql

#### 4) Enable mod rewrite

    $ sudo a2enmod rewrite
    $ sudo nano /etc/apache2/sites-enabled/000-default

Change (line 7 and line 11), "AllowOverride None" to "AllowOverride All".
That is the sections <Directory /> and <Directory /var/www/>.
[Ctrl + X ] then [Y] then [Enter] to Save and exit.

#### 5) That completes the server installation, restart the server to make everything above operational:

    $ sudo /etc/init.d/apache2 restart

### Installing emoncms:

#### 6) Install git (recommended but optional)

$ sudo apt-get install git-core

#### 7) Download Emoncms

First cd into the var directory:

    $ cd /var/

Set the permissions of the www directory to be owned by your username (on the raspberrypi its pi):

    $ sudo chown $USER www

Cd into www directory

    $ cd www

Download emoncms using git:

    $ git clone https://github.com/emoncms/emoncms.git

Alternatively download emoncms and unzip to your server:

[https://github.com/emoncms/emoncms](https://github.com/emoncms/emoncms)

#### 8) Create a MYSQL database

    $ mysql -u root -p

Enter the mysql password that you set above.
Then enter the sql to create a database:

    mysql> CREATE DATABASE emoncms;

Exit mysql by:

    mysql> exit

#### 9) Set emoncms database settings.

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

Enter in your database settings.

Save (Ctrl-X), type Y and exit

#### 10) In an internet browser, load emoncms:

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

#### 11) PHP Suhosin module configuration (Optional - Debian 6)

Dashboard editing needs to pass parameters through HTTP-GET mechanism and on Debian 6 the max
allowable length of a single parameter is very small (512 byte). This is a problem for designing of dashboard
and when you exceed this threshold all created dashboard are lost...

to overcame this problem modify "suhosin.get.max_value_length" in /etc/php5/conf.d/suhosin.ini" to large
value (8000, 16000 should be fine).

#### (Optional) Enable Multi lingual support using gettext

Follow the guide here step 4 onwards: [http://openenergymonitor.org/emon/emoncms/using-gettext-on-ubuntu](http://openenergymonitor.org/emon/emoncms/using-gettext-on-ubuntu)
