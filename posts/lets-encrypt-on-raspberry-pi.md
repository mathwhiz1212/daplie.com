---
title: "Let's Encrypt on Raspberry Pi"
description: ''
date: '2015-12-10 6:10 pm'
uuid: 3bf4e7d6-6284-45e7-8c51-0825f10452c2
permalink: /articles/lets-encrypt-on-raspberry-pi/
---

Updated for Let's Encrypt Public Beta
--------------------

Here's a recap of what's changed:

  * Simpler install process (auto detects environment and runs appropriate bootstrapper)
  * Installs to `~/.local/share/letsencrypt/bin/letsencrypt`
  * dvsni challenge replaced with http-01 (for registration) and tls-sni-01 (for renewal?)
  * Free `**your-name-here**``.daplie.me` dynamic ip domains
  * no `/etc/letsencrypt/cli.ini`

Goals
-----

Following the installation instructions at
[https://letsencrypt.readthedocs.org](https://letsencrypt.readthedocs.org/en/latest/using.html#installation)
we will:

  * build `letsencrypt`
  * use `letsencrypt` to get a valid TLS (SSL) cert for HTTPS
  * register a free dynamic domain
  * start an https-enabled webserver

You may also be interested in

  <!-- TODO XXX Daplie DNS
  * [QuickStart Guide to Daplie DNS](/articles/free-dynamic-domain-with-daplie-domains/)
  -->
  * [QuickStart Guide to FreeDNS](https://coolaj86.com/articles/free-dns-hosting-with-freedns-afraid-org.html)
  * [How to Multiplex Port 443 with HAProxy](./lets-encrypt-with-haproxy/)

`caddy` (The Easy Way)
------------

[`caddy`](https://caddyserver.com) has fully-automatic, built-in letsencrypt support.

Check out one of our Artciles on Caddy and Let's Encrypt:

  * [Let's Encrypt with Digital Ocean & Caddy! (tut + screencast)](https://daplie.com/articles/lets-encrypt-on-digital-ocean-with-caddy/)
  * [Let's Encrypt in (literally) 90 seconds (tut + screencast)](https://daplie.com/articles/lets-encrypt-in-literally-90-seconds/)

`letsencrypt` (official python client)
-------------------------------

This process is *super* slow on a raspberry pi because it has to compile a metric *ton*
of dependencies for `python`.

Wait Times:

  * **49 seconds** on Digital Ocean $5/month instance (512MB RAM 1x 2.4GHz)
  * **7.5 minutes** on Raspberry Pi B 2.0 (1GB RAM 4x 900MHz)
  * **21 minutes** on Raspberry Pi B (256MB RAM 1x 700MHz)

```bash
# Clone the repo
git clone https://github.com/letsencrypt/letsencrypt

# Push into the direcotry
pushd letsencrypt

# Run the automated installer (see wait times above)
time ./letsencrypt-auto
```

If you're on a Pi just go grab some munchies and wait it out...

Unblock ports 443 and 80
--------

**Firewall**:

**If** you have a firewall, don't forget to allow ports 443/tcp and 80/tcp.

```bash
sudo ufw allow http
sudo ufw allow https
```

**Port Forwarding**:

If you're running inside of your house you're going to need to use port forwarding
on your router to allow 443 and 80 to come through to you.

We've got some code in the works to help you do this automatically:

* <https://github.com/Daplie/holepunch>

Note: at the time of this writing the repository is just a stub,
but look forward to good things!

Get a Free Domain
-----------------

If you **already have a domain**, great. Use that.
Otherwise, here's one on the house.

Install the Daplie DNS Client

```bash
# Install node.js (excuse the iojs misnomer)
curl -fsSL https://bit.ly/iojs-min | bash

# Install the Daplie DNS client
npm install --global ddns-cli
```

Register a `.daplie.me` domain for free

```
ddns --hostname johndoe.daplie.me --agree --email john.doe@example.com
```

Or get a random name such as `rubber-duck-342.daplie.me`

```
ddns --random --agree --email john.doe@example.com
```

Get a certificate for your Domain
---------------------------------

**IMPORTANT**: Change the email address to **your email** and
change the domain to **your domain**:

Wait Times:

  * **a few seconds** Digital Ocean
  * **a few dozen seconds** Raspberry Pi 2 (1GB)
  * **about a 2 minutes** Raspberry Pi B (256MB)

```bash
# STANDALONE (you webserver must NOT be running)
# Get a 90-day certificate for your domain
sudo ~/.local/share/letsencrypt/bin/letsencrypt certonly \
  --agree-tos --email john.doe@example.com \
  --standalone \
  --domains example.com,www.example.com
```

**IMPORTANT**: standalone mode is fine for casual use,
but you will need to script webroot mode for automatic renewal (see next section)

Note: If you get an error about rate-limiting, check and see if
if [letsencrypt github issue #1251](https://github.com/letsencrypt/boulder/issues/1251)
and [public suffix list github issue #74](https://github.com/publicsuffix/list/pull/74)
have been resolved. As soon as those are taken care of that won't happen anymore.

Webroot
-------

Standalone mode is cool for testing, but it's not very useful in the real world.

Since your webserver needs ports 80 and 443 and letsencrypt also needs
to use ports 80 and 443 you can't just stop your webserver for whole seconds (or 2 minutes!!)
and let letsencrypt take over.

There were many [reasonable arguments](https://github.com/letsencrypt/acme-spec/issues/19)
and [much debate](https://github.com/letsencrypt/acme-spec/issues/33).

The resolution: Webroot mode.

Webroot mode effectively solves this problem without introducing a lot of programming headache:

  * You specify your *web root* (makes sense)
  * it initiates the proper requests to the ACME / boulder server
  * it write the necessary challenge files to `.well-known/acme-challenge/`
  * and then cleans up after itself

```bash
# WEBROOT (run while your webserver is live)
# http://example.com/.well-known/acme-challenge/xxxxxxxxxxxxxx
sudo ~/.local/share/letsencrypt/bin/letsencrypt certonly \
  --agree-tos --email john.doe@example.com \
  --webroot \
  --webroot-path /srv/www/example.com/
  --domains example.com,www.example.com
```

Note: Make sure you **ALLOW DOTFILES** in your server configuration,
as the server will request `.well-known`.

If you want to be able to **"watch" this process happening**, use manual mode:

```bash
# MANUAL (observe and take part in the process)
sudo ~/.local/share/letsencrypt/bin/letsencrypt certonly \
  --agree-tos --email john.doe@example.com \
  --manual \
  --domains example.com,www.example.com
```

You will need to do this in a new terminal window.
As soon as the certificates are granted you can stop this server.

Result
------

If all went well you can now run `tree /etc/letsencrypt/` and you'll be able to see
all of the registration files and keys.

Install and run tree

```bash
sudo apt-get install --yes tree
sudo tree /etc/letsencrypt/
```

The output should look like this:

`/etc/letsencrypt/`:
```
sudo tree /etc/letsencrypt
/etc/letsencrypt
├── accounts (ROOT PROTECTED)
│   └── acme-v01.api.letsencrypt.org
│       └── directory
│           └── 405ef3114adb6eabf5311420fa3f162d
│               ├── meta.json
│               ├── private_key.json
│               └── regr.json
├── archive (ROOT PROTECTED)
│   └── example.com
│       ├── cert1.pem       (server public certificate)
│       ├── chain1.pem      (intermediate certificate(s))
│       ├── fullchain1.pem  (server cert followed by intermediate(s))
│       └── privkey1.pem    (server private keypair)
|                           (NOTE: your browser already has the root cert)
├── csr
│   └── 0000_csr-letsencrypt.pem
├── keys (ROOT PROTECTED)
│   └── 0000_key-letsencrypt.pem
├── live (ROOT PROTECTED)
│   └── example.com
│       ├── cert.pem -> ../../archive/example.com/cert1.pem
│       ├── chain.pem -> ../../archive/example.com/chain1.pem
│       ├── fullchain.pem -> ../../archive/example.com/fullchain1.pem
│       └── privkey.pem -> ../../archive/example.com/privkey1.pem
└── renewal
    └── example.com.conf
```

Note: You don't usually need the root certificates
(because browsers, phones, devices, etc update them automatically)
but in the rare case that you do need them (because it does happen),
they're availabel here:

  * https://letsencrypt.org/certs/isrgrootx1.pem.txt
  * https://letsencrypt.org/certificates/

Other notes about the directory structure may be available via
[letsencrypt github issue 608](https://github.com/letsencrypt/letsencrypt/issues/608).

Testing your Certs
------------------

You can use `serve-https` to quickly test your certificates straight
from the `/etc/letsencrypt`.

```bash
npm install -g serve-https

sudo serve-https -p 8443 --letsencrypt-certs foo.example.com -c 'hello world!'

curl https://foo.example.com:8443
```

See [serve-https](https://github.com/Daplie/localhost.daplie.com-server).

Using your Certs with Node.js
----------------------

In node.js you would use these like so:

```js
'use strict';

var fs = require('fs');
var path = require('path');
var letsetc = '/etc/letsencrypt/live/';
var defaultdomain = 'dangerpi.devices.coolaj86.com';

function getSecureContext(domainname, opts) {
  if (!opts) { opts = {}; }

  opts.key = fs.readFileSync(path.join(letsetc, domainname, 'privkey.pem'))
  opts.cert = fs.readFileSync(path.join(letsetc, domainname, 'cert.pem'));
  opts.ca = fs.readFileSync(path.join(letsetc, domainname, 'chain.pem'), 'ascii')
    .split('-----END CERTIFICATE-----')
    .filter(function (ca) {
      return ca.trim();
    }).map(function (ca) {
      return (ca + '-----END CERTIFICATE-----').trim();
    });

  return require('tls').createSecureContext(opts);
}

//
// SSL Certificates
//
var options = {
, requestCert: false
, rejectUnauthorized: true

  // If you need to use SNICallback you should be using io.js >= 1.x (possibly node >= 0.12)
, SNICallback: function (domainname, cb) {
    var secureContext = getSecureContext(domainname);
    cb(null, secureContext);
  }
  // If you need to support SPDY/HTTP2 this is what you need to work with
//, NPNProtocols: ['http/2.0', 'spdy', 'http/1.1', 'http/1.0']
  , NPNProtocols: ['http/1.1']
};

// Start the server
server = https.createServer(getOpts(defaultdomain, options));
server.on('error', function (err) {
  console.error(err);
});
server.listen(443, function () {
  console.log('Listening');
});
server.on('request', function (req, res) {
  res.end('hello');
});
```
