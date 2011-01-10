---
layout: default
title: Amazon EC2 Ubuntu Install
---

<h1>{{ page.title }}</h1>

Some notes on installing an Ubuntu Server on a fresh EC2 instance. Get your Amazon credentials handy, and log into [AWS Consoles](https://console.aws.amazon.com/ec2/home).

Once you open the Management Console, you will be presented with the tools needed to get the job done. You also need to create a keypair. You should also set up your 'Security Groups' to allow ssh (whatever port you prefer) and http (port 80).

Copy the key pair file to ~/.ssh/ and run ssh-add on the file (you may need to chmod 700)

Note that the default user associated with the keypair is not *root* it is *ubuntu*.

Click 'Launch Instance' and follow the steps. For a small instance on the east coast I use `ami-88f504e1` (Ubuntu 10.04 32-bit) from Community AMIs. Once it is running, you can find the public DNS record by selecting it. To get onto your new machine:

  ssh -p 22 ubuntu@whatever-this-may-be

Run the following

	sudo apt-get update && sudo apt-get upgrade -y
	
Now we need to make sure this system is locked down. Edit the following file with:

    sudo nano /etc/ssh/sshd_config

Find this line

    PermitRootLogin yes
  
and change it to 

    PermitRootLogin no
  
You can also change your port number here as well. It is advisable to change it so something else, but it must be above 1025.

The next step is to modify the iptables file.

    sudo nano /etc/iptables.up.rules
  
Add this to the file. Make sure to use your selected port for ssh.

    *filter

    -A INPUT -i lo -j ACCEPT
    -A INPUT ! -i lo -d 127.0.0.0/8 -j REJECT

    -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

    -A OUTPUT -j ACCEPT

    -A INPUT -p tcp --dport 80 -j ACCEPT
    -A INPUT -p tcp --dport 443 -j ACCEPT

    # Put your SSH port here
    -A INPUT -p tcp -m state --state NEW --dport 22 -j ACCEPT

    -A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT

    -A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables denied: " --log-level 7

    -A INPUT -j REJECT
    -A FORWARD -j REJECT

    COMMIT
  
Save that file. Commit it by running:

    sudo /sbin/iptables-restore < /etc/iptables.up.rules
  
To make sure these rules are loaded at boot, create the following file.

    sudo nano /etc/network/if-pre-up.d/iptables
  
The file should contain the following line:

    /sbin/iptables-restore < /etc/iptables.up.rules
  
Save the file and run:

    sudo chmod +x /etc/network/if-pre-up.d/iptables
  
To apply all your changes run:

    sudo /etc/init.d/ssh reload
  
Ok, your server is more secure now. Time to make it useful. 

Stay tuned for future articles on adding Passenger, nginx, and Ruby.