---
layout: default
title: Wordpress Blog with Nginx and Passenger in single Rails App
---

<h1>{{ page.title }}</h1>

Ok, say you want a blog on your site. Say you want wordpress. Well that is cool, but your running rails with passenger and nginx. How will you do this? Just like I do.

Install php:

    sudo apt-get install php5-cli php5-cgi php5-common php5-curl php5-dev php5-gd php5-imagick php5-mcrypt php5-mhash
    
Assuming MySQL is installed:

    sudo apt-get install php5-mysql libmysqld-dev
    
Install spawn-fcgi:

    /tmp
    wget http://www.lighttpd.net/download/spawn-fcgi-1.6.3.tar.gz
    tar -xvzf spawn-fcgi-1.6.3.tar.gz
    cd spawn-fcgi-1.6.3
    ./configure --prefix=/usr
    make
    sudo make install
    
Make this file:

    sudo nano /usr/bin/php5-fcgi

Put this in it:
    
    /usr/bin/spawn-fcgi -f /usr/bin/php-cgi -a 127.0.0.1 -p 49232 -C 2 -P /var/run/fastcgi-php.pid

And make this init file: 

    sudo nano /etc/init.d/php5-fcgi
    
With this in it:

    #!/bin/sh
    PHP_SCRIPT=/usr/bin/php5-fcgi
    RETVAL=0
    case "$1" in
            start)
                    echo "Starting fastcgi"
                    $PHP_SCRIPT
                    RETVAL=$?
      ;;
            stop)
                    echo "Stopping fastcgi"
                    killall -9 php-cgi
                    RETVAL=$?
      ;;
            restart)
                    echo "Restarting fastcgi"
                    killall -9 php-cgi
                    $PHP_SCRIPT
                    RETVAL=$?
      ;;
            *)
                    echo "Usage: php-fastcgi {start|stop|restart}"
                    exit 1
      ;;
    esac
    exit $RETVAL
    
Set these files up:

    sudo chmod +x /usr/bin/php5-fcgi
    sudo chmod +x /etc/init.d/php5-fcgi
    sudo update-rc.d php5-fcgi defaults
    
Ok, assuming you already installed nginx on this system, we need to rebuild it with apache. I already have it downloaded in `/tmp`

    sudo /opt/ruby-enterprise-1.8.7-2010.02/bin/passenger-install-nginx-module
    
Choose option 2 and use these arguments
    
    --with-http_ssl_module --with-http_dav_module --with-http_gzip_static_module --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-client-body-temp-path=/var/lib/nginx/body --with-http_stub_status_module
    
Assuming you have a wordpress blog located at `RAILS_APP/public/blog/` use this nginx config

    server {
        listen 80;
        root /u/apps/magicapp/current/public;

        location / {
            passenger_enabled on;
        }

        location /blog {
            passenger_enabled off;
            index index.html index.php;
            try_files $uri $uri/ /blog/index.php?q=$uri;
        }

        location ~ \.php$ {
            passenger_enabled off;
            fastcgi_pass 127.0.0.1:49232;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include /opt/nginx/conf/fastcgi_params;
        }
    }
    
To start everything do the following. Note, I had to create the nginx directory as shown.

    sudo mkdir /var/lib/nginx/
    sudo /etc/init.d/php5-fcgi start
    sudo /etc/init.d/nginx restart
    
The magic should ensue.
