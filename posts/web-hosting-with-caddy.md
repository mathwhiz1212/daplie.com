---
title: Web Hosting with Caddy
description: ''
date: '2015-12-09 9:49 pm'
uuid: 7ea76588-9cf5-44a6-9c48-28ddf4d9097c
permalink: /articles/web-hosting-with-caddy/
---

DRAFT
-----

And if you want to add special options you can do so

```bash
sudo mkdir -p /srv/www/quiet-goat-64.testing.letssecure.org

sudo chown -R $(whoami) /usr/local
sudo chown -R $(whoami) /srv/www
```

`/srv/www/Caddyfile`:
```
# handles both https and http on default ports
quiet-goat-64.testing.letssecure.org {
  root / /srv/www/quiet-goat-64.testing.letssecure.org/
}
```

```
sudo setcap cap_net_bind_service=+ep ./caddy
```

`/etc/init/caddy.conf`:
```
description "Caddy Server startup script"
author "Mathias Beke"

start on runlevel [2345]
stop on runlevel [016]


#setuid some-caddy-user
#setgid some-caddy-group

respawn
respawn limit 10 5

script
    cd /srv/www/
    exec /usr/local/bin/caddy
end script
```

Much thanks to [DenBeke's Running Caddy Server as a service](http://denbeke.be/blog/servers/running-caddy-server-as-a-service/)
