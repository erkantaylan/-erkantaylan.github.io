---
layout: post
title: My Port is Always Used
date: 2024-02-17 19:22:00
description:
tags: windows network
categories: sample-posts
---

TL-DR; [Found the solution in here](https://superuser.com/a/1729731)

In windows I was having problem with my port 80 and 443 alongside with 8080, these 3 ports were always used somehow. So a way to check which process is your port

```
netstat -aon | findstr :80
```

Above code gives you something like this, (I give you the headers as well, but you won't have it when you run it.)

```
  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:8080           0.0.0.0:0              LISTENING       18476
  TCP    192.168.1.76:59686     192.229.221.95:80      TIME_WAIT       0
  TCP    192.168.1.76:59692     34.107.221.82:80       ESTABLISHED     11452
  TCP    192.168.1.76:59713     34.107.221.82:80       ESTABLISHED     11452
  TCP    [::]:8080              [::]:0                 LISTENING       18476
```

Since I already fix the problem you will have different results than this, also none of the Local Address is actually 8o port, findstr works like grep checking lines with text 80 in it so keep in mind.

You can go to task manager and check which process has that PID but you can also do it in cmd as well.

```
tasklist /fi "pid eq {PID}"
```

Result will be something like this

```
Image Name                     PID Session Name        Session#    Mem Usage
========================= ======== ================ =========== ============
firefox.exe                  11452 Console                    2    449,036 K
```

My problematic process was `iphlpsvc.exe` for some reason OS itself was locking my ports.

After some research, apparently I, myself (or some process but I kind of had some flashbacks that writing some commands) told the OS to lock it but forgot about it too.

So to find which ports are `reserved` you need to write this command as admin (I think you don't need, I just tried it but the link says otherwise. ¯\_(ツ)\_/¯ )

```
netsh interface portproxy show all
```

Result

```
Listen on ipv4:             Connect to ipv4:

Address         Port        Address         Port
--------------- ----------  --------------- ----------
0.0.0.0         22          172.22.178.69   2020
```

Right now it is only port `22` but I used to have 80, 443, 8080 as well.

Finally, the way you remove is

```
netsh interface portproxy delete v4tov4 listenport={Listen Port which is 22 in the sample} listenaddress={Listen Address on the Left Side which is 0.0.0.0}
netsh interface portproxy delete v4tov4 listenport=22 listenaddress=0.0.0.0
```

Another thing is `v4tov4` When you write `netsh interface portproxy delete help` you will see below result so if you have more complicated problem change `v4tov4` to your needs.

```>netsh interface portproxy delete help

The following commands are available:

Commands in this context:
delete v4tov4  - Deletes an entry to listen on for IPv4 and proxy connect to via IPv4.
delete v4tov6  - Deletes an entry to listen on for IPv4 and proxy connect to via IPv6.
delete v6tov4  - Deletes an entry to listen on for IPv6 and proxy connect to via IPv4.
delete v6tov6  - Deletes an entry to listen on for IPv6 and proxy connect to via IPv6.

```
