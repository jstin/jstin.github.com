---
layout: default
title: Capistrano Deploys with Passenger on Ubuntu
---

<h1>{{ page.title }}</h1>

So basically, if we use Capistrano's default deploy directory (I don't know why you would, but I don't know why you wouldn't), this is what we need to do. This will run one site on one server.

Add the site to nginx.

    sudo nano /opt/nginx/conf/nginx.conf
    
Throw in something like this. Remove the existing site on port 80 (if there is one).

    server {
        listen 80;
        root /u/apps/magicapp/current/public;
        passenger_enabled on;
    }

Now restart nginx:

    sudo /etc/init.d/nginx restart
    
Back on the local machine run

    capify .
    
Ok, lets deploy the code. Assuming it is on github, here is a recipe for ./config/deploy.rb:

    require 'bundler/capistrano'

    set :application, "magicapp"
    set :repository,  "git@github.com:you/magicapp.git"

    set :scm, :git
    set :deploy_via, :remote_cache
    default_run_options[:pty] = true
    ssh_options[:forward_agent] = true

    role :web, "ubuntu@whatever.com"                  
    role :app, "ubuntu@whatever.com"      

    namespace :deploy do
      task :start do ; end
      task :stop do ; end
      task :restart, :roles => :app, :except => { :no_release => true } do
       run "#{try_sudo} touch #{File.join(current_path,'tmp','restart.txt')}"
      end
    end
    
Locally, run this:

    cap deploy:setup
    
On production run this:

    sudo chown -R ubuntu:ubuntu /u/
    sudo chown -R ubuntu:ubuntu /home/ubuntu/.gem
    
Locally run this:

    cap deploy:check
    cap deploy:update
    cap deploy
    
On production run this inside the app/current directory:

    rake RAILS_ENV=production db:schema:load
    
Now run this locally:

    cap deploy:restart
    
Go to whatever.com and enjoy!