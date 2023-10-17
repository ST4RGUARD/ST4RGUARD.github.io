---
layout: post
title: Windows winRM Pivot
titledate: 10/17/23
tags: ["infosec","offsec","evil-winrm", "privesc", "powershell"]
---

#### winRM Pivot

if we have access to a windows system , and can log in as a domain user 

    runas /user:medtech\joe powershell.exe

then in our new domain session - we may be able to access an internal system via an open winrm port 5985,5956

    Enter-PSSession -ComputerName CLIENT02 -Credential medtech\poork