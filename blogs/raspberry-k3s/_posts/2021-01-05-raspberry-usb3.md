---
layout: post
title:  "If you have a Raspberry Pi 4 and are getting bad speeds transferring data to/from USB3.0 SSDs, read this"
date:   2021-01-05 17:25:38 +0100
categories: raspberry
---
## If you have a Raspberry Pi 4 and are getting bad speeds transferring data to/from USB3.0 SSDs, read this
### The most common symptoms of a misbehaving UAS device are

Extremely slow performance - in the kilobytes per second range \
Frequent disconnects-reconnects of the device with the desktop repeatedly displaying the "removable medium inserted" dialogue box \
The kernel message log (dmesg) reports errors relating to a UAS device that look like this: \
Code: Select all
```
[ 501.594683] usbcore: registered new interface driver usb-storage
[ 501.599729] scsi host6: uas
[ 501.599800] usbcore: registered new interface driver uas
...
[ 573.203294] sd 6:0:0:0: [sda] tag#29 uas_eh_abort_handler 0 uas-tag 9 inflight: CMD OUT
[ 573.203302] sd 6:0:0:0: [sda] tag#29 CDB: Write(10) 2a 00 00 4f a0 00 00 04 00 00
[ 573.205063] sd 6:0:0:0: [sda] tag#28 uas_eh_abort_handler 0 uas-tag 10 inflight: CMD OUT
[ 573.205070] sd 6:0:0:0: [sda] tag#28 CDB: Write(10) 2a 00 00 4f a4 00 00 04 00 00
[ 573.208537] sd 6:0:0:0: [sda] tag#27 uas_eh_abort_handler 0 uas-tag 6 inflight: CMD OUT
...
[ 573.269992] scsi host6: uas_eh_device_reset_handler start
[ 573.393710] usb 2-4: reset SuperSpeed Gen 1 USB device number 2 using xhci_hcd
[ 573.414256] scsi host6: uas_eh_device_reset_handler success
```
These errors may also appear due to poor power quality or overloading the Pi's maximum 1.2A 
downstream USB port current, but if they persist when using a powered hub then they are genuine UAS issues.

All UAS drives must support mass-storage as a fallback option. \
The kernel can be told to ignore the UAS interface of a device and just use mass-storage - \
the usb-storage driver has a "quirks" option for this purpose. As UAS is built-in to the kernel \
to allow the root filesystem to be installed on an SSD, the quirk needs to go into cmdline.txt as a module parameter. \
This parameter matches the USB Vendor ID (vid), Product ID (pid) and overlays the specified quirks that disable specific features for this device.

#### 1. Finding the VID and PID of your USB SSD
   Disconnect the USB SSD. In a terminal window, run the command sudo dmesg -C. \
   Now, plug in the SSD and run dmesg with no parameters. \
   You should get output that looks like this:
```
[ 4096.609817] usb 2-1: new SuperSpeed Gen 1 USB device number 4 using xhci_hcd
[ 4096.646369] usb 2-1: New USB device found, idVendor=2109, idProduct=0715, bcdDevice=a0.00
[ 4096.646385] usb 2-1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[ 4096.646397] usb 2-1: Product: SABRENT
[ 4096.646409] usb 2-1: Manufacturer: SABRENT
[ 4096.646421] usb 2-1: SerialNumber: 000000123AD2
[ 4096.655154] scsi host0: uas
[ 4096.669178] scsi 0:0:0:0: Direct-Access SABRENT 2210 PQ: 0 ANSI: 6
[ 4096.670993] sd 0:0:0:0: Attached scsi generic sg0 type 0
[ 4096.673710] sd 0:0:0:0: [sda] 234441648 512-byte logical blocks: (120 GB/112 GiB)
```
The idVendor and idProduct are the two hexadecimal numbers you need to take a note of.

##### 1a. Multiple SSDs
If you have multiple USB SSD devices plugged into a single Pi 4, \
then for each device experiencing issues repeat Step 1 above and make a note of each idVendor and idProduct pair. \
For multiple devices with different VID:PID pairs, expand the parameter \
with a comma between each vid:pid:u triplet like this: usb-storage.quirks=0123:4567:u,2109:0715:u.

Save the file and exit the editor.

#### 2. Add the quirks to /boot/cmdline.txt
   Run a text editor as root - sudo nano /boot/cmdline.txt from the console or sudo leafpad /boot/cmdline.txt from the desktop.
   At the start of the line of parameters, add the text usb-storage.quirks=aaaa:bbbb:u where aaaa is the idVendor \
   for your device and bbbb is the idProduct. \
   So, with the device above the string will be usb-storage.quirks=2109:0715:u.

#### 3. Reboot.
#### 4. Check that it worked
   To check that the quirk has been applied successfully, run dmesg | grep usb-storage and check that the VID and PID is listed as having a quirk applied:
   Code: Select all
```
[ 2.495725] usb 2-1: UAS is blacklisted for this device, using usb-storage instead
[ 2.512739] usb 2-1: UAS is blacklisted for this device, using usb-storage instead
[ 2.531823] usb-storage 2-1:1.0: USB Mass Storage device detected
[ 2.549642] usb-storage 2-1:1.0: Quirks match for vid 2109 pid 0715: 800000
[ 2.566177] scsi host0: usb-storage 2-1:1.0
```
Typically, most drives are still performant with usb-storage. \
They may not be able to saturate a USB3.0 connection but should still get 150-200MB/s under most workloads.
