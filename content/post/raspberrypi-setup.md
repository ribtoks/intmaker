---
title: How do I setup Raspberry Pi
date: 2019-09-25T20:55:23+00:00
author: "Taras Kushnir"
image: raspberry-pi.jpg
categories:
  - Programming
keywords:
  - ads
  - raspberry
  - diy
  - pihole
aliases:
  - /2019/raspberrypi-setup
---

Somehow I end up breaking my Raspberry Pi all of the time. Today it was the sudoers file that I modified using `nano` instead of `visudo` and voila - I cannot be root anymore. Another time I simply forgot my user's password and apparently it's impossible to login via ssh anymore (I don't have spare physical screen and keyboard). So somehow it happens all of the time and I need to reimage SD card and setup everything from scratch. That's the reason I'm writing this note - to future myself.

<!--more-->

### Prepare SD card

#### Download raspbian image

First thing to do is to visit [official website](https://www.raspberrypi.org/) and download latest Raspbian image.

Then, verify hashsum using `shasum -a 256 2019-07-10-raspbian-buster-lite.zip` (under macOS).

#### Extract image from the archive

Run Unarchiver or 7zip and extract `.img` from `.zip`.

#### Flush image to SD card

For that you need to know SD card disk number.

```
diskutil list
diskutil unmountDisk /dev/disk2
sudo dd bs=1m if=~/Downloads/Installs/2019-07-10-raspbian-buster-lite.img of=/dev/rdisk2 conv=sync
```

Then, when it will be automounted, create empty file with name "ssh" in `/boot` (that will be the only mounted device).

### System setup

#### Initial

Now you can insert SD card to Raspberry Pi and turn it ON. I use network cable so I don't need to care about WiFi setup. It's much faster and much easier in my case.

```
ssh pi@192.168.1.170
sudo raspi-config
```

Go to "Advanced" and "Expand filesystem" - this is a first thing to do before downloading updates. Let's use all 32GB of the SD card instead of 2.2GB of the image we flushed.

Then I install [log2ram](https://github.com/azlux/log2ram) and modify the limits using `sudo vi /etc/log2ram.conf` to have at least 200MB of log space available. This is done to preserve the SD card from frequent writes.

Now it's a good idea to reboot to make use of log2ram.

#### Security

I use some of the tricks described in [security](https://www.raspberrypi.org/documentation/configuration/security.md) section. I mean changing the default user, using complex autogenerated password, restricting ssh access and things alike.

```
sudo adduser pihole
sudo adduser pihole sudo
```

Now it should be possible to reconnect on SSH with `pihole` user.

```
sudo deluser -remove-home pi
sudo vi /etc/sudoers.d/010_pi-nopasswd
```

Editing sudoers is required to disallow passwordless `sudo` for all users: just change `NOPASSWD` to `PASSWD` for your new user and save the file.

Also you may want to add use to group "video" so you will be able to read RaspberryPi temperature with command `/opt/vc/bin/vcgencmd measure_temp`. In order to do so, run `sudo usermod -aG video <username>`.

What I also usually do is I configure `fail2ban` in order to restrict ssh attempts with wrong password.

```
sudo apt install fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo vi /etc/fail2ban/jail.local
```

As for the last command - modifying the config - it's just enough to find `[ssh]` section and set `enabled = true` and `maxretry = 6`.

#### Update

It will be a good idea to install all possible updates

```
sudo apt-get update
sudo apt-get dist-upgrade
sudo apt-get clean
```

### Pi-Hole

Now when the base system is ready, we can install Pi-Hole using commands that you can find in [official GitHub repository](https://github.com/pi-hole/pi-hole).

```
curl -sSL https://install.pi-hole.net | bash
```

After going through configuration steps, everything what is left to do is to setup static DNS in your router.

I'd like to state that Pi-Hole by itself is not enough for your privacy and security, but it's a good start.

### Kanboard

In order to install kanboard on raspberry pi, first install php with friends

```
sudo apt-get install php php-gd php-mbstring php-xml php-sqlite3
```

and git if it's not installed `sudo apt-get install git`.

After this, extract fresh Kanboard in the `www/` directory:

```
cd /var/www/html/
sudo git clone https://github.com/kanboard/kanboard.git
cd kanboard
sudo git checkout v1.2.12
cd ..
sudo chown -R www-data:www-data kanboard/data
service apache2 restart
```

For kanboard I usually also install [kanboard tdg plugin]({{< relref "post/how-to-return-to-flow" >}}) as a task-management system.

```
cd kanboard/plugins
sudo wget https://github.com/ribtoks/kanboard-tdg-import/releases/download/v0.0.4/TdgImport-0.0.4.zip
sudo unzip TdgImport-0.0.4.zip -d TdgImport
```

Ultimately, run

```
sudo service apache2 restart
```

and navigate to `http://ip-of-rpi/kanboard/`, login with `admin:admin` to kanboard and configure users and projects.
