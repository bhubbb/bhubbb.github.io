---
layout: post
title: "Block hosts with fail2ban for a caddy server"
tags:
  - caddy
  - fail2ban
  - linux
  - matchers
  - actions
---

 I've implemented hosting using Caddy (<https://caddyserver.com/>), and aimed to utilize Fail2ban for securing the server. However, I discovered that existing examples do not work.

## Overview

This configuration dump demonstrates how to configure Fail2ban to block unauthorized access at a Caddy server using Caddy request matchers (<https://caddyserver.com/docs/caddyfile/matchers>).

This setup involves having Fail2ban read the Caddy logs by employing an HTTP 400 Response filter, adding offending hosts to a Caddyfile, and subsequently reloading Caddy.

## Configurations

### /etc/caddy/Caddyfile 
```caddy
# Host configuration
example.com {
  # The file that holds the banned hosts
  import /etc/caddy/banned
  # API 
  handle_path /api/* {
    reverse_proxy 127.0.0.1:5000
  }
  # Static
  handle /* {
    root * /srv/example.com
    file_server
  }
  # Logs
  log {
    output file /var/log/caddy/example.com.log
    format console
  }
}
```
### /etc/caddy/banned
```caddy 
@banned {
  # We need at least one ip in here to be valid and 0.0.0.0 will never match anything
	remote_ip 0.0.0.0
}

# You can tweak the response and code to whatever you want
respond @banned "BANNED" 403 {
	close
}
```
### /etc/fail2ban/jail.d/00-caddy.conf
```fail2ban 
[caddy]
enabled = true
filter = caddy
logpath = /var/log/caddy/*.log
maxretry = 5
findtime = 300
bantime = 300
ignoreip = 192.168.1.1
action = caddy
```
### /etc/fail2ban/filter.d/caddy.conf
```fail2ban 
[Definition]
failregex = "remote_ip":\s+"<HOST>".*status":\s+4\d{2} 
ignoreregex =
```
### /etc/fail2ban/action.d/caddy.conf
```fail2ban 
#
# Brendan Hubble
# https://bhubbb.github.io/
#

[Definition]
actionstart = 
actionstop  = 
actioncheck =
actionban   = sed -i '1 a\	remote_ip <ip>' <caddy_ban_file>;
              caddy reload --config /etc/caddy/Caddyfile; 

actionunban = sed -i '/<ip>/d' <caddy_ban_file>;
              caddy reload --config /etc/caddy/Caddyfile;

[Init]
init           = 'Banning Host in Caddy'
caddy_ban_file = /var/opt/caddy/config/banned
```
