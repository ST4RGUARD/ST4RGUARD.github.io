---
layout: post
title: Web Enumeration - WordPress
titledate: 08/25/23
tags: ["infosec","offsec","burpsuite", "web", "wordpress", "feroxbuster","gobuster"]
---

During web enum exercises I'm picking up that WordPress could be likely target with a high attack surface area. I do know it is an immesley popular CMS for blogging

Here I wanted to document some tools and ideas for targetting it.

<h4>Enumeration</h4>

First we need to enumerate all of the site pages and directories - we have several great tools that can report different results :TODO: (script to combine them)

    $ gobuster dir -u http://192.168.255.16 -w /usr/share/wordlists/dirb/common.txt -b '403,404,301'
    -- blacklisting http response codes

    $ dirb http://192.168.255.16 -f

    $ feroxbuster -u http://192.168.255.16:5002 -x pdf -x js,html -x php txt json,docx -s 200,301

    $ feroxbuster -u http://192.168.203.193:2000 -w list.txt

<h4>Query API</h4>

with curl

    $ curl -i http://192.168.255.16:5002/books/v1

<h4>Comments</h4>

Believe it or not there is a lot to be discovered from comments. Reading the html source would be tedious so we can search the pages for html and js comment syntax

<h4>API</h4>

Observe what is possible via the rest api index.php?rest_route=/wp/v2/users/1 and how users and posts can be added or edited

As in the API priv post, perhaps we can create a user or modify a user that exists - check wp-includes etc

Methodology should be to scan ips then ports , once ports are up, if web ports can enumerate

And Finally an external source with some great enumeration examples for users and forms etc. :TODO: (need to compile/find wordlists with most common wp directories) - gobuster with the {GOBUSTER}/v1 etc pattern file is really useful to plugin here

<h3>NOTE: use curl to grab the html of a page you want to search for strings</h3>

[wordpress enum](https://www.armourinfosec.com/wordpress-enumeration/)
