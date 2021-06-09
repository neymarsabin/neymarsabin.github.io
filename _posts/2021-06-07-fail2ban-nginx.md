---
layout: post
title: fail2ban with nginx
type: blog
author: neymarsabin
tags:
- nginx
- fail2ban
- denial of service
---

Fail2ban is a renowned tool to update firewall rules to reject IP addresses. This tool is packed with a daemon that runs in your machine and filters ips/request hosts according to the rules defined. Multiple example rules are already present after the tool is installed. Recently used this tool to filter python requests in app servers. All of the rules are customizable according to need and supports multiple log files to parse information about requests. Awesome for preventions against DOS attacks. This tool can be used to prevent DDOS attacks if the user agent or any other information is recurring and specific to the attacker. Going through logs is the best way to find attacker specific information.

Installation is fairly simple. Go through [this docs](https://www.fail2ban.org/wiki/index.php/Main_Page) and you will find instructions for any distro you are using.

If your nginx or any other server is behind a load balancer then you need to forward the request host/ip and nginx needs to know that you are going to display forwarded IP in logs. To do that we can set the log format in nginx:

```
log_format custom '$http_x_forwarded_for - $remote_addr - $remote_user [$time_local] ' '"$request" $status $body_bytes_sent ' '"$http_referer" "$http_user_agent" "$gzip_ratio"';
access_log /var/log/nginx/access.log custom;
error_log /var/log/nginx/error.log custom;
```
$http_x_forwarded_for gives the original request IP forwarded by load balancer otherwise $remote_addr will be the IP address of load balancer.

Create a new file `/etc/fail2ban/jail.local` that will contain directive for our custom nginx configuration.
```
[nginx-bots]

enabled  = true
port     = http.https
filter   = nginx-bots
logpath  = /var/log/nginx/access.log
maxretry = 1
findtime = 604800
bantime  = 604800
```
`filter = nginx-bots` is the name of a file `/etc/fail2ban/filter.d/nginx-bots.conf`. The file contains regex rule to parse and filter out IPs that needs to be banned.
```
[Definition]
badbots = python # this can be filled with all known bot names like ruby|360Spider|Baiduspider
failregex = ^<HOST> - .* "-" ".*(?:Sogou web spider|Baiduspider|Abonti|Pixray|CPython|Spinn3r|libwww-perl|python|Ruby).*"$
ignoreregex = 
```
The regex should contain `<HOST>` because this field contains the IP that needs to be banned if the request agents from this `Sogou web spider|Baiduspider|Abonti|Pixray|CPython|Spinn3r|libwww-perl|python|Ruby` list is found in the log.

Now restart the fail2ban service
```
sudo systemctl restart fail2ban
```
Status and other commands can be executed by using the `fail2ban-client` cli tool.
```
sudo fail2ban-client status
sudo fail2ban-client -h
```
