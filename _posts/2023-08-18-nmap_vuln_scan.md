---
layout: post
title: Vuln Scanning - Nmap
titledate: 08/18/23
tags: ["infosec","offsec","vuln_scanning", "nmap"]
---

nmap NSE scripts have been updated to work with vulners

    $ sudo nmap -sV -p 443 --script "vuln" 192.168.220.10

Here we can check all vuln scripts that relate to a specific service

Also can download new nse scripts from community and choose 

    $ sudo nmap -sV -p 443 --script "http-vuln-cve2021-41773" 192.168.220.10

will dump output like vulnerable/not vulnerable