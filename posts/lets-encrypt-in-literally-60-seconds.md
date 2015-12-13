---
title: "Let's Encrypt in (literally) 90 seconds"
description: "The fastest you've ever set up a website in your entire life. Period."
date: '2015-12-10 0:02 pm'
uuid: 2d10d29e-1e80-45a2-881a-4e4cc77caab1
permalink: /articles/lets-encrypt-in-literally-90-seconds/
youtube: "22zg3TPSeEo"
player: "https://www.youtube.com/embed/22zg3TPSeEo"
---

This is ridiculous. Purely for Giggles.

But, yes, we're going to set up an HTTPS Web Server on a fresh system in (exactly) 90 seconds.

Assumptions
-----------

* you have a server / VPS / webhost
* you do commandline-fu
* you ssh

### If you're a n00b

Go here instead: [Free SSL with Digital Ocean & Caddy](/articles/lets-encrypt-on-digital-ocean-with-caddy/)

Go!
---

Set your stopwatch and copy and paste at your fastest.

0 to HTTPS in 90.... Go!

### Install Daplie DNS

The Daplie Dynamic DNS client is written in node.js,
so this script installs that and a few other goodies you
need for node.js development on a VPS.

```bash
curl -fsSL https://bit.ly/iojs-dev | bash

# Or shave a few seconds
# curl -fsSL https://bit.ly/iojs-min | bash

npm install --global ddns-cli
```

### Install Caddy

Caddy is a next-gen webserver. It's kinda like nginx but it's written and go,
is super easy to use, and comes complete with fully automatic Let's Encrypt integration.

```bash
curl -fsSL https://getcaddy.com | bash -s search

# allow caddy to use port 443 (https) and 80 (http)
sudo setcap cap_net_bind_service=+ep /usr/local/bin/caddy
```

#### Set EMAIL Variable

```
# IMPORTANT
# CHANGE THE EMAIL

EMAIL=john.doe@example.com
```

### Grab a Domain

These Daplie Dynamic DNS domains are registered only by email address for now,
but will have proper authentication in the future. Use an email address
you actually have access to or you may lose the domain once it's in proper beta.

This will give you a `.daplie.me` domain `rubber-duck-42.daplie.me` or `whatever.daplie.me`.

```bash
ddns --random --agree --email $EMAIL

# or if you want a custom subdomain do this
#ddns --hostname whatever.daplie.me --agree --email $EMAIL
```

#### Set DAPLIE_HOSTNAME

Just to make this process a little faster we can set the `.daplie.me` subdomain
to an environment variable.

```
#IMPORTANT
#CHANGE THE HOSTNAME

DAPLIE_HOSTNAME=rubber-duck-42.daplie.me
```

### Configure Caddy


`/srv/www/Caddyfile`:
```
# CHANGE THE HOSTNAME
rubber-duck-42.daplie.me {
  gzip
  tls john.doe@example.com
  root /srv/www/rubber-duck-42.daplie.me
}
```

Or, if you want to shave a few seconds, simply

```
mkdir -p "/srv/www/$DAPLIE_HOSTNAME/"

echo "$DAPLIE_HOSTNAME {"               >> /srv/www/Caddyfile
echo "  gzip"                           >> /srv/www/Caddyfile
echo "  tls $EMAIL"                     >> /srv/www/Caddyfile
echo "  root /srv/www/$DAPLIE_HOSTNAME" >> /srv/www/Caddyfile
echo "}"                                >> /srv/www/Caddyfile
```

### Make a Website

```bash
echo 'Hello, World!' > "/srv/www/$DAPLIE_HOSTNAME/index.html"
```

### Start Caddy, Bask in Glory

```bash
echo "https://$DAPLIE_HOSTNAME"

caddy --conf /srv/www/Caddyfile --agree --email $EMAIL
```

### Join the Scoreboard

Comment with your best time below!

Think you can make an even faster process? Let us know how.

Oh, and congrats btw. You're now a boss.
