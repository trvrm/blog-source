Title: Streaming PHP Output with FPM and NGINX
Category: Systems
Tags: PHP, FPM, NGINX
Author: Trevor
Date: 2015-10-25

**Problem:** We had a PHP report that takes around 45 seconds to render.  To give the user feedback
that their report was actually being generated, I wanted to change our PHP installation to deliver at least some output to the client as soon as possible, rather than waiting until the entire page was rendered.

In general, PHP and NGINX work hard to buffer output, so this was mostly an exercise in fighting with
PHP and NGINX to make them stop doing optimizations that do, in general, make sense.  It turns out that
to do this, we had to make changes at three levels of our software stack: in our PHP code,
in our PHP configuration settings, and in our NGINX configuration.


#### PHP changes

I added this at the top of the file in question:

    ob_implicit_flush(1);   


This tells PHP to simulate calling ``flush()`` after every output block.


#### PHP INI changes

I added a new .ini file in `/etc/php5/fpm/conf.d` with the following setting:

    output_buffering = Off

This tells PHP not to buffer output.


#### NGINX changes

In the .conf file for the reporting site, I added


    fastcgi_keep_conn on;
    gzip off;

in the location block, and

    ssl_buffer_size 1k;

in the server block. This last one took me a while to figure out - it defaults
to 16k, and even with all the other changes made, when accessing the site via
HTTPS, NGINX will still buffer your output.

#### References

[This](https://stackoverflow.com/questions/4870697/php-flush-that-works-even-in-nginx)
Stack Overflow question got me pointed in the right direction. 