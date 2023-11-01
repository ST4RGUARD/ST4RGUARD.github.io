---
layout: post
title: Dynamic Port Forwarding
titledate: 09/22/23
tags: ["infosec", "offsec", "tunnel", "proxychains", "smbclient"]
---

#### moar power

the benefit of dynamic pfw is that the listening socks port can forward incoming ssh traffic to any machine the ssh server can route to

- attacker -> target1 -> psql db -> target2
                                 |
                                  -> target3
                                 |
                                  -> target4

instead of only being able to reach target 2 from psql, if this machine has target 3&4 in its routes then we can communicate with them to

first we need to ensure we use tty by using pythons pty module

    target1$ python3 -c 'import pty; pty.spawn("/bin/bash")'

we use the Dynamic option

    target1$ ssh -N -D 0.0.0.0:9999 database_admin@10.4.1.215

we will use the tool proxychains to verify we are communicating ove a socks port

    $ tail /etc/proxychains4.conf

and add the following - our target ip and listening port

    - socks5 192.168.1.1 9999

now unlike in the local port forwarding example we will connect to the smb machine our psql target is connected to directly with smbclient

    $ proxychains smbclient -L //172.16.50.217/ -U hr_admin --password=Welcome1234

we can even use nmap with proxychains to enumerate this smb machine

    $ proxychains nmap -vvv -sT --top-ports=20 -Pn 172.16.50.217

