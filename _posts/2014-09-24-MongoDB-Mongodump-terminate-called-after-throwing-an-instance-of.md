---
layout: post
title: "MongoDB: Mongodump terminate called after throwing an instance of ‘std::runtime_error’"
description: "MongoDB: Mongodump terminate called after throwing an instance of ‘std::runtime_error’"
category: [mongodb]
tags: [mongodb]
---

**MongoDB: Mongodump terminate called after throwing an instance of ‘std::runtime_error’**

If you encounter this error:

 
	connected to: 127.0.0.1
	Mon Oct 21 10:49:30.638 DATABASE: soft_production to dump/soft_production
	terminate called after throwing an instance of 'std::runtime_error'
	what(): locale::facet::_S_create_c_locale name not valid
	Aborted

Please add this

 
````export LC_ALL=C````

 or

````export LC_ALL="en_US.UTF-8"````

either in the console (for current session) or in .bashrc file.

After that you should be ready to go with:

 
````mongodump --db soft_production````