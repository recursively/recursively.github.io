---
title: Accelerate Services Blacklisted by Gateway through CDN
date: 2021-04-13 16:30:52
categories: Network
tags: [CDN, Cloudflare]
keywords: [CDN, Cloudflare, vmess, v2ray, websocket, shadowsocket]
description: There are some useful methods to accelerate services that are blacklisted by the gateway, CDN is one of the most efficient methods. The reasonable combination of the CDN and protocol will make your services robust from the gateway blacklist.
---
## Packet Flow

The premise for using CDN to bypass the gateway blacklist is that the gateway will never block the traffic from CDN. According to the different CDN providers, there are basically three packet flows:

<img src="pic_1.png" width="100%" height="100%">

This is the optimal status that both inside and outside of the gateway have CDN nodes, which can make sure the IP address of the server will not be detected by the gateway.

<img src="pic_2.png" width="100%" height="100%">

It will be still OK if the CDN provider doesn't have any CDN node inside the gateway, all the requests to server will be handled by CDN and the real IP address won't be exposed.

<img src="pic_3.png" width="100%" height="100%">

It's impossible to bypass the gateway if the CDN provider doesn't have any node outside the gateway, the real IP address will be exposed on the Internet before the packet goes outside the gateway.

## Set Up Your Domain Name and Cloudflare CDN

A domain name is necessary if you want to use any CDN services. By the way, anyone can get a free domain name from [Freenom](https://www.freenom.com/).

If you already have a domain name, make sure you have changed the name server from the original server to the Cloudflare name server.

<img src="pic_4.png" width="100%" height="100%">

Next, you need to register a Cloudflare account and add your domain name.

<img src="pic_5.png" width="60%" height="60%">

Now we have come to the crucial step, configure your DNS records. Modify your root domain or any subdomains you like to point to the IP address of your server in the A record field.

<img src="pic_6.png" width="100%" height="100%">

Remember to enable the proxy and make sure the icon is orange, it's disabled if the color is grey.

<img src="pic_7.png" width="60%" height="60%">


## Modify The Protocol Configuration

This is an important part of whether your service can run normally. There's something we must be aware of is that the CDN protocol of Cloudflare only supports packet which has a Host header in the web requests, such as HTTP and Websocket. Non-web protocols like FTP and SSH don't include a HOST header, Cloudflare will not know how to route those packets. Besides, Cloudflare only supports some common ports like 80 and 443, for the other ports, you could check the official website to make sure they are available.

For the vmess protocol, it can be simply disguised as HTTP or Websocket.

For the server-side configuration, modify the *inbounds* field:

```json
    "protocol": "vmess",
    "port": 80,
    "streamSettings": {
        "network": "ws",
        "wsSettings": {
            "path": "/"
        }
    }
```

For the client-side configuration, change the address to your domain name, and switch the network to Websocket(ws), don't forget to fill in the *path* field.

```json

```

## Pros and Cons

The advantage is obvious. What can be confirmed is that the CDN is rarely banned because of so many services running on it. Any filtering operation by the gateway will cause a bunch of services not accessible unintentionally. The robustness of CDN can be guaranteed. Even if your server IP has been banned by the gateway, your services running on your server are still accessible.

The disadvantage cannot be ignored. The network latency will be high, in fact, sometimes the connection speed is really slow. 

pic

It's not suggested to apply this proposal for some services that require high performance and low latency.