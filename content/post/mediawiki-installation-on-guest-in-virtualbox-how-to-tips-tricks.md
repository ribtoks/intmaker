---
title: 'MediaWiki installation on a linux guest in the VirtualBox: how-to, tips & tricks'
date: 2013-11-22T18:51:36+00:00
author: "Taras Kushnir"
permalink: /mediawiki-installation-on-guest-in-virtualbox-how-to-tips-tricks/
categories:
  - Linux
tags:
  - apache
  - linux
  - mediawiki
  - opensuse
  - postgresql
  - sendmail
  - server
  - virtualbox
  - virtualhost
  - wiki
---
I've spend half of a day trying to get MediaWiki working on a VirtualBox for the internal project in the internal network. Looks like it's done now and I want to share tips to help someone to spend less time in the future.

I've been installing and configuring MediaWiki 1.21 on the OpenSUSE 12.3, but all tips are valuable for any linux distro.

<!--more-->

**Installation**

I've installed apache2, mediawiki from the _php:extensions_ repository (with tons of dependencies) and php5-related packages (sqlite, xmlreader, zlib and others). The _mediawiki_ package has _mysql_ in dependencies which is annoying, because I want to use PostgreSQL.

Mediawiki installation creates _/var/lib/mediawiki_ directory, in which almost every file is a simulink to the _/usr/share/mediawiki_ corresponding file. After the package is installed and apache is restarted using _sudo rcapache2 restart,_ you're able to start a mediawiki instance installation through _http://localhost/w/_ path in your browser. I recommend you to install some additional packages before the actual mediawiki instance (not package) installation:

<pre><em>php5-xcache, php5-intl, postgresql, php5-postgresql, postgresql-server, sendmail, php5-pear-Mail</em></pre>

_XCache_ and _Intl_ are required for MediaWiki, PostgreSQL is better than MySQL, _sendmail_ and php mail extensions are needed for email confirmation from wiki.

Then I suppose you should install _phpPgAdmin_ and go to _http://localhost/phpPgMyAdmin_ to properly configure one. At the moment, you won't be able to login using the default username-password combination (postgres:postgres). To do so, you should make a bit of DB console hacking and _/var/lib/pgsql/data/pg_hba.conf_ hacking (described in <a href="http://blog.milczarek.it/2012/09/install-postgresql-on-opensuse-12-1/" target="_blank" class="broken_link">this article</a>).

After you're able to login to phpPgAdmin, you should create user for mediawiki db connection, say, _mediawiki_user_.

When you're done, go to _http://localhost/w/_ and start the wiki installation process.

**Configuring**

During installation choose PostgreSQL instead of MySQL. Enter username you've created in phpPgAdmin before. Then you would be asked for username and password for a MediaWiki user, so make sure you're remembered/backed them up, cause they are username-password of so-called _sysop_ user (e.g. main administrator of MediaWiki installation).

After you would succeed with MediaWiki intallation, let's make this Wiki available outside of the VirtualBox. First, go to _/etc/apache2/vhosts.d/_ directory and create two files. One named _\_default\_vhost.conf_ (with an underscore before name to be the default VirtualHost entry - read more in the <a href="http://activedoc.opensuse.org/book/opensuse-reference/chapter-20-the-apache-http-server" target="_blank" class="broken_link">OpenSUSE apache docs</a> and <a href="http://httpd.apache.org/docs/2.2/vhosts/" target="_blank">Official apache docs</a>) with the following contents:

<pre><VirtualHost *:80>
    ServerName localhost
    DocumentRoot /srv/www/htdocs
</VirtualHost></pre>

And the second one, say, _your\_website\_wiki.conf_:

<pre><VirtualHost *:80>
    ServerAdmin your.name@mail.com
    ServerName <em>your_website_wiki</em>.local

    DocumentRoot /var/lib/mediawiki/webroot/

    ErrorLog /var/log/apache2/<em>your_website_wiki</em>.local-error_log
    CustomLog /var/log/apache2/<em>your_website_wiki</em>.local-access_log combined

    # don't loose time with IP address lookups
    HostnameLookups Off

    # needed for named virtual hosts
    UseCanonicalName Off

    <Directory "/var/lib/mediawiki/webroot">
    AllowOverride None
    Options +ExecCGI -Includes
    Order allow,deny
    Allow from all
    </Directory>
</VirtualHost></pre>

First file ensures correct resolution for requests to _localhost_ and the second one is for our wiki resolution for external requests. After VirtualHosts are added, edit the _/etc/apache2/listen.conf_ file and uncomment next line

<pre>NameVirtualHost *:80</pre>

Now restart apache (_rcapache2 restart_).

Don't forget to add HTTP server to the Yast Firewall exceptions and

<pre>127.0.0.1 your_website_wiki.local</pre>

to your _/etc/hosts_ file.

Now you should be able to get you wiki from browser at guest OS from _http://your\_website\_wiki.local_ address.

**External networking**

What's left is to allow access from the host OS. To do so, go to Port forwarding in the VirtualBox network preferences (I assume you have NAT enabled) and add a rule to forward _x.x.x.x_ with 80 port to the 80 port of guest (where _x.x.x.x_ is an address of you computer in your network). You can also play with Bridget networking but it's not the case.

After you're done, go to Control panel in Windows, choose Firewall and add domain exceptions to [_BranchCache - Content Retrieval (Uses HTTP)_].

There're two abilities for other people to access your wiki. First, you can add a dns entry to the _C:WindowsSystem32Driversetchosts_ file manually. And the other one can be made by your system administrator which can add DNS record to the nearest internal DNS server for your computer.
