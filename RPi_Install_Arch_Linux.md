# Raspberry Pi - Install Arch Linux and WiFi
A guide to install Arch Linux ARM on a Raspberry Pi or Raspberry Pi 2 device, by Johnny Sprada

This guide assumes you have a Linux host computer to set up your Micro SD card on.

## Install system to microSD card:
Insert your SD card into the appropriate USB adapter in your host computer.

Locate the name that your system assigned the SD card:

    $ lsblk
	NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sda      8:0    0 238.5G  0 disk
    ├─sda1   8:1    0   255M  0 part /boot
    ├─sda2   8:2    0 230.4G  0 part /
    ├─sda3   8:3    0     1K  0 part
    └─sda5   8:5    0   7.9G  0 part [SWAP]
    sdb      8:16   0 223.6G  0 disk
    ├─sdb1   8:17   0   350M  0 part
    ├─sdb2   8:18   0 222.8G  0 part
    └─sdb3   8:19   0   450M  0 part
    sdc      8:32   0 931.5G  0 disk
    └─sdc1   8:33   0 931.5G  0 part /mnt/1tb
    sdd      8:48   1  29.9G  0 disk
    ├─sdd1   8:49   1   128M  0 part
    └─sdd2   8:50   1  29.8G  0 part

In my example above, my SD card is called `sdd`  which is located at `\dev\sdd`   You MUST double check the name of this device because we're going to erase it.   If you accidentally use the wrong device name, you risk erasing any of your hard disks.  You've been warned!  Use the follwoing process to identify the name of your device.

To double check it, remove your SD card, and list your block devices again:

    $ lsblk
    NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sda      8:0    0 238.5G  0 disk
    ├─sda1   8:1    0   255M  0 part /boot
    ├─sda2   8:2    0 230.4G  0 part /
    ├─sda3   8:3    0     1K  0 part
    └─sda5   8:5    0   7.9G  0 part [SWAP]
    sdb      8:16   0 223.6G  0 disk
    ├─sdb1   8:17   0   350M  0 part
    ├─sdb2   8:18   0 222.8G  0 part
    └─sdb3   8:19   0   450M  0 part
    sdc      8:32   0 931.5G  0 disk
    └─sdc1   8:33   0 931.5G  0 part /mnt/1tb

Take note of which drives exist with your SD card unplugged.   Now plug it back in and list your block devices one last time.   Notice in the sample above, my `sdd` is missing.   If it reappears after plugging it in and running the command again, we'll know that's the one we want to erase and install our image to.


    $ lsblk
	NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sda      8:0    0 238.5G  0 disk
    ├─sda1   8:1    0   255M  0 part /boot
    ├─sda2   8:2    0 230.4G  0 part /
    ├─sda3   8:3    0     1K  0 part
    └─sda5   8:5    0   7.9G  0 part [SWAP]
    sdb      8:16   0 223.6G  0 disk
    ├─sdb1   8:17   0   350M  0 part
    ├─sdb2   8:18   0 222.8G  0 part
    └─sdb3   8:19   0   450M  0 part
    sdc      8:32   0 931.5G  0 disk
    └─sdc1   8:33   0 931.5G  0 part /mnt/1tb
    sdd      8:48   1  29.9G  0 disk
    ├─sdd1   8:49   1   128M  0 part
    └─sdd2   8:50   1  29.8G  0 part

Notice that `sdd` is back!  That's our SD card.  On your system it will likely have a different name than mine, but you can use this process to identify it.   Take note of the name and you will use that in place of  `<sdx>` in the documentation below

Now that you've identified your SD card, let's get started with the install!

### Partition the SD card
Start fdisk on your SD card :

Note: Double check, and triple check that you're using the correct device.
    # fdisk /dev/<sdX>


* Type `o`. This will clear out any partitions on the drive.
* Type `p` to list partitions. There should be no partitions left.
* Type `n` for new, then `p` `for primary, `1` for the first partition on the drive, press `ENTER`  to accept the default first sector, then type ``+100M` for the last sector.
* Type `t`, then c to set the first partition to type `W95 FAT32 (LBA)`.
* Type `n` for new, then `p` for primary, `2` for the second partition on the drive, and then press `ENTER` twice to accept the default first and last sector.
* Write the partition table and exit by typing `w`.

### Create and mount the FAT filesystem to boot:
    # mkfs.vfat /dev/sdX1
    # mkdir boot
    # mount /dev/sdX1 boot

