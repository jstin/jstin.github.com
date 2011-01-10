---
layout: default
title: Rails Stack with Passenger, nginx, and Ruby Enterprise Edition
---

<h1>{{ page.title }}</h1>

This is the coolest rails stack around. Just ask the cool kids about it. Odds are they'll ignore you...

Let's start with the basic packages needed. I'm running Ubuntu 10.04.

    sudo apt-get update
    sudo apt-get install build-essential patch zlib1g-dev libssl-dev libreadline5-dev
    sudo apt-get install libxml2 libxml2-dev libxslt1-dev
  
Now we should install the database of choice. My choice is MySQL, but your could be anything rails supports. Use a good password.

    sudo apt-get install mysql-server libmysqlclient15-dev
  
Lets install git. We'll use this later:

    sudo apt-get install git-core
  
One final dependency:

    sudo apt-get install libcurl4-openssl-dev
  
Thats good. We'll first start by creating a .gemrc file. This is a production server so we can speed things up by not creating rdocs and such. Create the file:

    nano ~/.gemrc

Add these lines and save:

    ---
    :sources:
    - http://gems.rubyforge.org
    - http://gems.github.com
    gem: --no-ri --no-rdoc
  
Now lets install Ruby. You might want to be hip and use [RVM](http://rvm.beginrescueend.com/). I'm a man who knows what he wants, so I'm just gonna install REE cold. Go to the [download page](http://www.rubyenterpriseedition.com/download.html) of Ruby Enterprise Edition, and find the most recent version. For me it is:

    http://rubyforge.org/frs/download.php/71096/ruby-enterprise-1.8.7-2010.02.tar.gz

So basically we do this

    cd /tmp
    wget http://rubyforge.org/frs/download.php/71096/ruby-enterprise-1.8.7-2010.02.tar.gz
    tar xvfz ruby-enterprise-1.8.7-2010.02.tar.gz
    cd ruby-enterprise-1.8.7-2010.02/
    sudo ./installer
  
Cool, it is installed. Now set it up:

    echo "export PATH=/opt/ruby-enterprise-1.8.7-2010.02/bin:$PATH" >> ~/.profile && . ~/.profile
  
Check your installation:

    ruby -v
    
We need the bundler gem so run this:

    sudo su
    export PATH=$PATH:/opt/ruby-enterprise-1.8.7-2010.02/bin
    gem install bundler
    exit
    
We want to be able use Capistrano so we need this line at the *top* of `nano ~/.bashrc`

    export PATH=/opt/ruby-enterprise-1.8.7-2010.02/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
  
Ok, now for nginx. Go to the [nginx page](http://nginx.org/en/download.html) to find the latest release. For me, well . . . just look at the code to find out!

    cd /tmp
    wget http://nginx.org/download/nginx-0.8.54.tar.gz
    tar xvfz nginx-0.8.54.tar.gz
    sudo /opt/ruby-enterprise-1.8.7-2010.02/bin/passenger-install-nginx-module
  
When the prompt for option 1 or option 2 comes, select 2. Choose the directory `/tmp/nginx-0.8.54`. We need SSL, so in the extra args part add `--with-http_ssl_module`

The next part is why we needed git (for this part, git is great for deployment as well).

    cd /tmp
    git clone git://github.com/jnstq/rails-nginx-passenger-ubuntu.git
    sudo mv rails-nginx-passenger-ubuntu/nginx/nginx /etc/init.d/nginx
    sudo /usr/sbin/update-rc.d -f nginx defaults
  
That will make nginx start at boot. Start it now with:

    sudo /etc/init.d/nginx start
  
Alright kiddos, that is all for now.
  