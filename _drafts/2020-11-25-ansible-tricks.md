---
layout: post
title: Ansible tricks
tags:
  - DevOps
  - Ansible
  - Linux
aside: true
---

### ssh with different user

I usually put in the inventory which user I want to use during ansible ssh into the hosts.
Here you can specify the remote user name and also the location of the private key file.
In this case when you write `become: yes` this user will make changes in the remote host.

```
[db-hosts]
...

[all:vars]
ansible_user=myadmin
ansible_ssh_private_key_file=~/.ssh/myadmin-key
```

### Proxying any port to 80

Lately I needed to work in a very restricted environment, I could access only the 80 and 443 ports
of the remote hosts. A lot of cases it is not a problem but when I installed Couchbase I wanted to
access the web interface. So for that I redirected all requests landing on 80 to 8091.

```yaml
- name: Forward port 80 to 3000
  become: yes
  iptables:
    table: nat
    chain: PREROUTING
    in_interface: eth0
    protocol: tcp
    match: tcp
    destination_port: "80"
    jump: REDIRECT
    to_ports: "8091"
```
