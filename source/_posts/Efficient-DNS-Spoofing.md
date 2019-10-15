---
title: Efficient DNS Spoofing
date: 2019-10-14 18:51:55
categories: Hacking
tags: [DNS spoofing, Bettercap, MITM]
keywords: [Bettercap, DNS spoofing, MITM]
description: If you seriously want to set others in your DNS spoofing trap, always remember don't let them detect anything suspicious. Bettercap is a good choice by the way.
---
There are a couple of DNS spoofing tools available on the Internet like arpspoof and ettercap. They are not convenient and robust enough as I expected. In fact, nobody wants to use such tools at the expense of low network speed, I mean, very very low speed that you cannot even open a single page most of the time as a victim. The reason is the bad performance of arpspoof. So bettercap will be a perfect proposal.

This is not a tutorial of bettercap, actually, I'm not familiar with many advanced features of it. Now let's get to know about the DNS spoofing in bettercap.

Install bettercap if you don't have it.
```shell
apt install bettercap
```
If you want to get some information about how to use bettercap, use the "help" command.

<img src="pic_2.png" width="100%" height="100%">

We use the DNS spoofing module here and modify the configuration.

<img src="pic_1.png" width="100%" height="100%">

Change the DNS server on the target host to our attacking machine.

<img src="pic_3.jpeg" width="50%" height="50%">

Create a new file named *readme.txt* on my attacking machine, and start up the python HTTP server for web page hijacking.

```shell
touch readme.txt
python -m http.server
```

Now, most of the work is done. Let's take a look at the effect. I visited *a.google.com:8000*.

<img src="pic_4.jpeg" width="60%" height="60%">

But we still have a problem here, how can we change all the DNS configurations in every single host within the same intranet? I have tried to add DNS forwarding in the OpenWrt router, but it didn't work, maybe I made some mistakes somewhere. So I have to try another stupid method which I mentioned in my previous post(https://recursively.review/2019/04/11/Get-Network-Traffic-of-Mobile-APPs/). Forwarding all the DNS requests to my attacking server. 

DNS requests use both TCP and UDP protocol, so we should make some changes to get it to work.

```shell
iptables -t nat -A PREROUTING -m iprange --src-range 192.168.2.2-192.168.2.253  -p tcp --dport 53 -j DNAT --to-destination 192.168.2.254:53

iptables -t nat -A PREROUTING -m iprange --src-range 192.168.2.2-192.168.2.253  -p udp --dport 53 -j DNAT --to-destination 192.168.2.254:53
```

So all the DNS requests traffic will be forwarded to my attacking machine without changing the DNS server in every single host.

<img src="pic_5.png" width="60%" height="60%">

we can browse other websites at the same time and nobody can detect anything abnormal.
