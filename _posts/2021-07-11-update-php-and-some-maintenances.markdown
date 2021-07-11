---
layout: post
title:  "Some maintenaces"
date:   2021-07-11 22:14:00 +0800
categories: guides
---

Some typical maintenance references


What you'll need:
- AWS Account
- PHP and Wordpress installed

Useful References
- https://greggborodaty.com/amazon-linux-2-upgrading-from-php-7-2-to-php-7-4/
- https://forums.aws.amazon.com/thread.jspa?threadID=305528
- https://techviewleo.com/install-php-7-on-amazon-linux/

Files to edit:

+++PHP UPDATE+++
- Check for update to make sure the system is up to date as is.
{% highlight terminal %}
$ sudo yum update
{% endhighlight %}

- Check your php version to confirm it is running PHP 7.2
{% highlight terminal %}
$ php -v
{% endhighlight %}
If running PHP 7.2, you should see something similar to the following: 7.2.xx

- Check to make sure that the amazon-linux-extras package is installed.
{% highlight terminal %}
$ which amazon-linux-extras
{% endhighlight %}

You should see some response like:  /usr/bin/amazon-linux-extras
- Check the PHP7.x topic is available in Amazon Linux 2:
{% highlight terminal %}
$ sudo amazon-linux-extras | grep php
{% endhighlight %}

It should show a list of [enabled] or [disabled] topics. You'll need to disable old topics and enable new ones. 
Make sure only the new ones are marked as enabled with the rest being available.

- disable both the php7.2 and lamp-mariadb10.2-php7.2 topics
{% highlight terminal %}
$ sudo amazon-linux-extras disable php7.2
$ sudo amazon-linux-extras disable lamp-mariadb10.2-php7.2
$ sudo amazon-linux-extras enable php7.4
{% endhighlight %}

- Recheck the PHP7.x availability again
- Update to PHP 7.4 by first cleaning up the metadata and then installing php along with the necessary php packages.
{% highlight terminal %}
$ sudo yum clean metadata
$ sudo yum install php php-{pear,cgi,common,curl,mbstring,gd,mysqlnd,gettext,bcmath,json,xml,fpm,intl,zip}
{% endhighlight %}

------------------------------------------------------------------------------------------------

+++ REMINDER ON IMPORTANT PATHS +++
{% highlight terminal %}
/var/www/html/
/etc/php-fpm.d/
{% endhighlight %}

+++ REMINDER ON IMPORTANT COMMANDS +++
{% highlight terminal %}
Text Editor: nano xxxx

{% endhighlight %}

Enjoy !
