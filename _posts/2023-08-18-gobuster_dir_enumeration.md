---
layout: post
title: Web Server Directory Enumeration - Gobuster
titledate: 08/18/23
tags: ["infosec","offsec","web", "gobuster"]
---

Gobuster will enumerate directories and files on a target web server through brute forcing using wordlists

    $ gobuster dir -u 192.168.200.10 -w /usr/share/wordlists/dirb/common.txt -t 5

Try various user agent combinations when hunting for a flag

