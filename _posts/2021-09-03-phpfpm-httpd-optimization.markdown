---
layout: post
title:  "PHP-FPM and HTTPD Optimization"
date:   2021-09-03 16:54:33 +0800
categories: test
---

http://www.ansoncheunghk.info/article/3-simple-steps-optimize-apache-aws-ec2-micro-instance
Amazon Web Services EC2 micro instances are a good way to test and develop a LAMP application. However, the default settings for Apache, MySQL and PHP on a Linux AMI (Amazon Managed Image), tends to have memory settings too high for using on a micro EC2 instance. Which can cause the LAMP application to crash on a memory error.

Here are 3 simple steps to optimize AWS EC2 micro instance:

Enlarge PHP memory_limit
Open the php.ini file [nano /etc/php.ini].

Search for memory_limit, change the memory_limit to 128M.

Optimize PHP Memory_limit

Exit and save the php.ini file.

Edit Apache prefork.c configuration
Open the apache configuration file [nano /etc/apache2/apache2.conf].



Inside httpd.conf, edit the following values in the prefork.c module tag:

<IfModule prefork.c>

 StartServers 3

 MinSpareServers 2

 MaxSpareServers 5

 ServerLimit 10

 MaxClients 10

 MaxRequestsPerChild 100

</IfModule>
Exit and save the httpd.conf file.

Edit the MySQL configuration
edit the mysql configuration file [nano /etc/my.cnf]

Add the line under the [mysqld] list:



innodb_buffer_pool_size = 1M



Exit and save.

We are done! Finally restart the Apache, MySQL servers.

[ignore]
http://httpd.apache.org/docs/2.4/mod/worker.html
https://community.intersystems.com/post/apache-httpd-web-server-configuration-healthshare
https://medium.com/@bagasdotme/serving-php-on-apache-httpd-with-php-fpm-224a98c7401e
https://ma.ttias.be/a-better-way-to-run-php-fpm/
https://www.tecmint.com/improve-php-fpm-performance/
