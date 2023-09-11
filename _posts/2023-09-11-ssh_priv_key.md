---
layout: post
title: SSH Private Key
titledate: 09/11/23
tags: ["infosec","offsec", "password", "ssh", "hashcat", "jtr" ]
---

during an engagement we may run across a target private id_rsa ssh key

    $ ssh -i id_rsa -p 2222 user@192.168.50.50

it is likely the key is password protected, some keys may have no password

examining the file will let us know what kind of hasing algorithm is being implemented

    id_rsa:$sshng$6$16$38298....

the $6$ indicates it is sha-512, so we remove the id_rsa: from the file

check the mode with hashcat

    $ hashcat -h | grep -i "ssh"

this will give us the mode that is required for $6$

in this example we create a custom rule file ssh.rule for passwords and relevant strings found on the target system

    c $1 $3 $7 $!
    c $1 $3 $7 $@
    c $1 $3 $7 $#

and then attempt to crack it with hashcat

    $ hashcat -m 22921 ssh.hash rockyou -r ssh.rule --force

however, hashcat gives us an error 

*Token length exception*

unfortunately hashcat does not support the aes-256-ctr cipher , but JTR will so we need to import our rules to JTR

first we can editor our ssh.rule

    [List.Rules:sshRules]
    c $1 $3 $7 $!
    c $1 $3 $7 $@
    c $1 $3 $7 $#

then import that rule to the john.conf

    $ sudo sh -c 'cat /home/kali/ssh.rule >> /etc/john/john.conf'

and execute with JTR

    $ john --wordlist=rockyou --rules=sshRules ssh.hash

now we can use the newly found key to connect to ssh

    $ ssh -i id_rsa -p 2222 user@192.168.50.50