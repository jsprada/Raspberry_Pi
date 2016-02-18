# Raspberry Pi MPD Server (for Arch Linux)
This guide will help walk you through the way that I install and configure MPD (Music Player Daemon - http://www.musicpd.org/) on an Arch Linux based Raspberry Pi.

by Johnny Sprada.

https://github.com/jsprada/Raspberry_Pi

These instruction assume that you have already installed Arch Linux on your device.   I have a guide at the link above that you can use to do this, if you haven't already.   For the sake of simplicity, these examples use `nano` text editor, as it's simple to learn, and included with the base installation.

Log in to your Raspberry Pi to perform the following work.

### Install Necessary Packages

#### Update Packages
As the "Arch Way" dictates, it's best to always update your system before installing new packages.

    $ sudo pacman -Syu


After everything updates, you may begin your install.


#### General Audio Packages


    $ sudo pacman -S alsa-utils alsa-firmware alsa-lib alsa-plugins alsa-tools


#### Install MPD, MPC:

MPD is the server software.  MPC, and NCMPCPP are clients that will allow you to interact with the MPD server.   I suggest installing these client packages as an easy way to control your music.  You may replace them with other client software in the future, if your project requires it (I will explain how I do this in a future guide).

    $ sudo pacman -S mpd mpc ncmpcpp


### Test Audio

##### Install wget to fetch a sound file off the internet

  $ sudo pacman -S wget

##### Get sound file from the Internet

Let's download a small audio file off the Internet,  just so we have a way to play and test the sound.

      $ wget http://www.wavsource.com/snds_2015-09-20_4380281261564803/sfx/boxing_bell_multiple.wav


##### Play sound

Play the sound we just downloaded.   You may return to using this command anytime you need to perform an audio output check.

    $ sudo aplay ~/boxing_bell_multiple.wav


If the sound did not play, the sound driver needs to be loaded.
Load the stock drivers,  just to get the system working.  We'll load the hifi drivers for the DAC  later.   The purpose of this guide is to simply get MPD up and running, we'll get the high end audio configured later.

##### Add sound drivers (lo-fi bulit-in)
Create a file called `/etc/modules-load.d/snd_bcm2835.conf`.

    $ sudo nano /etc/modules-load.d/snd_bcm2835.conf

Add the following line to it


    # Built-in RaspPi PWM Audio (low quality)
    snd_bcm2835


##### Load sound driver

    $ modprobe snd_bcm2835

##### Choose default sound card

Launch Alsamixer.

    $ sudo alsamixer


In Alsamixer, press F6, and choose the driver of your choice (bcm2835 for now)
Hit Esc. to exit

    $ sudo alsactl store





### Configure MPD

Add the mpd user to the audio group

    $ sudo gpasswd -a mpd audio

####  Edit /etc/mpd.conf

    $ sudo vim /etc/mpd.conf


    music_directory         "/var/lib/mpd/usb/music"
    db_file                 "/var/lib/mpd/database"
    log_file                "/var/lib/mpd/log"
    playlist_directory      "/var/lib/mpd/playlists"
    pid_file                "/var/lib/mpd/pid"
    state_file              "/var/lib/mpd/state"
    sticker_file            "/var/lib/mpd/sticker.sql"
    auto_update                     "yes"
    user                            "mpd"



    audio_output {
            type            "alsa"
            name            "MPD ALSA"
            mixer_type      "software"
            mixer_device    "default"
            mixer_control   "PCM"
    }


  Create the config files needed for MPD

        # sudo touch /var/lib/mpd/{database,log,pid,state,sticker.sql}

  make the mpd user the owner of the entire location
      $ sudo chmown mpd:mpd  -R /var/lib/mpd

### Start MPD

    $ sudo systemctl start mpd

Check the status, see if anything's awry

    $ sudo systemctl status mpd

Autostart MPD on boot

    $ sudo systemctl enable mpd


## Mount USB stick with music on it.

This step is optional, based on your application.  For my project, I want to store all of my music on a USB stick,  and save the SD card space for system files.   This allows me to move the USB stick to another computer to load it with music, then return it to the Raspberry Pi to play.  

I formatted a USB stick and created a directory called music on it.  Inside this directory, I copied my music directories by artist.


Check the existing disk configuration with `lsblk`:
    $ lsblk
        NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
        mmcblk0     179:0    0 14.9G  0 disk
        |-mmcblk0p1 179:1    0  128M  0 part /boot
        `-mmcblk0p2 179:2    0 14.7G  0 part /

Insert USB stick, and run `lsblk` again:

        $ lsblk
        NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
        sda           8:0    1 29.8G  0 disk
        mmcblk0     179:0    0 14.9G  0 disk
        |-mmcblk0p1 179:1    0  128M  0 part /boot
        `-mmcblk0p2 179:2    0 14.7G  0 part /

Note the difference between the results of each instance of lsblk.
In my example sda is the USB stick, it's located at /dev/sda

Create a mount point

        $ sudo mkdir /var/lib/mpd/usb

Mount the USB stick
        $ sudo mount /dev/sda /var/lib/mpd/usb


####  Add mount point to fstab, to automount on boot.

Note, if you are planning to use more than one USB stick in your project, for whatever reasons, it's probably best to mount these using the UUID rather than serial disk name.   Otherwise, your Pi could mix up the memory sticks and mount them to the wrong mount points.   I'd be happy to create a guide to demonstrate auto mounting by UUID, if you'd like (feel free to contact me and request it, if that's the case.)

Edit /etc/fstab as root

        $ sudo vim /etc/fstab

Add the folowing line:

        /dev/sda        /var/lib/mpd/usb        ext4    defaults,noatime        0   0


Reboot the Pi, and verify that it mounted the USB stick correctly.  It might be a good idea to test the audio again, just to be sure the settings stuck.

After adding music to your system, you'll want to tell MPD to update its database.  Assuming you installed the MPC client,  you can use this simple command to do so:

    $ mpc update



Test it!  (google how to use these)

    $ ncmpcpp

    $ mpc
