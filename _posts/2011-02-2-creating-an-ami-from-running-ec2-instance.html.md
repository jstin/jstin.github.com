---
layout: default
title: Creating an AMI from a Running EC2 Instance on Ubuntu
---

<h1>{{ page.title }}</h1>

Once you have your instance set up, do the following on that instance. We can install the needed tools from apt-get. Since the are not in universe, we need to update things. Install it this way:

    sudo sed -i.dist 's,universe$,universe multiverse,' /etc/apt/sources.list
    sudo apt-get update
    sudo apt-get install ec2-api-tools ec2-ami-tools
    
Perfect. Now we need to create the image. You need the public and private key files. So `scp` them onto the server, and stick them in `/mnt/creds`. Why? When we bundle the image, anything in `/mnt` will *not* be copied.

Now create the folder for the image:

    sudo mkdir /mnt/MYIMAGE
    
Now make the image

    sudo ec2-bundle-vol -d /mnt/MYIMAGE --privatekey /mnt/creds/pk-whatever.pem --cert /mnt/EC/cert-whatever.pem -u your_amazon_user_id -r i386 or x86_64 -p the_name_you_want_to_use
    
Upload the image to S3. 

    sudo ec2-upload-bundle -b your_bucket_name -m /mnt/MYIMAGE/the_name_you_want_to_use.manifest.xml -a your_access_key -s your_private_key
    
Now in the [Amazon EC2 console](https://console.aws.amazon.com/ec2/home) click AMIs, and create a new one. The path will be `:80/your_bucket_name/the_name_you_want_to_use.manifest.xml`

All done!
    