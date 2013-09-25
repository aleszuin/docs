## Emoncms on the Raspberry PI - Ready-to-go image

#### 1) Download the ready-to-go SD card image:
<br>

**Latest: emoncmspiv5avr_02_05_13.zip (770.9MB)**

[Dropbox Download Mirror 1](https://www.dropbox.com/s/koqvr4yp8spl59d/emoncmspiv5avr_02_05_13.zip)
* with minimal log files turned on by default to reduce Raspberry Pi SD card wear

<br>


**Older: emoncmspi image (1.1GB)**

[Dropbox Download Mirror 1](https://www.dropbox.com/s/mz3klx0a2caelzo/emoncmspiv5avr_19-05-13.img.zip)

[Dropbox Download Mirror 2](https://www.dropbox.com/s/6x095kmgkead78u/emoncmspiv5avr_19-05-13.img.zip)

[Ubuntu One Download Mirror 3](http://t.co/xwVRlrC5Vj)

[Torrent Download](http://torcache.net/torrent/AED670234438B2DDB7FC4842F429041F31696693.torrent) -please help seed, more info [here](http://openenergymonitor.org/emon/node/2278)


The image need a minimum SD card size of 4GB. On a 4GB card 1.5GB is free for data storage. If you're card it larger than 4GB you can expand the partition by running **$ sudo raspi-config** once your're up and running


#### 2) Write image to an SD card (Linux)

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

    $ sudo dd bs=4M if=emoncmspiv5avr_2may.img of=/dev/sdb  


<br>
#### 3) Intall RFM12Pi
Install RFM12Pi hardware expansion module onto the Pi's GPIO pins taking care to align up pin 1. **The RFM12Pi should be installed BEFORE the Pi is booted up.**

#### 4) Power it up!

That's it all you need to do now is insert the SD card in the pi, connect ethernet and power it up! Access the PI via your internet browser in the same way as you would access your home router. You can usually find the ip address of the PI by looking at the DHCP table in your router. Alternatively, you can use a network scanning app such as Fing (android), iNet (Mac), iNet (iPhone) to scan your network.
    
Return to the OpenEnergyMonitor Guide to setup your sensor nodes and map the inputs in emoncms: http://openenergymonitor.org/emon/guide

<div class='alert alert-info'>

<h3>Note: Browser Compatibility</h3>

<p><b>Chrome Ubuntu 25.0.1364.160</b> - developed with, works great.</p>

<p><b>Chrome Windows 25.0.1364.172</b> - quick check revealed no browser specific bugs.</p>

<p><b>Firefox Ubuntu 15.0.1</b> - no critical browser specific bugs, but movement in the dashboard editor is much less smooth than chrome.</p>

<p><b>Internet explorer 9</b> - works well with compatibility mode turned off. F12 Development tools -> browser mode: IE9. Some widgets such as the hot water cylinder do load later than the dial.</p>

<p><b>IE 8, 7</b> - not recommended, widgets and dashboard editor <b>do not work</b> due to no html5 canvas fix implemented but visualisations do work as these have a fix applied.</p>

</div>
