---
title: Bridge and Router Working Mode on OpenWrt
date: 2020-11-13 06:18:21
categories: IoT
tags: OpenWrt
keywords: OpenWrt
description: This post briefly demonstrates the configuration of bridge and router working mode of OpenWrt. 
---
## Bridge mode

Bridge mode allows your OpenWrt device connects to the LAN network as a bridged AP. 

<img src="pic_1.png" width="60%" height="60%">

You can use this device to extend your current router or even just plays the role of a gateway which handles all the traffic flows through your local network. More details can be found in this OpenWrt official page: https://openwrt.org/docs/guide-user/network/wifi/bridgedap

## Router mode

The router mode will be easier to understand. It's more of a new router device added into the current local network. This router has it's own LAN and WAN interface. And it's LAN interface works on a different layer which controls by the NAT rules.

<img src="pic_2.png" width="60%" height="60%">

You can also go to the OpenWrt official page to see how it works: https://openwrt.org/docs/guide-user/network/routedclient

## Compile your own packages

In case of there are some packages are not compatible with your kernel or it doesn't exist in your package list. You have to compile them from the source by yourself.

clone the whole project and follow the compile tutorial to config properly. You can edit your configuration conveniently through menuconfig. Assume that I need to build the drivers for my network adapter, I should choose the compatible driver in the list.

pic

