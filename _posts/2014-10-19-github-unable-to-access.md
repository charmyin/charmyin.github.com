---
layout: post
title: "Solution for --- fatal: unable to access 'https://github.com/charmyin/DS1307.git/': "
description: "fatal: unable to access 'https://github.com/charmyin/DS1307.git/': server certificate verification failed. CAfile: /etc/ssl/certs/ca-certificates.crt CRLfile: none"
category: [github]
tags: [github, git]
---

##Problem:

		fatal: unable to access 'https://github.com/charmyin/DS1307.git/': server certificate verification failed. CAfile: /etc/ssl/certs/ca-certificates.crt CRLfile: none

##Solution

````git config http.sslVerify false````
