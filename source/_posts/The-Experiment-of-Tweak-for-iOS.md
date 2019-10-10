---
title: The Experiment of Tweak for iOS
date: 2018-11-20 20:34:02
categories: Reverse Engineering
tags: [iOS, Theos]
description: Thanks to the release of Unc0ver suite which allows jailbreak on iOS11 and even on iOS12. I have the opportunity to try out the experiment of tweak on the iOS11 device. It's not complicated as I expected and meanwhile, interesting.
---
## Tweak for what?
You can make a choice of whatever you want to add a tweak on iOS. For me, I started with the SpringBoard of iOS. So, what is SpringBoard? SpringBoard is the application that manages the home screen on iOS devices. Essentially SpringBoard is like the mobile version of a desktop. Mac OS X features the Finder while Windows computers have the Explorer. And what does the tweak affect? This tweak works when the user triggers a respring(A respring restarts the user interface (SpringBoard) of the iOS operating system. The main difference between a restart and a respring is that a respring doesn’t switch off the system.).

## Get the environment ready
The framework I used during the tweak development is Theos(https://github.com/theos/theos), an efficient and powerful framework. It's simple to clone the project and execute the _chmod_ directive, so I omit that here and come to the steps different from the old version of Theos.

Install dpkg and ldid which is used to sign your package instead of codesign in Xcode.
```shell
brew install dpkg ldid
```
If you don't have Homebrew, you just need one command to get it and then you're good to go.
```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
The operation of _sudo /opt/theos/bin/bootstrap.sh substrate_ is not needed with the latest version of Theos. When everything is done, remember to set the 
environment variables:
```shell
export THEOS=/opt/theos
export PATH=/opt/theos/bin/:$PATH
export THEOS_MAKE_PATH=$THEOS/makefiles
```
All the preparations have been finished, we can now dive into the interesting section.

## Functions hooking
I post the final result appears on my device here:
<img src="pic_1.png" width="60%" height="60%">
Apple has given many APIs for AppStore developers, but it's not enough compared to the mammoth APIs which can be exposed on the jailbroken device. When it comes to developing tweaks, it's actually changing the behavior by hooking functions. But it's not easy to find out how the functionality implemented among the code. In fact, it takes lots of time to figure out the logic of the substrate. I just implement the common work supplied by other people.

### Generate a template.
Type nic.pl and choose an option from the given list. We want to generate a tweak template, so input 13. Then finish the following information.
<img src="pic_2.png" width="100%" height="100%">
When you see the output of "Done.", there will be 4 files generated under your working directory: 
```shell
Makefile    commonproject.plist    Tweak.xm    control
```

### Modify files as you need
_Makefile_ is generally used in most projects to get everything done properly. In our project, it used to point out files, libraries and frameworks we need.
```shell
THEOS_DEVICE_IP = 10.1.2.34
ARCHS = armv7 arm64
TARGET = iphone:latest:8.0
include $(THEOS)/makefiles/common.mk

TWEAK_NAME = commonproject
commonproject_FILES = Tweak.xm
commonproject_FRAMEWORKS = UIKit

include $(THEOS_MAKE_PATH)/tweak.mk

after-install::
        install.exec "killall -9 SpringBoard"
```
We write our code about functions hooking and other useful snippets in the  _Tweak.xm_ file.
```shell
%hook SpringBoard
  
// Hooking an instance method with an argument.
- (void)applicationDidFinishLaunching:(id)application {

        %orig; // Call through to the original function with its original arguments.
        UIAlertView *alert = [[UIAlertView alloc]initWithTitle:@"此广告位常年招商" message:nil delegate:self cancelButtonTitle:@"OK"otherButtonTitles:nil];
        [alert show];
        [alert release];

        // If you use %orig(), you MUST supply all arguments (except for self and _cmd, the automatically generated ones.)
}

// Always make sure you clean up after yourself; Not doing so could have grave consequences!
%end
```
The _control_ file contains the basic information of your deb package, all of them will be packed in your deb package.
```shell
Package: apple
Name: commonproject
Depends: mobilesubstrate
Version: 0.0.1
Architecture: iphoneos-arm
Description: iOS tweak learning.
Maintainer: z
Author: z
Section: Tweaks
Homepage: http://recursively.review
```
The *.plist file contains the configuration of your package.
```shell
{ Filter = { Bundles = ( "com.apple.springboard" ); }; }
```
### Install your package
Next, we need to install our package onto the iOS device with directive _make package install_ remotely through the ssh. But firstly, you should have installed OpenSSH. You need to input your password of ssh twice during the installation process. If no errors prompt out you can respring your iOS device and easily see the result I've shown above.

## Sources
_iOS App Reverse Engineering_