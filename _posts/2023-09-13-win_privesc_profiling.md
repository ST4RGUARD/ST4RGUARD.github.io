---
layout: post
title: Windows Target Profiling
titledate: 09/13/23
tags: ["infosec","offsec", "privesc"]
---

#### Target Info

### NOTE: use single quotes when passing spaced arguments to powershell inside cmd.exe i.e - C:> powershell -command Get-LocalUser 'User A'

Username and hostname

    C:> hostname

the hostname can tell us what the purpose of the server is, a client system or maybe MSSQL server

Group memberships of the current user

    C:> whoami /groups

the groups can tell us what privileges the user has, or what services it may be able to connect to like Remote Desktop Users

Existing users and groups

    C:> net user
    PS:> Get-LocalUser

list of all local user accounts
    
    PS:> Get-LocalGroup

list all groups
    
    PS:> Get-LocalGroupMember adminteam

displays members of a group

Operating system, version and architecture

    PS:> systeminfo

display the system info

Network information

    C:> ipconfig /all

display network interfaces

    PS:> route print

display all route info

    PS:> netstat -ano

display all active network connections

the listening ports can also indicate running services

Installed applications

    PS:> Get-ItemProperty 'HKLM:\\SOFTWARE\\Wow6432Node\\Microsoft\\Windows\\CurrentVersion\\Uninstall\\*' | select displayname

get x64 installed applications

    PS:> Get-ItemProperty 'HKLM:\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Uninstall\\*' | select displayname

get x32 installed applications

Running processes

    PS:> Get-Process |Select-Object -ExpandProperty Path

list of currently running processes with path

if keepass and xampp are installed there should be password manager databases

we can run a search for these database files with 

    $ Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue

xampp search for files 

    $ Get-ChildItem -Path C:\xampp -Include *.txt,*.ini -File -Recurse -ErrorAction SilentlyContinue

we can search the users home directory/downloads/desktop/documents

    $ Get-ChildItem -Path C:\Users\dave\ -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue

if we find any relevant user or password information, we can cross reference it with the group they belong to

    $ net user dan

perhaps we can test a different user wlike dan and connect to rdp to see if he mas access to any txt or ini files etc

we may be able to pivot from user to user

we can run commands as a specific user like below if we do not have RDP access

    $ runas /user:backupadmin cmd