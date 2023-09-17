---
layout: post
title: Linux Manual Privilege Escalation
titledate: 09/17/23
tags: ["infosec", "info_gathering", "offsec", "privesc"]
---

#### linux enumeration

    $ id

tell us the current user, uid, and gid

    $ cat /etc/passwd

give us all users as well as potential installed services with users like www-data and sshd

    $ hostname

gives us the system hostname

    $ uname -a

    $ cat /etc/os-release

    $ cat /etc/issue

give us system info

    $ ps aux can list running processes

display networking interface information

    $ ipconfig 

    $ ip

print routing information 

    $ route

    $ routel

may be able to gather iptables information

    $ cat /etc/iptables/rules.v4

list linux scheduled tasks with cron

    $ ls -lah /etc/cron*

system admins may add task to the crontab

    $ sudo crontab -l

list installed packages of package manager

    $ dpkg -l

searching writeable directories

    $ find / -writable -type d 2>/dev/null

list mounted drives

    $ cat /etc/fstab

    $ mount

list all available disks

    $ lsblk

list driver and kernel modules

    $ lsmod

to list more info about a specific module

    $ /sbin/modinfo libata

### NOTE: if we can bypass a setuid root program to call a cmd of our choice we can impersonate the root user and gain equivalent access

    $ find / -perm -u=s -type f 2>/dev/null

search for SUID marked files

    $ cat binary |strings|grep 'word'

#### automating enumeration

we can use the tool /usr/bin/unix-privesc-check on our box to look for system misconfiguration

we can try both standard and detailed mode - copy the script over the system and dump it to a file

    ./unix-privesc-check standard > output.txt

some other tools that can check linux systems are LinEnum and LinPeas