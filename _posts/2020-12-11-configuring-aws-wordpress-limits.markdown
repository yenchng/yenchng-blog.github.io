---
layout: post
title:  "Configuring AWS Wordpress Limits"
date:   2020-12-11 21:23:00 +0800
categories: guides
---

Took me a while to figure out a stable AWS wordpress hosting. Had memory leaks for first 3 weeks and poking around almost all settings.


What you'll need:
- AWS Account
- PHP and Wordpress installed

Useful References
- https://www.dreamhost.com/wordpress/guide-to-wp-functions/
- https://geekflare.com/php-fpm-optimization/
- http://www.isthisyourhomework.com/how-to-fix-the-wordpress-white-screen-of-death/
- https://www.wpbeginner.com/wp-tutorials/fix-wordpress-memory-exhausted-error-increase-php-memory/

Files to edit:
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


