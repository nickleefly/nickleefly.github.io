---
layout: post
title: "google Nginx proxy with https from letsencrypt"
date: 2019-06-08 20:30
comments: true
categories: nginx google
---

In this post, we are going to create a nginx proxy google with https, the cerficate is from letsencrypt generated by certbot

You will need a domain name, eg google.exmaple.com
Then Add a A record in your domain DNS record.

In your VPS, please install `docker`, `docker-compose` before you continue.
Clone [nginx-certbot][1]

in nginx-certbot/data/nginx/app.conf
put the following content into the file

```
server {
    listen 80;
    server_name google.example.com;
    server_tokens off;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    server_name google.example.com;
    server_tokens off;

    ssl_certificate /etc/letsencrypt/live/google.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/google.example.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    resolver 8.8.8.8;
    location / {
        google on;
        google_scholar on;
        google_language zh-CN;
    }

    # block spider
    if ($http_user_agent ~* (baiduspider|360spider|haosouspider|googlebot|soso|bing|sogou|yahoo|sohu-search|yodao|YoudaoBot|robozilla|msnbot|MJ12bot|NHN|Twiceler)) {
        return  403;
    }

}
```

in `init-letsencrypt.sh`, change the domain name to google.example.com
and change the email address.

in nginx-certbot directory,
run the following command

update nginx base image to `nickleefly/nginx-proxy-google` in `docker-compose.yaml`

Then run
```
./init-letsencrypt.sh
docker-compose up -d
```

Now check your google.example.com


More information, please refer to [Nginx and Let’s Encrypt with Docker in Less Than 5 Minutes
][2]

[1]: https://github.com/wmnnd/nginx-certbot
[2]: https://medium.com/@pentacent/nginx-and-lets-encrypt-with-docker-in-less-than-5-minutes-b4b8a60d3a71
