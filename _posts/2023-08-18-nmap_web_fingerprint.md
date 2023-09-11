---
layout: post
title: Fingerprint Web Server - Nmap
titledate: 08/18/23
tags: ["infosec","offsec","web", "nmap", "http"]
---

We can use Nmap to fingerprint the target web server using service discovery

    $ sudo nmap -p80  -sV 192.168.200.10

or a specific NSE script like http-enum

    $ sudo nmap -p80 --script=http-enum 192.168.200.10
