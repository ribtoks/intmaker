---
title: TeamCity 8 on CentOS 6.4 from scratch
date: 2013-11-26T16:22:43+00:00
author: "Taras Kushnir"
permalink: /teamcity-8-on-centos-6-4-from-scratch/
categories:
  - Linux
  - Programming
tags:
  - autostart
  - centos
  - config
  - continuous integration
  - database
  - how-to
  - init
  - init.d
  - installation
  - iptables
  - jetbrains
  - linux
  - postgresql
  - scratch
  - script
  - startup
  - teamcity
  - tomcat
  - tutorial
---
In this post I'll describe whole TeamCity installation process on a fresh CentOS (on the moment I'm writing it's 6.4) for your private project or startup.

First, proceed to the <a title="Download CentOS" href="http://www.centos.org/modules/tinycontent/index.php?id=30" target="_blank">CentOS downloads page</a> and download CentOS distro through <em>.iso</em>, <em>.torrent</em> or <em>whatever-you-like</em> and start the installation process. If you're familiar with any  linux distro installation, it won't take you much time to complete the installation. I'm using the minimal configuration without any Desktop Environment and with minimum core- and other system utils. Nonetheless, consider installing <em>iptables, elinks and java (openjdk)</em>.

After the installation is over, login, add a user for yourself with <em>adduser</em> and let's start with TeamCity (maybe you'll also consider adding yourself to the sudoers file).

First, create <em>/opt/jetbrains/</em> directory and <a title="Download teamcity" href="http://www.jetbrains.com/teamcity/download/index.html" target="_blank">download latest TeamCity</a> using <em>wget</em>.

```shell
mkdir /opt/jetbrains
cd /opt/jetbrains
wget http://download.jetbrains.com/teamcity/TeamCity-8.0.5.tar.gz
tar -xpzf TeamCity-8.0.5.tar.gz
```

Now lets create a system user (e.g. no home directory) for TeamCity to resolve security issues correctly:

```shell
adduser -r teamcity
passwd teamcity
chown -R teamcity:teamcity TeamCity
```

We'll use PostgreSQL for the internal database for TeamCity. So let's install postgresql packages (8.4 for CentOS 6.4). For details of the PostgreSQL installation please refer to <a title="PostgreSQL installation" href="http://wiki.postgresql.org/wiki/YUM_Installation" target="_blank">official webpage</a>.

```shell
yum install postgresql
service postgresql initdb
chkconfig postgresql on
```
  
Also edit the <em>/var/lib/pgsql/data/pg_hba.conf</em> file to allow authorization from <em>localhost</em>. Go to the end and replace <em>ident</em> to <em>trust</em> for <em>localhost</em> in the configuration for hosts. Now lets create another user for the future TeamCity database and create a database for that user.
  
```shell
sudo -u postgres psql postgres
password postgres
sudo -u postgres createuser -D -A -P teamcity_user
sudo -u postgres createdb -O teamcity_user teamcity_db
```
  
(D = Cannot create databases, A = Cannot add users, P = Force password prompt)
  
<!--more-->
  
To start TeamCity as a service in CentOS, you'll have to register it with <em>chkconfig</em> and to do so, let's create startup script in the <em>/etc/init.d/</em> directory and save it with any name (e.g. teamcity-script):
  
```shell
#!/bin/bash
#
# chkconfig: 235 10 90
# description: TeamCity startup script
#

TEAMCITY_USER=teamcity
TEAMCITY_DIR=/opt/jetbrains/TeamCity/
TEAMCITY_SERVER=bin/teamcity-server.sh

TEAMCITY_DATADIR="/opt/jetbrains/TeamCity/.BuildServer"

. /etc/rc.d/init.d/functions

case "$1" in
start)
    sudo -u $TEAMCITY_USER -s -- sh -c "cd $TEAMCITY_DIR; <strong>TEAMCITY_DATA_PATH</strong>=$TEAMCITY_DATADIR $TEAMCITY_SERVER start"
    ;;
stop)
    sudo -u $TEAMCITY_USER -s -- sh -c "cd $TEAMCITY_DIR; TEAMCITY_DATA_PATH=$TEAMCITY_DATADIR $TEAMCITY_SERVER stop"
    ;;
*)
    echo "Usage: $0 {start|stop}"
    exit 1
    ;;
esac

exit 0
```
  
Note the <em>TEAMCITY_DATA_PATH</em> system variable which is used to overwrite default <em>~/.BuildServer</em> path for the teamcity data. First commented lines exist for <em>chkconfig</em> compatability. Now you're able to register it:
  
```shell
chkconfig --add teamcity-script
chkconfig teamcity-script on
```
  
Now our script would autostart on system startup in 2, 3 and 5 runlevels.
  
But what does actually <em>teamcity-server.sh</em> script do? It's located in the <em>bin/</em> directory of the TeamCity installation directory and it launches Tomcat server with initial config (<em>conf/server.xml</em> file). This config can be useful for us, so let's modify it slightly:
  
