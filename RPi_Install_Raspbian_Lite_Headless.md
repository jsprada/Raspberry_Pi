# Setup Raspberry Pi - Base Minimal Headless
By: Johnny Sprada 11/13/2018

## Objectives
### Primary Objective:
Use a Linux host computer to set up a fresh Raspberry Pi that can be used as headless server. over wifi.

It should be noted that this basic configuration is intended to build a device for internal use only.  It should not be exposed to the open Internet unless it's first hardened.   We will not be making any security configurations during this procedure.  If there's demand for hardening a Raspberry Pi for public use, let me know and I'll build a guideline for doing that.

### Secondary Objective:
Target audience for this procedure is aimed at people who are newish to Linux, who are transitioning into intermediate.    The procedure is geared more toward *learning* by doing rather than creating the quickest or most efficient path to completion.

There are easier/quicker ways to perform several of these steps, but in the spirit of learning we are going to make several configurations by hand.



## Requirements
* Host computer with your favorite GNU/Linux installed.
* Raspberry Pi 3  (This particular demo was created on a model B)
* MicroSD card (8gb, because that's what I have on hand)
* USB MicroSD card adapter
* Micro USB cable, to supply power
* Network cable, and an available port on your switch/router/modem



## Get and install operating system

These instructions are rough notes for installing Raspbian on an SD card, for use in a Raspberry Pi device.

Download latest Raspbian Stretch Lite from this link:
https://www.raspberrypi.org/downloads/raspbian/

SHA-256 at time of download (11/13/2018).
98444134e98cbb27e112f68422f9b1a42020b64a6fd29e2f6e941a3358d171b4

The SHA-256 code is a hash which was created at the time the file was published for downloading.   This is provided by the author which allows us, as end users to validate that the source code we are downloading has not been modified since it's been published.

After downloading the file, we can create our own hash using the same algorithm as the author (SHA-256) and we should return an *identical* hash code.   If for any reason the hash you generate in the following steps is different from the hash published on the author's web site, the code should not be used and you should notify the author.

#### Dowload image:

    $ curl -O http://director.downloads.raspberrypi.org/raspbian_lite/images/raspbian_lite-2018-10-11/2018-10-09-raspbian-stretch-lite.zip

#### Check hash
Run the following command to verify that your file will generate the same hash that the author provided.  If you get something different, do not proceed, or do so at your own risk.

    $ sha256sum 2018-10-09-raspbian-stretch-lite.zip
    98444134e98cbb27e112f68422f9b1a42020b64a6fd29e2f6e941a3358d171b4  2018-10-09-raspbian-stretch-lite.zip

#### Extract image file
After extracting the file, I like to use `ls` to verify that the *.img file exists.   This is the disk image that we will write to the SD card.

    $ unzip 2018-10-09-raspbian-stretch-lite.zip
    $ ls
    2018-10-09-raspbian-stretch-lite.img


#### Write disk image to SD compared

Get a baseline list of the serial block devices already attached to the system using the command `lsblk`.  For more information about this utility, see `man lsblk` :

    $ lsblk
    NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sda      8:0    0 111.8G  0 disk
    ├─sda1   8:1    0   512M  0 part /boot
    ├─sda2   8:2    0  99.5G  0 part /
    ├─sda3   8:3    0     8G  0 part [SWAP]
    └─sda4   8:4    0   3.8G  0 part
    sdb      8:16   0 238.5G  0 disk
    └─sdb1   8:17   0 238.5G  0 part /home
    sdc      8:32   0 931.5G  0 disk
    └─sdc1   8:33   0 931.5G  0 part /mnt/1tb

Take note of the *last*  block device node in use (`sdc` in this example).

Insert your SD card into the SD card adapter, and plug it into an available USB slot on the host computer.

Run `lsblk` again to get the name of the new block device.


    $ lsblk
    NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sda      8:0    0 111.8G  0 disk
    ├─sda1   8:1    0   512M  0 part /boot
    ├─sda2   8:2    0  99.5G  0 part /
    ├─sda3   8:3    0     8G  0 part [SWAP]
    └─sda4   8:4    0   3.8G  0 part
    sdb      8:16   0 238.5G  0 disk
    └─sdb1   8:17   0 238.5G  0 part /home
    sdc      8:32   0 931.5G  0 disk
    └─sdc1   8:33   0 931.5G  0 part /mnt/1tb
    sde      8:64   1   7.4G  0 disk

Note that `sde` is new in this list, when compared to the baseline list.  It is VERY important that you take a moment to validate that this is indeed the device that you intend to install your image on.

If this is the first time that you have used this method, I would suggest unplugging the USB device, and running through the last couple of steps a few times until you are comfortable distinguishing the new device from the existing devices.

We are going to run a command that will *overwrite* everything on the the target block device.  If you accidentally overwrite the wrong block device, you could very well destroy the data on your operating system disk.
Double check, triple check, that the system detected the micro SD card, and note the new node.  In this example, it's 'sde'  Yours will be different.

We will use the program called `dd` which has been nicknamed "Disk Destroyer" because that is what it will do if you aren't very careful with it.  The utility is used to make a bit by bit copy of a device to another device, or a device to image file, or in this case, we're using it to take an image file and writing it to our device.  For more information on the utility, see `man dd`

To write an image to a block device, `dd` requires at least two parameters.  The first paramerter is an input file (designated by "if=/path/file").  The second parameter is an output file (designated by "of=/path/file").

For our purpose, the input file will be the .img image we extracted previously, and the output file will be the SD card block device.

Running this command will DESTROY the data on the device where the output file ('of') directive is pointed at.  In this example, it's pointed at `*sde*` which will not run correctly with the astrisks.  In your example, you cannot include astrisks.  I included these to prevent anyone from just copy/pasting the command without thinking about it.   Do not just copy this command verbatim, you must undsertand what your objective is, and which parameters to use, and type it out yourself.

FYI, I'm including `&& sync`  at the end of this command, which is sort of like a catch-all to reduce problems on some systems that might cache changes to disk before writing them.   Use of this command is not always necessary, but used as a precaution.  For more information about it, see `man sync`.


    $ sudo dd if=/home/johnny/Downloads/2018-10-09-raspbian-stretch-lite.img of=/dev/*sde* && sync


After the command completes (might take several minutes, with no visual indicators that it's working,  to finish writing image to the device)

When the command completes, remove SD card from host computer, insert into Raspberry Pi, and boot the Raspberry Pi.


## Log in to Raspberry Pi and setup.

Connect Pi to keyboard, monitor, network connection, insert SD card into Pi and power up.

Using the attached keyboard, the following commands will be completed on the Raspberry Pi, not the host computer.

    Login: pi
    Password: raspberry

### Enable SSH

    $ sudo systemctl enable ssh.socket
    $ sudo systemctl start ssh.socket


Find IP address:

    $ ip addr
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether b8:27:eb:2c:85:35 brd ff:ff:ff:ff:ff:ff
        inet 192.168.1.175/24 brd 192.168.1.255 scope global eth0

Note that in this example, third line down,  the IP address is 192.168.1.175.  Yours will vary from the example here, write this number down.

## Log into Raspberry Pi, remotely from the host computer, via SSH

Go back to the host computer, and SSH to the Pi using the IP address discovered in previous step:

    $ ssh pi@192.168.1.175

After authenticating, it's okay to disconnect the monitor and keyboard.  The remaining configuration will be done via SSH from the host computer.




## Congifure base operating system

Log into Raspberry Pi, via SSH.

### Check for and install updates

    $ sudo apt-get update
    $ sudo apt-get -y dist-upgrade

It will take several minutes to update, depending on Internet connection speed.

Once complete,check and set the timezone if necessary:

### Check and set timezone

Check current timezone, note, your results will vary from mine.:

    $ timedatectl status
                                Local time: Sun 2018-11-18 12:45:35 PST
                         Universal time: Sun 2018-11-18 20:45:35 UTC
                                   RTC time: Sun 2018-11-18 20:45:36
                                 Time zone: America/Los_Angeles (PST, -0800)
    System clock synchronized: yes
                               NTP service: active
                          RTC in local TZ: no


If you need to change the time zone, you may query a list of valid options:

    $ timedatectl list-timezones

And set it using the appropriate time zone:

    $ sudo timedatectl set-timezone America/Los_Angeles

### Setup WiFi

Configuring wifi requires you to add your wireless SSID and passphrase into a text file, stored on the Raspberry Pi.  This may or may not be okay with you, but I would prefer to explain how we can encrpyt the passphrase for some extra security.

Here's a sort of magic all in one shortcut/step.  You could just as well edit the `/etc/wpa_supplicant/wpa_supplicant.conf` file by hand, configuring your SSID and passphrase if you'd like.  Instead, I'm providing a shrtcut/seemingly magic command that will create your encrypted passphrase then append the information to the correct configuration file.

Type the following, but use your real SSID.  When the program pauses, enter your wifi password and hit enter:

For more information about the commands used, see `man wpa_passphrase` and `man tee`.

    $ wpa_passphrase "your_ssid" | sudo tee -a /etc/wpa_supplicant/wpa_supplicant.conf > /dev/null

Restart wifi:

    $ wpa_cli -i wlan0 reconfigure

#### Important Note about WiFi
Every country has different laws about radio frequency use.  It is your responsibility to understand this, and configure your device for correct use.  In the US, you can get up to $25k fine from the FCC if you use incorrect frequencies and interfere with someone elses radio traffic.

Please take a moment to configure your device for the country you will use it in.  For more information, google "wifi cdma."

Apply your wifi cdma country code.  Mine is US:

    $ echo country=US | sudo tee -a /etc/wpa_supplicant/wpa_supplicant.conf

Check your wifi IP address:

    $ ip addr

Mine looks like this, 192.168.1.176 in this example.  Again yours will be different, but located in the third line of output.  Take note of this IP address.:

    3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether b8:27:eb:79:d0:60 brd ff:ff:ff:ff:ff:ff
        inet 192.168.1.176/24 brd 192.168.1.255 scope global wlan0


Now you can disconnect your ethernet cable, and SSH to the device using your wifi IP address.

Go to the host computer and try this (using your IP address, not mine):

    $ ssh pi@192.168.1.176


    ### Make any remaining configuration changes

    Change anything you'd like:
        $ sudo raspi-config

I recommend changing default password, host name, locale, expand filesystem, etc.

I will enable I2C for a battery backup device that I'm adding, you may not use this.

I will also be enabling 1-wire for the temp sensors.



## (Optional - if you have supporting hardware) Enable UPS Battery Backup

    $ sudo apt-get -y install build-essential git
    $ git clone https://github.com/xorbit/LiFePO4wered-Pi.git
    $ cd LiFePO4wered-Pi/
    $ ./build.py
    $ sudo ./INSTALL.sh

Connect hardware device, reboot.





## Get Science Project Code

Log in to the Pi via SSH over
    $ git clone https://github.com/jsprada/science_project
    $ cd science_project
    $ sudo sh install.sh
