---
title: Going Deep with LetsEncrypt WebRoot
description: ''
date: '2015-12-10 0:46 am'
uuid: 9ae04f98-dba2-4e91-9138-714a4d911176
permalink: /articles/going-deep-with-letsencrypt-webroot/
---

STUB
=====

This article will be coming within the next few days. Here's the synopsis:

The LetsEncrypt protocol has one
[highly debated](https://github.com/letsencrypt/acme-spec/issues/33) and critical flaw:
it requires the use of ports 443 and 80 - the very same ports that your webserver
desperately needs to do its own job.

After a lot of though and consideration the LetsEncrypt group members decided on this compromise:

  * the LE python client can initiate the registration to the ACME go server
  * the LE python client writes some files to a directory you specify (the website public root)
  * the ACME go server will validate the challenge through your webserver
  * the LE python client will complete the registration logic

more to come...
