## Building the Full emoncms on the raspberrypi software stack

Start by either installing the oem_gateway image as detailed here or if you wish follow the gateway installation guide here.

The gateway image comes with a couple of things that make getting started easier, primarily that it already has ssh server installed and so you dont need a hdmi monitor or keyboard connected to your pi to follow the rest of this guide. You can just log into your pi over your network.

The oem_gateway image, comes with the following things pre-installed: ipe debian linux, ssh server, serial settings for rfm12pi, git and Jerome's oem gateway.

One you have written the image to your SD card, insert it in the pi, connect your pi up to ethernet and power and then wait for it to appear in your devices list on your internet router.

SSH into the pi with:

    ssh root@192.168.1.77
    
and password **root**

The default mode for the SD Card is read-only. Change the mount mode to read/write by typing:

    ipe-rw
    
### Step 1: Resize the root partition so that we have some room to install emoncms

    fdisk /dev/mmcblk0

    press d to delete
    press 2 for second partition
    press n to create new partition
    press p for primary
    press 2 for second partition
    press enter to start partition at default
    enter (1200MB*1024*1024) / 512 = 2457600 to create 1200MB partition
    press w to write
    
reboot the pi

    reboot
    
ssh back into the pi and put the pi in read/write mode with: *ipe-rw*

    resize2fs /dev/root
    
### Step 2: Create a new partition that will hold the data

    fdisk /dev/mmcblk0

    press n to create new partition
    press p for primary
    press 3 for third partition
    press enter to start partition at default
    press enter to end partition at default
    press w to write

reboot the pi

    reboot

ssh back into the pi and put the pi in read/write mode with: *ipe-rw*

Now that we have a partition for our data the next step is to create or set the filesystem. Create an ext2 file system which is non journaling with the following:

    mkfs.ext2 /dev/mmcblk0p3
    
Create a mount point for the data partition

    mkdir /data

Mount the filesystem at /data at startup:

    nano /etc/fstab
    
Add the line: 
tep
    /dev/mmcblk0p3  /daSta           ext2    errors=remount-ro 0     0

### Step 3: Install Apache, PHP, MySQL, Timestore and Redis

    apt-get update
    
    mkdir /data/log
    
Install Apache-Mysql-PHP
    
    apt-get install apache2
    apt-get install mysql-server mysql-client
    apt-get install php5 libapache2-mod-php5
    apt-get install php5-mysql
    apt-get install php5-curl
    apt-get install php5-dev
    
    mkdir /data/mysql
    cp -rp /var/lib/mysql/. /data/mysql
    
Install php serial library:

    apt-get install php-pear
    pecl install channel://pecl.php.net/dio-0.0.6
    nano /etc/php5/cli/php.ini

Install redis server

    apt-get install redis-server

Install php redis client:

    git clone https://github.com/nicolasff/phpredis.git
    cd phpredis
    phpize
    ./configure [--enable-redis-igbinary]
    make && make install
    
    mkdir /data/redis
    chown redis:redis /data/redis
    mkdir /data/log/redis
    chown redis:redis /data/log/redis
    
Add both redis client and serial library to php.ini:

    nano /etc/php5/cli/php.ini
    
and:
    
    nano /etc/php5/apache2/php.ini

Add to the beginning of the ;Dynamic Extensions; section on line 843. [Ctrl+W] then enter Dynamic Extensions can be used to search for the correct section. Add the following lines:

    extension=redis.so
    extension=dio.so
    
Install timestore

    cd
    git clone https://github.com/TrystanLea/timestore

    mkdir /data/timestore

Edit timestore data directory 

    nano timestore/src/main.c
    
Change line 43 from

    #define DEFAULT_DB_PATH "/var/lib/timestore"
    
to

    #define DEFAULT_DB_PATH "/data/timestore"
    
and change line 44 to:

    #define DEFAULT_LOG_FILE "/var/log/timestore.log"
    
to

    #define DEFAULT_LOG_FILE "/data/log/timestore.log"
    
Save and exit.

    cd timestore
    
    make

Install timestore init script:

    echo "DIRECTORY=/root/timestore/src/" > /etc/default/timestore
    
change entries in init script from 

    /var/lib/timestore/adminkey.txt
    
to 

    /data/timestore/adminkey.txt
    
    cp initscript/timestore /etc/init.d/
    chmod 755 /etc/init.d/timestore
    update-rc.d timestore defaults

Apache mod rewrite and log settings:

    a2enmod rewrite
    nano /etc/apache2/sites-enabled/000-default
    
Change (line 7 and line 11), "AllowOverride None" to "AllowOverride All". 
Turn off the access.log by adding a # in front of the line that starts with CustomLog. 
[Ctrl + X ] then [Y] then [Enter] to Save and exit.

    nano /etc/apache2/conf.d/other-vhosts-access-log

Place a # in front of CustomLog again, save and exit.

    nano /etc/apache2/envvars
    
    export APACHE_LOG_DIR=/data/log/apache2$SUFFIX
    
    mkdir /data/log/apache2
    
MYSQL log and data location settings

    nano /etc/mysql/my.cnf

    change line datadir to /data/mysql

Redis settings

nano /etc/redis/redis.conf

set LogFile location to /data/log/redis/redis-server.log

change save to: save 900 1 only

chage working directory

mkdir /data/phptimeseries

## Install emoncms

git clone -b redismetadata https://github.com/emoncms/emoncms.git

cp default.settings.php settings.php
 nano settings.php
 
mysql -u root -p 

CREATE DATABASE emoncms;

cat /data/timestore/adminkey.txt 

Install raspberrypi module

 nano /etc/rc.local

(sleep 10; python /root/oem_gateway/oemgateway.py --config-file /boot/oemgateway.conf

sudo cp /var/www/emoncms/Modules/raspberrypi/rfm12piphp /etc/init.d/
nano /etc/init.d/rfm12piphp

remove lock file 

### Install ufw

http://blog.al4.co.nz/2011/05/setting-up-a-secure-ubuntu-lamp-server/

   apt-get install ufw
   ufw allow 80/tcp
   ufw allow 443/tcp
   ufw allow 22/tcp
   ufw enable
   
Set root password

   passwd root
   
   default:
   {@.o7SNf~Qg3-0}#SRCM
   
Create a user 

   useradd -m -G sudo,adm -s /bin/bash pi
   
   passwd pi
   
   default: (change this for secure installation)
   raspberry 
   
Disable root login via ssh
 
   nano /etc/ssh/sshd_config
   
set 

    PermitRootLogin to no

at the bottom add the lines:

    AllowUsers pi
    AllowGroups adm

Secure MySQL   

Remove anonymous accounts
DROP USER ''@'localhost';
DROP USER ''@'oemgateway';
DROP DATABASE test;


User login for default emoncms account is

admin
raspberry


To get sudo to work I had too:
http://www.linuxfromscratch.org/blfs/view/cvs/postlfs/sudo.html
wget http://www.sudo.ws/sudo/dist/sudo-1.8.8.tar.gz
tar -zxvf sudo-1.8.8.tar.gz

./configure --prefix=/usr                      \
            --libexecdir=/usr/lib/sudo         \
            --docdir=/usr/share/doc/sudo-1.8.8 \
            --with-timedir=/data/sudo       \
            --with-all-insults                 \
            --with-env-editor                  &&
make


Add redirect in /var/www

    <?php header('Location: ../emoncms'); ?>

    <html><body><h1>Welcome</h1>
    <p><a href="emoncms" >Goto Emoncms</a></p>
    </body></html>
    

Set emoncms to use timestore data directory

Install usefulscripts

Port data across from second raspberrypi or emoncms.org account
