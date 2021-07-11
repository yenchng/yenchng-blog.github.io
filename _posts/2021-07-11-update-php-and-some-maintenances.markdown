---
layout: post
title:  "Some maintenaces"
date:   2020-07-11 22:14:00 +0800
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

1) WORDPRESS >> wp-config.php file.
{% highlight php %}
define('WP_MEMORY_LIMIT', '256M');
{% endhighlight %}

2) WWW >> check current settings via phpinfo() - create dummy php page in www/ folder
{% highlight php %}
<?php
// Show all information, defaults to INFO_ALL
phpinfo();
?>
{% endhighlight %}

3) PHP >>PHP.ini file
{% highlight php %}
memory_limit = 256M;
upload_max_size = 64M;
post_max_size = 64M;
upload_max_filesize = 64M;
max_execution_time = 300;
max_input_time = 1000;
{% endhighlight %}

4) WORDPRESS >> functions.php file
-- generalise login error messages
{% highlight php %}
function no_wordpress_errors(){
return 'Something went wrong!';
}
add_filter( 'login_errors', 'no_wordpress_errors' );
{% endhighlight %}

-- estimated reading time
{% highlight php %}
function reading_time() {
$content = get_post_field( 'post_content', $post->ID );
$word_count = str_word_count( strip_tags( $content ) );
$readingtime = ceil($word_count / 200);
if ($readingtime == 1) {
$timer = " minute";
} else {
$timer = " minutes";
}
$totalreadingtime = $readingtime . $timer;
return $totalreadingtime;
}
{% endhighlight %}


5) WORDPRESS >> THEME >> template-parts > post > content.php.
-- look for places where you want the reading time estimate to show. Meta data is a good place to start.
{% highlight php %}
echo reading_time();
{% endhighlight %}



6) PHP >> php-fpm.conf (mine was /etc/php-fpm.conf)
-- this is to optimize PHP performance, and was the solution to my never-ending memory leak
read more: https://geekflare.com/php-fpm-optimization/
-- file looks something like this
{% highlight php %}
 ;;;;;;;;;;;;;;;;;;;;;
; FPM Configuration ;
;;;;;;;;;;;;;;;;;;;;;

; All relative paths in this configuration file are relative to PHP's install
; prefix (/usr). This prefix can be dynamically changed by using the
; '-p' argument from the command line.

;;;;;;;;;;;;;;;;;;
; Global Options ;
;;;;;;;;;;;;;;;;;;

[global]
; Pid file
; Note: the default prefix is /var
; Default Value: none
pid = /run/php/php7.2-fpm.pid

; Error log file
; If it's set to "syslog", log is sent to syslogd instead of being written
; into a local file.
; Note: the default prefix is /var
; Default Value: log/php-fpm.log
error_log = /var/log/php7.2-fpm.log

{% endhighlight %}

-- Look for these parameters and set them accordingly. Tweak as needed.
{% highlight php %}
emergency_restart_threshold 10
emergency_restart_interval 1m
process_control_timeout 30s
{% endhighlight %}


7) 6) PHP >> php-fpm.d >> www.conf
-- this is to optimize PHP performance, and was the solution to my never-ending memory leak
read more: https://geekflare.com/php-fpm-optimization/
-- file looks something like this

{% highlight php %}
pm = dynamic
pm.max_children = 10
pm.start_servers = 3
pm.min_spare_servers = 2
pm.max_spare_servers = 4
;pm.max_requests = 200

{% endhighlight %}

Enjoy !
