---
layout: post
title: "Resolving Docker Port 4243 Conflict with CrashPlan"
date: 2014-02-18 15:17:32 +0800
comments: true
categories: 
---
Hit a little snag while initializing [boot2docker](https://github.com/steeve/boot2docker):

``` console
$ boot2docker init
[2014-02-18 12:34:56] DOCKER_PORT=4243 on localhost is used by an other process! Change the port to a free port.
```

lsof shows it's [CrashPlan](http://www.code42.com/crashplan/) conflicting:

``` console
$ sudo lsof -i :4243
COMMAND     PID USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
CrashPlan 20270 root   79u  IPv4 0x3c409a82447e9111      0t0  TCP localhost:4243 (LISTEN)
```

Seems like [Docker](https://www.docker.io/) will be [changing their default port](https://github.com/dotcloud/docker/issues/1440) soon. In the meantime, here's how to change CrashPlan backup service's port on OS X:

{% gist_no_css 9065790 change_crashplan_backup_service_port.sh %}