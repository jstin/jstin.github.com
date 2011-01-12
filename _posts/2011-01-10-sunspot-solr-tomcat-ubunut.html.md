---
layout: default
title: Sunspot Solr and Tomcat on Ubuntu
---

<h1>{{ page.title }}</h1>

So you wanna have search on your server? Really? That is so 90's. *sigh*. Ok, here are the steps I used.

First install java and ant if you don't have it.

    sudo apt-get install openjdk-6-jdk ant
  
We are now gonna get tomcat6.

    sudo apt-get install tomcat6 tomcat6-admin tomcat6-common tomcat6-user tomcat6-docs
    sudo apt-get install libmysql-java
  
Test this

    sudo /etc/init.d/tomcat6 start
    sudo /etc/init.d/tomcat6 status
  
Now we need to allow our user to access this.

    sudo nano /etc/tomcat6/tomcat-users.xml
  
Add this to the file
  
    <role rolename="manager"/>
    <role rolename="admin"/>
    <user username="admin" password="SPECIAL" roles="manager,admin"/>

Ok now we need solr:

    cd /tmp
    wget http://www.bizdirusa.com/mirrors/apache//lucene/solr/1.4.1/apache-solr-1.4.1.tgz
    tar -xzvf apache-solr-1.4.1.tgz
    cd apache-solr-1.4.1/
  
Install into tomcat:

    sudo cp dist/apache-solr-1.4.1.war /var/lib/tomcat6/webapps/solr.war
    sudo cp -R example/solr/ /var/lib/tomcat6/solr/
  
Make config file:

    sudo nano /var/lib/tomcat6/conf/Catalina/localhost/solr.xml

Add this to it:

    <Context docBase="/var/lib/tomcat6/webapps/solr.war" debug="0" privileged="true" allowLinking="true" crossContext="true">
        <Environment name="solr/home" type="java.lang.String" value="/var/lib/tomcat6/solr" override="true" />
    </Context>

Data directory!

    sudo mkdir /var/lib/tomcat6/solr/data
    sudo chown -R tomcat6:tomcat6 /var/lib/tomcat6/solr/data/

Some permissions:

    sudo chown -R tomcat6:tomcat6 /var/lib/tomcat6/
    sudo chmod 775 /var/lib/tomcat6/conf/tomcat-users.xml
    sudo chmod 775 /var/lib/tomcat6/conf/Catalina/localhost/solr.xml
  
And restart

    sudo /etc/init.d/tomcat6 restart
  
To test this run

    curl http://127.0.0.1:8080/solr/admin/

One last thing, if you use rails, copy the config file to:

    sudo cp RAILS_ROOT/solr/conf/schema.xml /var/lib/tomcat6/solr/conf/schema.xml
    sudo /etc/init.d/tomcat6 restart