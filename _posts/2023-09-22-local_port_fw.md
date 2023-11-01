---
layout: post
title: Local Port Forwarding
titledate: 09/22/23
tags: ["infosec", "offsec", "tunnel", "smbclient"]
---

#### local port forwarding

in the case we cannot run socat we can create a connection that goes

Kali -> target1 -> DMZpsqltarget -> newtarget

assuming we have python3 installed - from our target1 machine we can use python to create a connection to the DMZpsqltarget 2 machine

    $ python3 -c 'import pty; pty.spawn("/bin/bash")'

    $ ssh database_admin@10.12.1.1

now we log into the psql machine via ssh with the credentials that we discovered previously from the psql database

we can see what machines are connected to it with 

    $ ip addr

and discover an additional subnet , with nc available we can do some ping sweeps

    $ for i in $(seq 1 254); do nc -zv -w 1 172.16.1.$i 445; done

if we discover a host that is up we can setup a local port forward to relay connections

with openssl -l - IPADDRESS:PORT:IPADDRESS:PORT

    target1: ssh -N -L 0.0.0.0:4455(target1/confluence machine):172.16.1.217:445(discovered smb machine) database_admin@10.4.50.215(psql machine)

    ssh -N -L 0.0.0.0:4455:172.16.241.217:4242 database_admin@10.4.241.215

here we create an ssh connection with -N to have no shell listening on all interfaces of our target1 machine and passing all packets through our psql system

so now we need another reverse shell on another listening port to our target1/confluence machine so that we can interact with it

to get this reverse shell we just modify the original confluence exploit to use a different port

once we have this shell we can confirm the ssh port is open and listening 

    target1: ss -ntplu

now we can enumerate the smb machine that is only discovered through the psql machine

    $ smbclient -p 4455 -L //192.168.1.1/ -U hr_admin --password=Welcome1234

above we provide credentials we found when cracking the psql database

