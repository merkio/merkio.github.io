---
layout: post
title:  "Raspberry Pi 4 Start with USB boot"
date:   2021-01-03 17:25:38 +0100
categories: raspberry
---
# Headless Install

## Required things:

microSD card (> 4Gb)
downloaded raspberry pi image and unzip it \
(https://downloads.raspberrypi.org/raspios_arm64/images/) \
installed tool to flash the card (dd, Raspberry Pi Imager)

## Step 1
After download an image insert microSD card to your laptop. \
Using cli command lsblk find the name of the device \
`sudo lsblk` \
Here as example I would assume the name of the device /dev/sdb \
You can use dd command or GUI tool Imager to flash the card, \
here is the command to flash card with dd tool \
`sudo dd if=/home/user/Downloads/image.img of=/dev/sdb bs=1M status=progress`

## Step 2
Reattach card to the laptop and mount partitions \
Open boot partition and create a new empty file with name ssh

### Wifi connection
to connect to your wifi at first start create a wpa_supplicant.conf file in the boot partition with wifi settings.
```
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
scan_ssid=1
ssid="your_wifi_ssid"
psk="your_wifi_password"
}
```
## Step 3
Connect to the raspberry pi via ssh

Find the ip address
To find the address of the raspberry pi in your local network

Look at your dhcp server or router what IP address is used by raspberry pi. \
Or you can use nmap cli tool to scan your local network \
I would prefer to reserve this address for raspberry pi \
for the future to avoid problems with ip addresses. \
Use any ssh client to connect to the raspberry pi.
```
ssh pi@<ip address>
Default configuration for connection:
username: pi
password: raspberry
```
Also you can add your ssh key to the raspberry pi to connect without password. To do this you should add your public key to the authorized_keys into .ssh/authorized_keys in the pi home directory.

## Step 4
Update firmware \
`sudo apt update` \
`sudo apt full-upgrade` \
`sudo rpi-update`

Reboot

Update bootloader \
`sudo rpi-eerom-update -d -a`

Reboot

## Step 5
Use raspi-config tool to launch configuration tool \
`sude raspi-config` \
Change boot options
Boot ROM version => Latest version => No (Don't reset defaults) \
Select boot order => USB boot => reboot No

## Step 6
Switch off raspberry pi \
Copy sd card to the USB drive e.g. \
`sudo dd if=/dev/<sd card> of=/dev/<usb drive> bs=1M status=progress` \
after that you can using Disks or other tool change the size of the partitions in your USB drive

## Step 7
Connect the USB drive to the raspberry pi and boot.
