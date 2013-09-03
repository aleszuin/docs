## Emoncms on the Raspberry PI - Build from scratch

Download Raspbian 'Wheezy' SD card image [here](http://www.raspberrypi.org/downloads) (This guide was made using 9th February 2013 release). Extract the image.

### Write image to an SD card (Linux)

Start by inserting your SD card, your distribution should mount it automatically so the first step is to unmount the SD card and make a note of the SD card device name, to view mounted disks and partitions run:

    $ df -h

You should see something like this:

    Filesystem            Size  Used Avail Use% Mounted on
    /dev/sda6             120G   90G   24G  79% /
    none                  490M  700K  490M   1% /dev
    none                  497M  1.7M  495M   1% /dev/shm
    none                  497M  260K  497M   1% /var/run
    none                  497M     0  497M   0% /var/lock
    /dev/sdb1             3.7G  4.0K  3.7G   1% /media/sandisk

Unmount the SD card, change sdb to match your SD card drive:

    $ umount /dev/sdb1 

If the card has more than one partition unmount that also: 

    $ umount /dev/sdb2

Locate the directory of your downloaded emoncms image in terminal and write it to an SD card using linux tool *dd*:

<div class='alert alert-error'><i class='icon-fire'></i> <b>Warning:</b> take care with running the following command that your pointing at the right drive! If you point at your computer drive you will loose a lot of data!</div>

    $ sudo dd bs=4M if=2012-02-09-wheezy-raspbian.img of=/dev/sdb

Once complete insert the SD card into the PI and connect ethernet and power. All the lights in the corner should light up and flicker. 

The next step is to connect to your PI via SSH from your computer, to do this find the ip address of the PI on your local network, then using linux terminal run:

    $ SSH pi@xxx.xxx.xxx.xxx

On windows you can do the above step with a nice bit of software called putty. Once you make the connection request the PI will ask for a password. The default password is 'raspberry'.

Now that you have your RaspberryPi up and working, we are now going to install the Emonncms software. To do this we need to install the webserver software (Apache), the database (MySQL) as well as the Emoncms scripts (written in a programming language called php).

    $ sudo apt-get update

### Apache, Mysql and PHP

This installation guide was create when installing emoncms on the PI with a blank install of raspbian “wheezy”, however as the raspbian wheezy is based on debian this guide should work on most debian systems including Ubuntu.

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
[Ctrl + X ] then [Y] then [Enter] to Save and exit.

### Installing emoncms:

#### 6) Install git (recommended but optional)

    $ sudo apt-get install git-core

#### 7) Download Emoncms

First cd into the var directory:

    $ cd /var/

Set the permissions of the www directory to be owned by the pi username:

    $ sudo chown pi www

Cd into www directory

    $ cd www

Download emoncms using git:

    $ git clone https://github.com/emoncms/emoncms.git

<div class='alert'><b>Note:</b> Be aware that installing Emoncms to any directory other than /var/www/ will break the data import scripts. (see http://openenergymonitor.org/emon/node/1329#comment-7526).</div>

#### 8) Install the raspberrypi module

Navigate to the emoncms modules folder 

    $ cd /var/www/emoncms/Modules 

Download the Raspberry Pi emoncms module into the Modules folder using git: 

    $ git clone https://github.com/emoncms/raspberrypi.git

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

Open settings.php in an editor, nano works great on the pi:

    $ nano settings.php

Enter in your database settings.

    $username = "root";
    $password = "raspberry";
    $server   = "localhost";
    $database = "emoncms";

Save (Ctrl-X), type Y and exit

### RFM12BPi Setup

Make sure your Raspberry Pi’s UART is disconnected from the console and available for programs to use.

#### 1) Backup cmdline.txt: 

    $ sudo cp /boot/cmdline.txt /boot/cmdline_backup.txt

#### 2) Edit cmdline.txt to remove references to Pi’s UART (ttyAMA0)

    $ sudo nano /boot/cmdline.txt

edit it from 

    dwc_otg.lpm_enable=0 console=ttyAMA0,115200 kgdboc=ttyAMA0,115200 console=tty1 root=/dev/mmcblk0p2 rootfstype=ext4 elevator=deadline rootwait 

to 

    dwc_otg.lpm_enable=0 console=tty1 root=/dev/mmcblk0p2 rootfstype=ext4 elevator=deadline rootwait 

[Ctrl+X] then [y] then [Enter] to save and exit

#### 3) Edit inittab

    $ sudo nano /etc/inittab 

At the bottom of the file comment out the line (by adding a '#' at begining)

    # T0:23:respawn:/sbin/getty -L ttyAMA0 115200 vt100

[Ctrl+X] then [y] then [Enter] to save and exit

### Install rfm12pi gateway service

Install one of the two available gateway scripts and let them run on startup. The PHP gateway is the latest and greatest:

#### PHP Gateway

Install serial PHP libraries

    $ sudo apt-get install php-pear php5-dev
    $ sudo pecl install channel://pecl.php.net/dio-0.0.6
    $ sudo nano /etc/php5/cli/php.ini

add *extension=dio.so* to file in the beginning of the ;Dynamic Extensions; section on line 843. [Ctrl+W] then enter *Dynamic Extensions* can be used to search for the correct section  

[Ctrl+X] then [y] then [Enter] to save and exit

Install rfm12piphp gateway service:

    sudo cp /var/www/emoncms/Modules/raspberrypi/rfm12piphp /etc/init.d/
    sudo chmod 755 /etc/init.d/rfm12piphp
    sudo update-rc.d rfm12piphp defaults

#### Python Gateway

  Install python serial port and mySQL modules

    $ sudo aptitude install python-serial python-mysqldb
  
  Ensure the script is executable
  
    $ chmod 755 /var/www/emoncms/Modules/raspberrypi/rfm2pigateway.py
  
  Create group emoncms and make user pi part of it

    $ sudo groupadd emoncms
    $ sudo usermod -a -G emoncms pi

  Create a directory for the logfiles and give ownership to user pi, group emoncms

    $ sudo mkdir /var/log/rfm2pigateway
    $ sudo chown pi:emoncms /var/log/rfm2pigateway
    $ sudo chmod 750 /var/log/rfm2pigateway

  Make script run as daemon on startup

    $ sudo cp /var/www/emoncms/Modules/raspberrypi/rfm2pigateway.init.dist /etc/init.d/rfm2pigateway
    $ sudo chmod 755 /etc/init.d/rfm2pigateway
    $ sudo update-rc.d rfm2pigateway defaults 99

### Reboot

To complete all of the above reboot the pi

    $ sudo reboot

#### 10) In an internet browser on the local ntwork, load emoncms by browsing to the Pi's IP address:

<div class='alert alert-info'>

<h3>Note: Browser Compatibility</h3>

<p><b>Chrome Ubuntu 23.0.1271.97</b> - developed with, works great.</p>

<p><b>Chrome Windows 25.0.1364.172</b> - quick check revealed no browser specific bugs.</p>

<p><b>Firefox Ubuntu 15.0.1</b> - no critical browser specific bugs, but movement in the dashboard editor is much less smooth than chrome.</p>

<p><b>Internet explorer 9</b> - works well with compatibility mode turned off. F12 Development tools -> browser mode: IE9. Some widgets such as the hot water cylinder do load later than the dial.</p>

<p><b>IE 8, 7</b> - not recommended, widgets and dashboard editor <b>do not work</b> due to no html5 canvas fix implemented but visualisations do work as these have a fix applied.</p>

</div>

[http://localhost/emoncms](http://localhost/emoncms)

The first time you run emoncms it will automatically setup the database and you will be taken straight to the login screen. 

Login with username **raspi** and password **raspberry**


### Optional

If your SD Card is larger than 4GB you may want to expand the partition to fill the disk, this can be done easily with the raspi-config utility:

    $ sudo raspi-config

Select Expand root partition to fill SD card. Before rebooting you may also like to change the following:

- If you plan to run the Pi as a headless dataloggin emoncms server as we do then select memory-split and choose the first setting a 240/16 split, the gives the CPU more memory at the expense of graphics which we're not using.

- It is also recommended to disabling booting straight into a desktop since this increases boot time and wastes system resources. If required a desktop can be loaded with $ startx.

- Set locale and timezone as required.

- Change password for user Pi to something of your choice, make it secure it will be storing your home energy data!

Finish and reboot, remember to use your new password when SSH'ing back in!

#### Host name

Set a host name for the Pi to enable host name to be used instead of IP address when connecting to the Pi
Change the default raspberrypi host name to that of your choice eg.emoncms in the file 

    $ sudo nano /etc/hostname 

[Ctrl+X] then [Y] then [Enter] to save and exit

and in the file 

    $ sudo nano /etc/hosts 

[Ctrl+X] then [Y] then [Enter] to save and exit

Reboot the Pi (sudo reboot) and you should be able to SSH back in with $ ssh pi@emoncms if 'emoncms' was your chosen host name. 

<div class='alert'><b>Note:</b> this won't work with all routers, you might need to set the Pi as a 'Fixed Host' in the router config</div>

#### Install minicom

Minicom is a useful tool for reading from the serial port that the rfm12pi board is connected to:

    $ sudo apt-get install minicom

#### Add redirect index.php in /var/www

    <?php header('Location: ../emoncms'); ?>

    <html><body><h1>Welcome</h1>
    <p><a href="emoncms" >Goto Emoncms</a></p>
    </body></html>

