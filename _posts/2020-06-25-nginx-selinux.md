---
layout: post
title:  "Adventures with Nginx and SELinux"
tags:
  - DevOps
  - Linux
  - SELinux
aside: true
---
Well SELinux is very useful additional security layer in modern Linux systems but until we don't understand what it really does and provide, it can be a real pain in the neck. Sometimes we just want to install something and get it working, but we run into strange errors. What can we do? I will write my journey with nginx and hope it helps you.

### Why is there SELinux?

In Linux without any security additions we really need to know what we allow and what we deny. Sometime tools by default gives the users too much permissions and we don't even know about them. In NSA they made a set of recommendations and added it to an extension which can be in three mode: enforcing, permissive and disabled. By default it enforces the recommendations. But you can always ask with `sestatus`.

### nginx

Let us install nginx and set up some proxy mode. Also let us set up nginx to serve a CIFS mounted directory. For the sake of simplicity just add this to the default `/` location in `/etc/nginx/nginx.conf`.

```conf
location /content/ {
    proxy_pass http://10.1.0.8/;
}
```

Reload nginx and just hit the endpoints inside the content path. You will notice permission errors and when you go to nginx error logs you will see errors like this

```
*8 connect() to 10.181.20.1:80 failed (13: Permission denied) while connecting to upstream
```

The reason is that SELinux by default doesn't allow http server to send requests to another host. With `getsebool -a` you can check all the options SELinux has.

```
$ getsebool -a | grep ^http
...
httpd_can_network_connect --> off
...
```

So `httpd` cannot connect to the network. It sounds good, but how the hell we can find what SELinux doesn't like? The answer is the audit log, just check this log and you will exactly see what is going on.

```
type=AVC ...: avc:  denied  { name_connect } ... comm="nginx" dest=80 ...
type=SYSCALL ...: ... success=no exit=-13 ... exe="/usr/sbin/nginx" \
  subj=system_u:system_r:httpd_t:s0 key=(null)^]ARCH=x86_64 SYSCALL=connect
```

You can change it permanently with

```
setsebool -P httpd_can_network_connect=on
```

### CIFS mount

The same happens when you want to serve a mounted directory. In my case I just wanted to share a CIFS mounted directory with nginx, I ran into the same permission denied problem. Then I had more experience so I listed all the httpd options and I found the `httpd_use_cifs` which was obviously `off`. So I just needed to set it to `on`. So

```
setsebool -P httpd_use_cifs=on
```
