---
layout: post
title: "Install mysql on linux"
description: "Install mysql on linux"
category: [mysql]
tags: [mysql, linux]
---

---------------------------------------

####Ubuntu
**Install**
1.

````apt-get install mysql-server````

**Config for remote access**

1.vim /etc/my.cnf

Comment this line：bind-address=127.0.0.1 ==> #bind-address=127.0.0.1

2.Login

````mysql -u root -p"youpassword"````

3.Authorize

````GRANT ALL PRIVILEGES ON *.* TO root@"172.16.16.152" IDENTIFIED BY "youpassword" WITH GRANT OPTION;````

4.Reload authorization table

````FLUSH PRIVILEGES;````

5.exit mysql

````exit````

###Error case sensitive

Log in as root: in /etc/my.cnf or /etc/myql/my.cnf, After [mysqld], append

````lower_case_table_names=1````

eg: vi /etc/my.cnf

6.Open linux port!