* Find the Connector node and change port to whatever you like (but don't use port 80 - to be discussed later in this post)
* Find the Host node and change <em>name</em> attribute from the <em>localhost</em> to whatever you like host, which also should be duplicated in the <em>/etc/hosts</em> file (e.g. if you host's name is <em>my_server.my_domain</em>, than the <em>/etc/hosts</em> file should also contain <em>"127.0.0.1 my_server.my_domain"</em>)
* Host element has also one useful child node: Alias. You can add <em><Alias>other_server_name</Alias></em> to use aliasing. See <a title="Tomcat aliasing" href="http://tomcat.apache.org/tomcat-6.0-doc/config/host.html#Host_Name_Aliases" target="_blank">official Tomcat documentation on this topic</a>.
* If your server is being accessed through proxy (or it's running on a virtual machine with forwarding from host), you'll have to uncomment last Valve node in the Host config and change the <em>internalProxies</em> respectively to your proxy.
  
Another important thing is <em>iptables</em>. This is a server, right? So let's configure a firewall properly. You can paste below config as a working one for a server in the <em>/etc/sysconfig/iptables</em> file:
  
```
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p ah -j ACCEPT
-A INPUT -p esp -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 443 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 53 -j ACCEPT
-A INPUT -m state --state NEW -m udp -p udp --dport 53 -j ACCEPT
<strong>-A INPUT -m state --state NEW -m tcp -p tcp --dport 8111 -j ACCEPT</strong>
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
```
The most valuable line for us is highlighted and allows packets on <strong>8111</strong> port (the one Tomcat is listening to). You can easily configure Tomcat to listen to another port in the <em>conf/server.xml</em> file on your TeamCity installation. You can also find lots of articles on the web about making a proxy with <em>apache</em> or <em>lighttpd</em> for Tomcat just in order to enter urls like <em>http://my-server/teamcity</em> instead of <em>http://my-server:8111/teamcity/</em>. Believe me, <del>java and tomcat are not worth normal url in your browser! Besides,</del> you'll have to spend your time to deal with several issues:

* If you want make Tomcat listen to 80 port (in the Connector node in <em>conf/server.xml</em>), you would have to launch <del>TeamCity</del> Tomcat from a <em>root</em> user instead of non-privileged, because only root can listen to 80 port
* You'll have to spend plenty of time looking for proper configuration when trying to configure a proxy for Tomcat regarding build agents which would need special configuration too (just like the server)

So, I assume, you have enough reasons to leave the 8111 port as the default one. Let's ensure that <em>iptables</em> function properly:

```  
# service iptables restart
# chkconfig iptables on
# service teamcity-script start
```
Now you can launch a browser and navigate to <em>http://my_server.my_domain:8111 </em>(TeamCity data is stored in <em>webapps/ROOT</em> directory and you're able to move it to some other like <em>webapps/teamcity</em> but in this case your address would be <em>http://my_server.my_domain:8111/teamcity</em>) and see TeamCity page with a correct DATA_PATH (check if it's same like the one you specified in the <em>/etc/init.d/teamcity-script</em>). Check if user <em>teamcity</em> is able to create a directory DATA_PATH.  If everything is correct, proceed, agree to the license terms and wait while TeamCity will finish with basic installation. Please, note, that for now TeamCity is using internal DB which is not for the production purposes! Let's fix this using the <a title="PostgreSQL for TeamCity" href="http://confluence.jetbrains.com/display/TCD8/Setting+up+an+External+Database" target="_blank">official how-to</a> by downloading proper postgresql jdbc driver (for postgresql version on current CentOS) and configuring PostgreSQL authentication:
  
```shell
service teamcity-script stop
cd YOUR_DATA_PATH
cd lib/jdbc
su - teamcity
wget jdbc.postgresql.org/download/postgresql-8.4-703.jdbc4.jar
cd ../../config/
cp database.postgresql.properties.dist database.properties
```
  
 Now edit <em>database.properties</em> with you favorite editor (say, <em>vi</em>) and replace placeholder credentials with those you've specified in the PostgreSQL post-installation configuration.

```
connectionUrl=jdbc:postgresql://<host>/<database name>
connectionProperties.user=<user>
connectionProperties.password=<password>
```
After you're done, relaunch TeamCity:
```
service teamcity-script start
```
  
Open your browser, go to the teamcity page and you should see TeamCity complaining about missing database. You'll have only one-button choice, so proceed. Now teamcity will try to create another database (many thanks to <a title="TeamCity and YouTrack on Ubuntu" href="http://jerryemilo.com/2012/12/29/teamcity-and-youtrack-on-ubuntu-12-10-quantal-quetzal/" target="_blank">this tutorial</a>) but it will require an authorization key, which can be found at the end of <em>logs/teamcity-server.log</em> file in your TeamCity main directory. Copy it and paste in the browser textbox, press <em>Next</em> and now TeamCity should launch correctly with PostgreSQL in the backend.

A plenty of useful links:

* http://jerryemilo.com/2012/12/29/teamcity-and-youtrack-on-ubuntu-12-10-quantal-quetzal
* http://kogentadono.com/category/teamcity/
* http://stackoverflow.com/questions/4247685/teamcity-webserver-with-apache-proxy-get-method-get-not-implemented-try-post
* http://www.cyberciti.biz/tips/postgres-allow-remote-access-tcp-connection.html
* http://www.warp1337.com/content/scientifc-linux-centos-rhel-and-lighttpd-lighttpd-doesnt-start-solved
* http://zeroturnaround.com/rebellabs/rebel-labs-report-why-devs-love-ci-a-guide-to-loving-continuous-integration/5/
