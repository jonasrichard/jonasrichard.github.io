---
layout: post
title:  "Install and setup SE linux"
tags:
  - DevOps
  - Linux
aside: true
---

```bash
ls -Z file
yum install -y policycoreutils-python-utils
yum install -y policycoreutils-devel
sepolicy manpage -a -p /usr/share/man/man9
mandb
man -k selinux
```
