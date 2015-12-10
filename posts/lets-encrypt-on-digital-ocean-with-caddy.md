---
title: "Let's Encrypt (free SSL) on Digital Ocean with Caddy"
description: ''
date: '2015-12-09 4:39 pm'
uuid: 92ec9d40-d0b9-451e-b6dc-ac6b8aa060aa
permalink: /articles/lets-encrypt-on-digital-ocean-with-caddy/
author: coolaj86
---

What do we want? Encryption!

When do we want it? Now!

How do we want it? Free and Easy!

Free and Easy SSL with Digital Ocean (and Caddy)
-------------

Thanks to the folks at [LetsEncrypt](https://letsencrypt.org/) - that being Mozilla, the EFF, Akamai,
and a number of other sponsors - you can now get **HTTPS certificates for free**.

It's important to note that these certificates are only valid **for 90 days**,
however they are also **automatically renewable**.

[Caddy](https://caddyserver.com) is the first webserver with Let's Encrypt registration and renewal built-in.

### You Will Succeed

At the end of this tutorial you will have:

* Your very own domain
* Caddy as your webserver
* HTTPS turned on

### Prerequisites

With every tutorial there are some assumptions made.

Here we're assuming that you consider yourself a smart cookie who is confident in its
skills in copy/paste and well practiced in google-fu.

We're also assuming that you've used VPS Web Hosting of some sort
(whether Digital Ocean, Chunkhost, Linode, whatever) before.

* Sepia belt in [following instructions](https://xkcd.com/518/)
* copy/paste (mauve belt or higher)
* Google-fu ("San Marino Blue" belt or higher)
* Digital Ocean / Web Hosting / VPS (Lemon belt or lower)
* Terminal (striped green belt)
* SSH (no belt required, refer to copy/paste if lost)

If you **haven't used Digital Ocean** you may benefit
from [Digital Ocean for noobs (screencast)](https://www.youtube.com/watch?v=ypjzi1axH2A)
and [How to Secure Your VPS (screencast)](https://www.youtube.com/watch?v=YZzhIIJmlE0).

### Tools and Topics

In order to make this process super fast and easy we're gonna use these tools:

* letsencrypt / ACME (client and standard for digital certificate registartion)
* Caddy (the new successor in the long line of nginx, apache, etc)
* Digital Ocean (everyone's favorite VPS and hosting provider)
* Daplie DNS (quick and easy dynamic dns)

Quick Start
-----------

### 1. Login to the VPS

For reference, these are the settings I chose for my "droplet":

```
Ubuntu 14.04 LTS
$5 / month (512 MB / 20 GB)
NYC
SSH key enabled (or password if you prefer)
```

When you sign up for Digital Ocean you should either select a key or get a username and password in your email.

**IMPORTANT** Copy and paste these instructions into the terminal, but **change them** to match **your email**.

```bash
ssh root@<<ip-address>>
```

**Example**:
```bash
ssh root@104.236.107.139
```

**Follow the instructions**.
Yes, you're sure you want to connect.
Yes, it is taking your password even though it doesn't show it.

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

This will install caddy:
```bash
curl -fsSL https://getcaddy.com | bash -s search
```

### 3. Point a domain to your server

For the sake of making this tutorial quick and easy, we're going to give you a domain that you can play with
right here and now. Yay!

**Run this command** in Terminal (in the window as the `root@ubuntu-whatever:~#` prompt)

```bash
ddns-testing
```

You will see output that is **similar to this**:

```

----------------------------------------------
Hostname: quiet-goat-64.testing.letssecure.org
IP Address: 104.236.107.139
----------------------------------------------
config saved to '/root/.ddnsrc-testing.json'


Test with
dig quiet-goat-64.testing.letssecure.org A

```

Yay! You now have a domain name you can test with to follow this tutorial!

**Shameless Plug**:

When you're ready to purchase your domain for reals:

I'd love to tell you to use Daplie Domains to purchase a domain,
but since we haven't finished building that I recommend what I've been
using personally for the past few years: [name.com](https://www.name.com).

### 4. Automatic HTTPS with Caddy

You need to **create a `Caddyfile`** and a little index.html to serve.

```bash
# creates a file with only the hostname
# accept all default options

echo 'quiet-goat-64.testing.letssecure.org' >> ./Caddyfile

# creates an html file to serve with caddy
echo 'Hello, World!' >> ./index.html
```

Now you can run caddy with that configuration file

**IMPORTANT**: Replace this with **YOUR EMAIL**!!!

```bash
caddy --conf ./Caddyfile --agree --email 'John.Doe@EXAMPLE.COM'
```

You can run `caddy --help` to see all of the options.

### 6. Where the HTTPS Certificates are


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
    │   └── sweet-cow-57.daplie.me
    │       ├── sweet-cow-57.daplie.me.crt
    │       ├── sweet-cow-57.daplie.me.json
    │       └── sweet-cow-57.daplie.me.key
    └── users
        └── coolaj86@gmail.com
            ├── coolaj86.json
            └── coolaj86.key
```

### More Fun with Caddy

If all of that worked for you and you're ready to dive a little deeper into caddy:
