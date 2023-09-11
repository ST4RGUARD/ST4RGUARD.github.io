---
layout: post
title: Mutating Password Lists with Hashcat
titledate: 09/10/23
tags: ["infosec","offsec", "password", "hashcat"]
---

#### Mutating Passwords

with hydra we can create rules that can make it easier to mutate password lists

if we want a rule with the ! character and a number 1 that is capitalized

    $ echo \$1 c $! demo1.rule

then we can create a reverse order rule

    $ echo \$! c $1 demo2.rule

a rule with several different orders might look like - several.rule

    $1 c $!
    $2 c $!
    $3 c $!
    $1 $2 $3 c $!

can test a rule with a test file including a couple words

    $ hashcat -r hash.rule --stdout test.txt

and then to crack the hash

    $ hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt -r several.rule --force
    
can look at predefined rules to mutate lists provided with hashcat and then externally

    $ /usr/share/hashcat/rules