#### From v6 to v7 (redis)

As part of recent work to improve the performance of emoncms because of high load's on emoncms.org redis was introduced to store feed and input meta data including last feed time and value fields which where causing significant write load on the server. This change benefits all installation types of emoncms whether emoncms.org or a raspberrypi as it siginficantly reduces the amount of disk writes. 

Using redis in this way leads to quite a big performance improvement. Enabling almost 5 times the request rate in benchmarking.

Blog post: [http://openenergymonitor.blogspot.co.uk/2013/11/improving-emoncms-performance-with_8.html](http://openenergymonitor.blogspot.co.uk/2013/11/improving-emoncms-performance-with_8.html)

To upgrade you will need redis server installed and the phpredis client:

    sudo apt-get install redis-server
    sudo pecl install redis
    
Add pecl redis module to php5 config
    
    sudo sh -c 'echo "extension=redis.so" > /etc/php5/apache2/conf.d/20-redis.ini'
    sudo sh -c 'echo "extension=redis.so" > /etc/php5/cli/conf.d/20-redis.ini'

#### From v5 to v6 (timestore+)

Emoncms version 6 brings in the capability of a new feed storage engine called timestore.
Timestore is time-series database designed specifically for time-series data developed by Mike Stirling.

[http://mikestirling.co.uk/redmine/projects/timestore](http://mikestirling.co.uk/redmine/projects/timestore)

Timestore's advantages:

**Faster Query speeds**
With timestore feed data query requests are about 10x faster (2700ms using mysql vs 210ms using timestore).
*Note:* initial benchmarks show timestore request time to be around 45ms need to investigate the slightly slower performance may be on the emoncms end rather than timestore.

**Reduced Disk use**
Disk use is also much smaller, A test feed stored in an indexed mysql table used 170mb, stored using timestore which does not need an index and is based on a fixed time interval the same feed used 42mb of disk space. 

**In-built averaging**
Timestore also has an additional benefit of using averaged layers which ensures that requested data is representative of the window of time each datapoint covers.

#### Using MYSQL or PHPTimeSeries instead of Timestore

If you are familiar with mysql and want to use mysql to do your own queries and processing of the feed data you may want to select mysql as the default data store rather than timestore. The disadvantage of MYSQL is that it is much slower than timestore for common timeseries queries such as zooming through timeseries data.

There is also another feed engine called PHPTimeSeries which provides improved timeseries query speed than mysql but is still slower than timestore. Its main avantages is that it does not require additional installation of timestore as it uses native php file access, it also stores the data in the same data file .MYD format as mysql which means you can switch from mysql to phptimeseries by copying the .MYD mysql data files directly out of your mysql directory into the PHPTimeSeries directory without additional conversion.

To select either MYSQL or PHPTimeSeries instead of timestore as your default engine set the default engine setting in the emoncms settings.php file to:

    $default_engine = Engine::MYSQL;
    
or: 

    $default_engine = Engine::PHPTIMESERIES;

You can skip steps 1 and 2 if you do not wish to use timestore.

### 1) Download, make and start timestore

    cd /home/username
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
    
### 2) Install php curl
    
    sudo apt-get update
    sudo apt-get install php5-curl

### 3) Update the emoncms application

Run git pull in your emoncms directory

    cd /var/www/emoncms
    git pull

### 4) Update settings.php

Backup and remove your old settings.php file and create a new one from default.settings.php.

Open settings.php in an editor:

    $ nano settings.php

Enter in your database settings.

    $username = "USERNAME";
    $password = "PASSWORD";
    $server   = "localhost";
    $database = "emoncms";
    
If you're using timestore enter the adminkey found by typing:

    cat /var/lib/timestore/adminkey.txt
    
    $timestore_adminkey = "";
    
If you're not using timestore set the default engine to your selected engine:

    $default_engine = Engine::MYSQL;
    
or

    $default_engine = Engine::PHPTIMESERIES;

### 5) Update database

Log in with the administrator account (first account created)

Click on the *Admin* tab (top-right) and run database update.

Click on feeds, check that everything is working as expected, if your monitoring equipment is still posting you should see data coming in as usual.

### Converting existing feeds to timestore

So far we've got everything in place for using timestore but any existing feeds are still stored as mysql tables. To convert existing mysql feeds over to timestore a module has been written specifically for managing the conversion of the feeds, to download and run it:

    cd /var/www/emoncms/Modules

    git clone https://github.com/emoncms/converttotimestore
    
Again log in with the administrator account (first account created)
Click on the *Admin* tab (top-right) and run database update.

Navigate to the convert to timestore menu item in the dropdown menu titled Extras and follow the steps outlined.
    
#### Need help?
See timestore forum discussion: [http://openenergymonitor.org/emon/node/2651](http://openenergymonitor.org/emon/node/2651)


## Upgrading from version 4.0 to 5.0

**Note:** Version 4.0 was commonly refered to as modular emoncms. 

### 1) Download
Download the latest version either by clicking on the zip icon in github or if you previously used git clone you can download the latest changes with:

    $ git pull origin master
    
### 2) Settings.php
Make a copy of your current *settings.php* file and create a new *settings.php* anew from *default.settings.php* Enter your emoncms database settings.

Add the following line to the bottom of *settings.php* to enable a special database update only session, be sure to remove this line from settings.php once complete:
 
    $updatelogin = true;
    
### 3) Run the database updater
In your internet browser open the admin/view page and click on the database update and check button to launch the database update script.

    http://localhost/emoncms/admin/view
    
You should now see a list of changes to be performed on your existing emoncms database.
You may at this point want to backup your input and users table before applying the changes.

### 4) If you're running emoncms on a Raspberry Pi with an RFM12Pi and RaspberryPi emoncms module
You will need to remove the cron job entry for the PHP gateway script. Run 
    
    $ sudo nano /etc/crontab 
    
and remove the entry with emoncms php. Now restart the Pi. The new Deamon script should be runnning see RaspberryPi emoncms module Github Readme for info on how to check

That should be it.

#### Troubleshooting

You may need to clear your browser cache if an interface appears buggy.


## Upgrading from older versions, version 2 and 3

Upgrade scripts for older versions will be reintroduced very soon, which will make this much easier.

### 15th November 2012 - Removal of feed_relation table and new userid field in feeds table.

If you're upgrading from a pre November 2012 (emoncms3) to the current version https://github.com/emoncms/emoncms the new feeds module implementation associates a user with their feeds in a different way. This information is now stored in the feeds table rather than a seperate feed_relation table.

A small script has been created to make this conversion easy. In your emoncms directory you will see a script called **migrate.php** 

- Open this script in a text editor and remove the line that says die; so that you can run the script.
- Run the script in your browser, this is usually pretty quick.
- Once complete you can either remove the script or stop it from running again by re-entering die; at the top.

### 15th July 2012 - New dashboard implementation

With the new dashboard implementation and the new visual dashboard editor dashboards created in versions of emoncms before mid-july 2012 no longer work with the current version as the dashboard renderer has been reworked. It is best to rebuild any dashboards using the new visual dashboard editor or the new ckeditor rebuild, there is a guide on how to do this in the using emoncms documentation.

### 13th April 2012 - Feed tables now use integer timestamp, conversion needed.

If your emoncms version is pre 13th of April 2012, you will need to convert your feed tables from datetime to integer timestamp time format. A conversion script has been created to make this process easy. For this script line 13 needs to be uncommented before running it. For more information see [http://openenergymonitor.blogspot.com/2012/04/speeding-up-emoncms-feed-data-requests.html](http://openenergymonitor.blogspot.com/2012/04/speeding-up-emoncms-feed-data-requests.html)
