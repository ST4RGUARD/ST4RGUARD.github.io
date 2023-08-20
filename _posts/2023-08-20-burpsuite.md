---
layout: post
title: Burpsuite
titledate: 08/20/23
tags: ["infosec","offsec","burpsuite", "web"]
---

Burpsuite is a tool that acts as a proxy and allows us to inspect and modify web traffic 

<h4>burp setup</h4>

<h3>NOTE - burp seems to take up tons of resources randomly so worth increasing vm hardware</h3>

under the proxy tab in burpsuite turn intercept to on and make sure the default proxy settings 127.0.0.1 port 8080 are remembered (or wtv address)

we set the proxy for our browser firefox in settings/network settings to localhost port 8080 to match burpsuite

check also use this proxy for HTTPS

![_config.yml]({{ site.baseurl }}/images/web/burp_proxy.png)

<h4>burp repeater</h4>

we can see intercepted requests in the left panel of burpsuite showing our HTTP GET request

if we want to modify or craft a request to replace the default request we can right click the request and send to repeater

here we can insert or modify the request and send it to the target IP

![_config.yml]({{ site.baseurl }}/images/web/burp_repeater.png)

<h4>burp intruder</h4>

<b>Intruder</b> like <b>Repeater</b> is a way to use burp as a proxy to capture and modify requests and send them to the target

the big difference being the ability to generate variables for traffic, this is particularly useful for fuzzing forms/fields with requests

to utilize this feature, leave intercept off, browse to the form, find the request in the burp panel and send it to intruder

in the intruder panel highlight the variable you want to fuzz, highlight it and click Add

next click on the Payloads tab under Intruder, there are a lot of payload choices to use, but we will select from a list

![_config.yml]({{ site.baseurl }}/images/web/burp_intruder.png)

