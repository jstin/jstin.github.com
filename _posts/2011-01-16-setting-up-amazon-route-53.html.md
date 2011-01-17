---
layout: default
title: Setting Up Amazon Route 53 on Amazon EC2
---

<h1>{{ page.title }}</h1>

So you need DNS for you EC2 instance, and don't want to run it yourself, 'cause that costs lostsa money? Whatever the reason, here are the steps I used.

After trying to use the Amazon perl script, and not having any success, I decided to go the Ruby approach. This [fine gem](https://github.com/pcorliss/ruby_route_53) is just what we need!

    sudo gem install route53
    
Good, assuming you have your elastic IP for your ec2 instance (that is easy to setup in Amazon's EC2 management pane), this is what you need to do for the domain of your choice.

    route53 -n internetmodulation.com.
    
The first time you run this gem, it will ask for your Amazon Credentials, so enter them. The other values should work as default.

Now we need just one more command. Add the `A` record like this.

    route53 --zone internetmodulation.com. -c --name internetmodulation.com. --type A --ttl 3600 --values 127.0.0.1
    
Of course, replace `127.0.0.1` with your actual IP address. Now to list the nameservers:

    route53 -l internetmodulation.com.
    
That's it! Add those four nameservers to your domain. 