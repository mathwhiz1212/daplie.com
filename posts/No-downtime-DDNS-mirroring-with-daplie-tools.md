---
title: 'Easy, no downtime, mirroring using daplie-tools and DDNS.'
description: "Easy, no downtime, mirroring using daplie-tools and DDNS."
permalink: /articles/No-downtime-DDNS-mirroring-with-daplie-tools/
---

**NOTE:** For this tutorial, you will want to replace **[devicename]** with your device name and **[ipaddress]** with either your IP address or a dummy one. You will want to replace **[example.daplie.me]** with your domain. You will need to replace **[tokenurl]** with your token URL you get in [this section](#your-device-token).

# Requirements

* You need daplie-tools. If you don't have it, you can find the install instructions here: [https://github.com/daplie/daplie-tools#install](https://github.com/daplie/daplie-tools#install)

* You need a domain name set up in daplie-tools. If you haven't done that yet, you can find out how to set it up here: [https://github.com/daplie/daplie-tools#purchase-your-own-com-org-net-etc](https://github.com/daplie/daplie-tools#purchase-your-own-com-org-net-etc)

* You will need screen, you can install this using `sudo apt-get install screen`

# Create a device

This command creates a device and assigns it an IP address. The IP address will be changed later.

```
daplie devices:set --device '[devicename]' --addresses '[ipaddress]'
```

# Attach a Device

This command attaches your device to the domain name you set up previously in daplie-tools. You will want to attach your devices in the order you want them to fall back in, the first device last. For instance, if I have rp1 and rp2 and I want to have rp1 as the primary device and for it to fall back on rp2 if rp1 goes offline then I would attach rp2 first then attach rp1.

```
daplie devices:attach --device '[devicename]' -n '[example.daplie.me]'
```

# Your Device Token

This command will display your device's token. This is needed to update your DNS record.

```
daplie devices:token --device '[devicename]'
```

Example output:

```
https://oauth3.org/api/com.enom.reseller/ddns?token=askldfjasdlkf556klasalskjfsa908903458e0989sdjflksdjfklkaskldfjasdlkf556klasalskjfsa908903458e0989sdjflksdjfklkaskldfjasd
```

# Update DNS

We need to update the DNS using the URL. A simple solution is a BASH script that runs every 5 minutes.

Create a file named ddns.sh

Paste this into it:

```
while :
do

#Replace tokenurl with your device token URL.
wget [tokenurl]

#Every 5 minutes.
sleep 600
done
```

Then run `chmod +x ddns.sh` and screen `./ddns.sh`

Repeat for every device.

# Conclusion

You should have MDDNS (Mirrored Dynamic DNS) set up. If any one of them goes offline the other should *immediately* take its place. No downtime! In my test I had 2 points, if one went offline the 2nd point would remain.

Hi, I'm Josh Mudge. If you find what I do useful please visit my blog which is coming soon at [https://ltlto.com/](https://ltlto.com/)
