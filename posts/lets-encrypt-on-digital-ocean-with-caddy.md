---
title: "Let's Encrypt with Digital Ocean & Caddy!"
tagline: ""
description: "Host your own HTTPS website with Free SSL using next-gen weberver Caddy in 10 minutes."
date: '2015-12-09 4:39 pm'
uuid: 92ec9d40-d0b9-451e-b6dc-ac6b8aa060aa
permalink: /articles/lets-encrypt-on-digital-ocean-with-caddy/
author: coolaj86
youtube: "TPlHJhN--74"
player: "https://www.youtube.com/embed/TPlHJhN--74"
---

<!--
lucky-duck-42
silly-goose-42
-->

What do we want? Encryption!

When do we want it? Now!

How do we want it? Free and Easy!

How are you going to get it today?

* [LetsEncrypt](https://letsencrypt.org/) is an industry movement for security and privacy across the web.
* [Caddy](https://caddyserver.com) is the first webserver with Let's Encrypt registration and renewal built-in.
* [DigitalOcean](https://digitalocean.com) provides great webhosting with awesome VPSes.
* [Daplie](https://daplie.com) (that's us) is here to help you take back the Internet and make it yours.

Free and Easy SSL
-----------------

Thanks to kind folks at Mozilla, the EFF, Akamai,
and a number of other sponsors you can now get **HTTPS certificates for free**, *forever*.

* Free - no money (as in $0)
* 90-day lifetime (with automatic renewable)
* Email-only registration
* Domain ownership validation

Again, this is **legit** free - not some junk trial.

### What You Will Accomplish

At the end of this tutorial you will have:

* A Free Custom Domain (via **Daplie**)
* Fast & Lightweight Web Server (via **Caddy** or any web server you choose)
* Free SSL (via **LetsEncrypt**)
* $5/month hosting (via **Digital Ocean**)

### Prerequisites & Assumptions

With every tutorial there are some assumptions made.

Here what we're assuming:

* you consider yourself a **smart cookie**
* who is confident
* who can [follow instructions](https://xkcd.com/518/)
* who has mad skills in copy/paste
* and is well-practiced in google-fu
* and are willing to spend *literally* 1.4¢ (Digital Ocean charges by the hour and costs 0.7¢)

We're also assuming that you don't mind getting your hands a little
dirty and that you have experience with VPS Web Hosting of some sort
(whether Digital Ocean, ChunkHost, Linode, AWS, or whatever).

Required Technical Chops:

* some knowleged Web Hosting / VPS (Lemon belt or lower)
* Terminal (striped green belt)
* SSH (no belt required, refer to copy/paste if lost)

If you **Haven't used Web Hosting** before, but are somewhat familiar with Terminal check these out:

  * [Digital Ocean for noobs (screencast)](https://www.youtube.com/watch?v=ypjzi1axH2A)
  * [How to Secure Your VPS (screencast)](https://www.youtube.com/watch?v=YZzhIIJmlE0)

In order to make this process super fast and easy we're gonna use these tools:

* letsencrypt / ACME (client and standard for digital certificate registartion)
* Caddy (the new successor in the long line of nginx, apache, etc)
* Digital Ocean (everyone's favorite VPS and hosting provider)
* Daplie DNS (quick and easy dynamic dns)

Quick Start
-----------

### For the Pros

If you can't be bothered to read and just want the command line stuff:

* [Let's Encrypt in (literally) 90 seconds](/articles/lets-encrypt-in-literally-90-seconds/)

### 1. Login to the VPS

Go to [DigitalOcean.com](https://digitalocean.com/) and create a **droplet**.

For reference, these are the settings I chose:

```
Choose an image
  Ubuntu 14.04.3 x64
  (it's an LTS release)

Choose a size
  $5 / month (0.7¢ / hour)
  512 MB / 20 GB / 1000 GB

Choose a datacenter region
  New York
  3
  (SF is also nice)

Select additional options
  (none)

Add your SSH keys
  (ignore for now if you're unfamiliar)

Finalize and Create
  How many Droplets? 1 Droplet
  Choose a hostname (accept the default)
```

Next you need to **check your email** that's where you will find:
  * ip address
  * username
  * password

**IMPORTANT** Copy and paste these instructions into the terminal, but use **your ip address**, NOT mine.

To be clear, you'll replace `<<ip-address>>` with your IP.

```bash
ssh root@<<ip-address>>
```

**Example**:

```bash
ssh root@104.236.107.139
```

Then hit **enter** and **follow the instructions**.

* Yes, you're sure you want to connect.
* Yes, it is taking your password even though it doesn't show it.

You'll end up at a nice happy "prompt" that probably **looks similar to** this:

```bash
root@ubuntu-512mb-nyc3-01:~# █
```

That's good.

### 2. Install the Tools

You don't have to use all of these tools, but they make it simple for this tutorial.

Just copy and paste each of these into the terminal one-by-one:

This will install node.js and a Daplie DNS client:
```bash
curl -fsSL https://bit.ly/iojs-min | bash

npm install --global ddns-cli
```

This will install caddy and allow it to use http and https:
```bash
curl -fsSL https://getcaddy.com | bash -s search

# allow caddy to use port 443 (https) and 80 (http)
sudo setcap cap_net_bind_service=+ep /usr/local/bin/caddy
```

### 3. Point a domain to your server

For the sake of making this tutorial quick and easy, we're going to give you
a domain that you can play with right here and now. Yay!

**Run this command** in Terminal (in the window as the `root@ubuntu-whatever:~#` prompt)

Change the example email to **your email** so that the domain will be available to you.

```bash
# Run this if you want a custom domain
ddns --hostname johndoe.daplie.com --agree --email john.doe@example.com

# Or run this for a random domain
ddns --random --agree --email john.doe@example.com
```

You will see output that is **similar to this**:

```

----------------------------------------------
Hostname: rubber-duck-42.daplie.me
IP Address: 104.236.107.139
----------------------------------------------
config saved to '/root/.ddnsrc.json'


Test with
dig rubber-duck-42.daplie.me A

```

Yay! You now have a domain name you can test with to follow this tutorial!

**Note**:
* Our domain service is **alpha**.
* As of December 2015 we are actively developing on it.
* We will **email you** when the "real" user registration is available.

Want **your own .com**?

**If you can wait** until January we'd love to have you as a Daplie Domains beta customer.
Until then, feel free to use a `.daplie.me`.

**If you can't wait** then I recommend [name.com](https://www.name.com).
I've personally been a satisfied customer of theirs for years.

### 4. Automatic HTTPS with Caddy

Even if you don't use Caddy as your main webserver (although we hope you do),
it's the easiest way to get letsencrypt certificates.

You need to **create a `Caddyfile`**.
Change to **your hostname**.

```bash
# Creates a file with only the hostname
# accept all default options
echo 'rubber-duck-42.daplie.me' >> ./Caddyfile
```

You'll also want something to welcome others to your cozy home on the web:

```bash
# creates an html file to serve with caddy
echo 'Hello, World!' >> ./index.html
```

Now you can run caddy with that configuration file

**IMPORTANT**: Replace this with **YOUR EMAIL**!!!
(otherwise you lose the ability to renew your certificates for free)

```bash

$
caddy --conf ./Caddyfile --agree --email 'john.doe@EXAMPLE.COM'
```

You can run `caddy --help` to see all of the options.

### 6. Check it out!

Here's the fun part.

Go to <https://YOUR-SUBDOMAIN.daplie.me> (yes, change it out for yours).

### 6. Where the HTTPS Certificates are

They're here:

```
~/.caddy/letsencrypt/rubber-duck-42.daplie.me/

  ├── rubber-duck-42.daplie.me.json
  ├── rubber-duck-42.daplie.me.crt (fullchain.pem)
  └── rubber-duck-42.daplie.me.key (privkey.pem)
```

Now that you know where they are, I would suggest backing up the entire
`~/.caddy/letsencrypt` directory to your local computer - it
contains both certificates and domain registration information.

Also, here's how you can see the entire directory structure for yourself:

```bash
# install tree
sudo apt-get install tree

# use tree
tree ~/.caddy
```

The output looks **similar to this**:

```
/root/.caddy
└── letsencrypt
    ├── sites
    │   └── rubber-duck-42.daplie.me
    │       ├── rubber-duck-42.daplie.me.crt (fullchain.pem)
    │       ├── rubber-duck-42.daplie.me.json
    │       └── rubber-duck-42.daplie.me.key (privkey.pem)
    └── users
        └── coolaj86@gmail.com
            ├── coolaj86.json
            └── coolaj86.key
```

### Recap

You now have

  * your own `.daplie.me` vanity domain.
  * a Web Host
  * a Web Server
  * a Free LetsEncrypt SSL Certificate

You can use those certificates with Caddy, or with any other web server (including email).

### Next Steps

Hopefully you've wet your appetite, but you're not quite done yet.

If you're interested in Caddy you probably want to set up
**Caddy as a system service with upstart** and learn a little more about it, in general.

* [Web Hosting with Caddy](/articles/web-hosting-with-caddy/)

Otherwise, if this simple proof-of-concept made you hungry for learning how to use
the mainstream `letsencrypt` python client with other software.

* [Advanced Let's Encrypt Configuration](/articles/going-deep-with-letsencrypt-webroot/)