#### Create and mount the ext4 filesystem root:
    # mkfs.ext4 /dev/sdX2
    # mkdir root
    # mount /dev/sdX2 root

Create and change directory into a location on your disk that you can work from, download and install files.  

    $ mkdir archpi
    $ cd archpi


Download and extract the root filesystem (as root, not via sudo):
    $ su

Enter your root password, then choose ONE of the next following steps, depending on whether you have a Raspberry Pi, or Raspberry Pi 2.

##### Raspberry Pi

For the original Raspberry Pi, A, B, or Zero, use the following:

    # wget http://archlinuxarm.org/os/ArchLinuxARM-rpi-latest.tar.gz
    # bsdtar -xpf ArchLinuxARM-rpi-latest.tar.gz -C root
    # sync

##### Raspberry Pi 2

For the Raspberry Pi 2, use the following:

    # wget http://archlinuxarm.org/os/ArchLinuxARM-rpi-2-latest.tar.gz
    # bsdtar -xpf ArchLinuxARM-rpi-2-latest.tar.gz -C root
    # sync

After downloading and extracting the appropriate files, and waiting for them to sync:

Move boot files to the first partition:
    # mv root/boot/* boot

Unmount the two partitions:

    # umount boot root

Leave your superuser mode:

    # exit

Congrats!  You just installed the basic Arch Linux image on your SD card.  You can safely  remove the SD card from your host computer.

Insert the SD card into your Raspberry Pi, connect an ethernet cable to your Pi, and a router.

Apply 5V power to your Pi.

Use the serial console or SSH to the IP address given to the board by your router.

You may wish to google how to "port scan" your subnet, to locate which IP your Pi might be using.  Note the IP.

Login as the default user `alarm` with the password `alarm`
  OR
log in as  `root` with a password of `root`.

I woudl suggest logging in using the `root` user account first, to set everything else up.

# Login
-------------------
Connect keyboard, network, and monitor.  Log in as root.
-------------------
### Set timezone:

	# timedatectl set-timezone America/Los_Angeles


### Set hostname

		# hostenamectl set-hostname <name_your_pi_here>

### Upgrade Base System

    # pacman -Syyu

### Install some essential utils

    # pacman -S  sudo


###  Create new user

    	# useradd johnny

##### Set password for the new user

    	# passwd johnny

##### Create user home directory

    	# mkdir /home/johnny

##### Make user owner of the home directory

    	# chown johnny:johnny /home/johnny

##### Add user to sudoers

	# export EDITOR=vim
	# visudo


Edit the sudoers file, find the User privilege specification section and add user.

    ## User privilege specification
    root ALL=(ALL) ALL
    johnny ALL=(ALL) ALL

If you are not configuring WiFi now, you may skip to the last section "Get IP, Reboot.."


# Configure Wifi

This assumes that you have an appropriate USB WiFi Dongle for the Raspberry Pi, and it's plugged into one of the Pis USB ports.

### Always update Pacman before installing new packages

    # pacman -Syu


### Install wireless tools (if using wifi)
    # pacman -S dialog wpa_supplicant iw crda wireless-regdb

#### Set Regdomain

  This next step is necessary to define which frequencies your WiFi will work on, based on which country you're located in.   In the US, this is regulated by the FCC, and you DO NOT want to be caught using the inappropriate settings here, as you could earn yourself a huge fine from the federal government.   It's best to configure your regdomain, and not have to worry about it.

Edit `/etc/con.d/wireless-regdom`  and uncomment the proper regulatory domain.  These settings will apply on reboot.

    # nano /etc/con.d/wireless-regdom

Uncomment the appropriate setting for your country.  My example below shows the entry for the United States "US"

    #WIRELESS_REGDOM="UG"
    WIRELESS_REGDOM="US"
    #WIRELESS_REGDOM="UY"

### Enable wireless adapter

    $ sudo ip link set wlan0 up
    $ sudo wifi-menu

Follow the menus to configure your network and credentials. Note the name of the network profile you save (I generally use the name of the SSID) You'll need it in the next couple steps


###  make Wifi connect to default network
		$ netctl list

		$ sudo enable <network_profile_name>



		Wireless should connect automatically on next reboot.

### Get the IP, reboot,  and log in via SSH

		    # ip address
		    # reboot now

# Login via SSH using new user, and whichever network interface you choose!

		  $ ssh johnny@<your_pi_ip_address>
