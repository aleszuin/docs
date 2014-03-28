# Shared Linux Hosting

Your shared hosting provider should already have a LAMP server installed. You may need to ask your hosting provider to enable mod_rewrite. It's unlikely that redis will be available (redis is used to improve performance through caching), but emoncms can be run without it.

To install emoncms on a shared server

1) Download the emoncms zip file from:

[https://github.com/emoncms/emoncms/archive/master.zip](https://github.com/emoncms/emoncms/archive/master.zip)

Unzip to your shared server's public_html folder, rename the folder to emoncms.

2) Create a mysql database for your emoncms installation, note down its name, username and password.

3) In your shared hosting /home/username folder create a folder called emoncmsdata to hold your emoncms feed data. (Note: NOT public_html as the data files should not be publicly accessible).
Then create three folders within your emoncmsdata folder called: phpfiwa, phpfina and phptimeseries

4) In the emoncms app directory make a copy of default_settings.php and call it settings.php. Open settings.php and enter your mysql username, password and database. In the feed_settings section uncomment the datadir defenitions and set them to the location of each of the feed engine data folders on your system.

5) Thats it, emoncms should now be ready to use! 
