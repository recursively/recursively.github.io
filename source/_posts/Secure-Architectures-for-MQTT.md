---
title: Secure Architectures for MQTT
date: 2020-10-15 10:33:18
categories: IoT
tags: MQTT
keywords: [MQTT, IoT]
description: MQTT has a basic authentication mechanism as a universal IoT transmission protocol. We can basically use a username and password to achieve that. But as the number of devices increases, the risk of password leakage increases. Maybe we can design different kinds of architectures to prevent the possible risk. Even more, we can use different architectures in complex situations.
---

For most cases, we use one broker to forward the MQTT topics like the chart below. 

<img src="pic_1.png" width="60%" height="60%">

All topics that publishers have published are accessible for other people who can pretend to be the subscribers. Imagine that we have published some topics to the MQTT broker including some sensitive topics which we don't want other people to access. Or if we use username and password to authenticate separately on each IoT device. The passwords may be leaked in some way. It's necessary to store the passwords on the trusted devices. The chart may help you understand. 

<img src="pic_2.png" width="60%" height="60%">

Assume that we have already got one trusted device, so we can start to set up the environment step by step. We need to startup two MQTT brokers. All outcome subscribers' topics should be sent to broker2, and the other one used to provide service for publishers. So we can set our rules between these two brokers. Firstly generate your password file, you can set the user name and password whatever you like, here I set my user name and password to forward/forward123.
```shell
sudo mosquitto_passwd -c /etc/mosquitto/passwd forward
Password: forward123
```
The password file is used by the broker which provides access control service. Now we are modifying the configuration of this broker. Use the listener to specify the address and port.
```ini
listener 1883 127.0.0.1
```
Then include the password file and specify the acl file, before that you should make sure the allow_anonymous option is false.
```ini
allow_anonymous false
password_file /etc/mosquitto/passwd
acl_file /etc/mosquitto/acl
```
I gave the configuration file name "mosquitto.conf".

Now create the acl file and add some rules to it. Note that there are three types of permissions: read, write, and readwrite. You can also use wildcards in the acl file.
```ini
user forward
topic readwrite mqtt/#
```

For the broker which used to receive subscriber's requests, you need to specify the address and port and apply the bridge function properly. I named this configuration file "mosquitto.conf2".
```ini
listener 1883 172.20.1.92
```
```ini
connection bridge-01
address 127.0.0.1:1883

remote_username forward
remote_password forward123

topic # out 0
topic # in 0
```
Now all of the environment settings are ready, we can run these two brokers separately:
```shell
sudo mosquitto -c mosquitto.conf
```
```shell
sudo mosquitto -c mosquitto.conf2
```
We can simply check if it's working well by creating a subscriber here:
```shell
mosquitto_sub -h 172.20.1.92 -t "mqtt/test"
```
And we create a publisher to push messages to the topic:
```shell
mosquitto_pub -h 127.0.0.1 -t "mqtt/test" -m "Hello mqtt" -u "forward" -P "forward123"
```
If everything set properly, you should notice that there's message appears in the subscriber's prompt:
```plain
mqtt/test Hello mqtt
```
Try changing the topic to another one that's different from the "mqtt" prefix or try changing the password in the *passwd* file, and restart these two brokers. It's expected that the subscriber will not receive any messages sent from the publisher.

Somehow, you can also allow anonymous access to brokers although it's a little bit unsafe. That will be much easier to set up the environment. For the broker which is accessible to subscribers, append this content to the configuration file and name it "mosquitto.conf2".
```ini
listener 1883 172.20.1.92
```
```ini
connection bridge-01
address 127.0.0.1:1883

topic # out 0
topic # in 0
```
For the configuration of the other broker, append these content and name it "mosquitto.conf".
```ini
listener 1883 127.0.0.1
```
```ini
acl_file /etc/mosquitto/acl
```
Then modify the acl file:
```ini
topic readwrite mqtt/#
```
Now let's have a test. Start both brokers and execute commands as subscriber and publisher separately.
```shell
sudo mosquitto -c mosquitto.conf
```
```shell
sudo mosquitto -c mosquitto.conf2
```
```shell
mosquitto_sub -h 172.20.1.92 -t "mqtt/test"
```
```shell
mosquitto_pub -h 127.0.0.1 -t "mqtt/test" -m "Hello mqtt"
```
If everything goes well, the subscriber will receive the message "Hello mqtt". Then we can subscribe another topic different from "mqtt/#" as a subscriber, and send the message as a publisher again. You can find that the broker will block the subscriber to receive the message.