## Backing up a raspberrypi or emoncms.org account

### 1) Install emoncms on your backup machine following the guide here: 

[http://emoncms.org/site/docs/installlinux](http://emoncms.org/site/docs/installlinux)

Create an account and note down your mysql credentials.

### 2) Download the usefulscripts repository

[https://github.com/emoncms/usefulscripts](https://github.com/emoncms/usefulscripts)

There are two scripts available under usefulscripts/replication

    import_full.php
    import_inputs.php

### 3) Importing inputs

Open to edit import_inputs.php. Set your mysql database name, username and password, the same credentials as for the settings.php step of the emoncms installation. Set the $server variable to the location of the raspberrypi or emoncms.org account you want to backup and set the $apikey to the write apikey of that account.

In terminal, goto the usefulscripts/replication directory. Run import_inputs.php:

    php import_inputs.php

If successful you should now see your input list and all input processing information backed up in your local emoncms account.

### 4) Backup all feed data:

As for importing inputs open import_full.php and set the database credentials and remote emoncms account details.

Run the backup script with sudo

    sudo php import_full.php

That's it, it should now work through all your feeds whether mysql or timestore making a local backup. When you first run this script it can take a long time. When you run this script again it will only download the most recent data and so will complete much faster.
