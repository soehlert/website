---
title: "Centos7 Owncloud"
date: 2016-02-29T22:56:53-06:00
draft: true
tags: ["owncloud", "centos"]
categories: ["how-to"]
type: "post"
toc: true
body_classes: "blog"
---

## Background
I have been thinking about ownCloud for a while as a means to keep a few files available to me on a few different hosts. I could use something like dropbox, but some of these files are semi private (think operationally private, not ss# or anything). Plus it gives me an excuse to mess around with a new thing. Originally, I wanted to create a container, but I ended up backing off of that when I realized that an LXC container mounting an NFS share on Proxmox isn't exactly reliable. I don't want to have my actual data stored in the container, so I'm going with a VM instead. These instructions should work on any sort of Centos 7 install; Proxmox VM, ESXi VM, DigitalOcean, Linode, hardware, etc.

The install instructions are pretty short, but we'll also take a look at a little performance boost as well as hardening our install in case we want to open it up to the world. Even if we don't, it's not that hard (as you'll see), to get a more secure installation, so why not?

## Installation
Alright here we go:

Install Prerequisites
Grab repo file
Install
Give access to owncloud directory to run as apache user
Enable and start Mariadb
Enable and start httpd
One hardening step real quick...secure the mariadb installation
Log into mariadb
Create the database
Allow the clouddb user access to the database we just created on localhost only (no need to allow remote login here). Make sure to change the password in '' to something better. That's the clouddb user's password.

```bash
yum -y install wget mariadb-server php-mysql php-pecl-apc
wget http://download.opensuse.org/repositories/isv:ownCloud:community/CentOS_7/isv:ownCloud:community.repo -O /etc/yum.repos.d/owncloud.repo
yum install owncloud
chown -R apache.apache /var/www/html/owncloud/
systemctl enable mariadb; systemctl start mariadb
systemctl enable httpd; systemctl start httpd
mysql_secure_installation
mysql -u root -p
create database clouddb;
grant all on clouddb.* to 'clouddbuser'@'localhost' identified by 'password';
```

Quit mariadb prompt and then go to http://your-ip-address/owncloud to see that it worked!

Note: you may notice a warning on your admin page that looks like this:

```
This server has no working Internet connection. This means that some of the features like mounting external storage, notifications about updates or installation of third-party apps will not work. Accessing files remotely and sending of notification emails might not work, either. We suggest to enable Internet connection for this server if you want to have all features.
```

This appears to be due to this bug in the nss library that curl on Centos 7 is using.

### PHP Memcache
Another warning you may have could look like this:

```
No memory cache has been configured. To enhance your performance please configure a memcache if available. Further information can be found in our documentation.
```

Configuring memcache will speed up retrieval of frequently requested files, improving performance in some cases. I mostly did this just because the warnings bothered me...

Since Centos 7 is still on PHP 5.4 we can use APC for our local cache as described here.

Restart your webserver (systemctl restart httpd) and then addd this line to the config file at /var/www/html/owncloud/config/config.php

```
'memcache.local' => '\OC\Memcache\APC',
```

Now, upon refreshing your admin page, you should be all set.

## Hardening
Now that we have everything installed we are...not done yet. We are going to need to do some securing as things are not in a good state as is.

### Data directory
According to owncloud's documentation, your data directory should not be located in your web root. Strangely enough, it defaults to being put there during installation, though you can adjust that with the advanced settings during installation. If you did not take care of that before, we can still fix that now.

First we need to create the new directory. You can choose to put this wherever you'd like (though outside of /var/www/, otherwise what's the point?). I chose to put my directory at /media/ocdata since I'll have more space there as well. Make sure this is owned by the apache user.

```bash
mkdir /media/ocdata chown apache.apache /media/ocdata
```

Now we need to tell owncloud where to look for its data directory. Edit the config file at /var/www/html/owncloud/config/config.php and make sure you have a line like this (the line should exist but should previously say /var/www/html/owncloud/data):

```
'datadirectory' => '/media/ocdata',
```

Now restart apache and all should be fine.

### Apache Hardening
Now let's go ahead and secure our webserver. Currently, we allow both HTTP and HTTPS logins and browsing, which we should take care of. Never send credentials in clear text.

### SSL
First we need to get an SSL certificate to use. You can use whatever you want; Verisign, StartSSL, Let's Encrypt, GoDaddy or self signed are some options. We're going to look at self signed cert, since there's nobody else that's going to use my ownCloud server but me.

By default, openssl uses sha1 hashing, which is probably vulnerable, so we're going to use sha256 for some further security.

```bash
openssl req -new -sha256 -x509 -nodes -days 365 -out your.website.net.pem -keyout your.website.net.key
```

Now that we have our key and cert, we have to point apache to them. I created a directory at /etc/http/ssl to put them in. In /etc/httpd/conf.d/ssl.conf there's a block that starts with <VirtualHost _default_:443>. Inside of that block we need to make sure we have two lines (they should be there, we just need to edit the paths they point to):

```
SSLCertificateFile /etc/httpd/ssl/owncloud.crt SSLCertificateKeyFile /etc/httpd/ssl/owncloud.key
```

### Ciphers
We want to make sure to use secure ciphers since it would defeat the purpose to leave ourselves vulnerable here. Starting with Qualys' recommended ciphers, we can then tighten things up (the recommendations are a few years old after all). The necessary lines we end up with are as follows. Granted, I'm not a crypto expert, so this might even allow for more tightening, please let me know if so!

```
SSLEngine on SSLProtocol all -SSLv2 -SSLv3 -TLSv1 SSLCipherSuite HIGH:MEDIUM:!aNULL:!MD5:!SEED:!IDEA SSLCipherSuite EECDH+AESGCM:ECDHE-RSA-AES256-SHA
```

One thing to note with this is that if you have an Android phone 4.4 or older, you will need to allow TLSv1.

### Forcing HTTPS
First, make sure the VirtualHost in /etc/httpd/conf.d/ssl.conf is set to correct document root and server name. We should now be able to test HTTPS. Now that that works, we can make sure everyone has to use HTTPS to login and browse. It's good courtesy to add a redirect on port 80 so anyone trying to go there automatically gets redirected to our encrypted HTTPS port. In /etc/httpd/conf.d/ssl.conf we need to set a stanza above the VirtualHost 443 section we messed with earlier. It should look like this (changing the server name and redirect to fit your needs):

```
javascript <VirtualHost *:80> ServerName cloud.owncloud.com Redirect permanent / https://cloud.owncloud.com/ </VirtualHost>
```

The next section is shamelessly ripped straight from the ownCloud hardening guide located [here](https://doc.owncloud.com/server/admin_manual/configuration/server/harden_server.html)

While redirecting all traffic to HTTPS is good, it may not completely prevent man-in-the-middle attacks. Thus administrators are encouraged to set the HTTP Strict Transport Security header, which instructs browsers to not allow any connection to the ownCloud instance using HTTP, and it attempts to prevent site visitors from bypassing invalid certificate warnings.

This is achieved by editing the /etc/httpd/conf.d/ssl.conf file again to make your VirtualHost 443 stanza look like this (snipped for brevity):

```
javascript <VirtualHost *:443> ServerName owncloud.example.com <IfModule mod_headers.c> Header always set Strict-Transport-Security "max-age=15768000; includeSubDomains; preload" </IfModule>
```

### Encryption
Owncloud includes server side encryption by default. This encrypts the files at rest both on your server as well as remote storage that is connected to your server. Keep in mind this only encrypts the contents of the files and not the filenames and directory structure. To turn this on, simply go to the admin page of your installation, and set a recovery key. MAKE SURE TO KEEP THIS IN A SAFE PLACE. You do not want to lose this.

### Firewall
Make sure firewalld is installed and then we can open up HTTP and HTTPS (this assumes your zone is "public"):

```bash
firewall-cmd --zone=public --permanent --add-service=http firewall-cmd --zone=public --permanent --add-service=https
```

### Fail2ban
Now we can set up fail2ban to help mitigate any brute force attacks at your login page.

```bash
yum install fail2ban
```

Taken from fail2ban's site,

Fail2ban scans log files (e.g. /var/log/apache/error_log) and bans IPs that show the malicious signs -- too many password failures, seeking for exploits, etc. Generally Fail2Ban is then used to update firewall rules to reject the IP addresses for a specified amount of time, although any arbitrary other action (e.g. sending an email) could also be configured.

### ownCloud
First we have to configure owncloud to keep track of failed log ins. Make sure you have these lines in your /var/www/html/owncloud/config/config.php file and that the timezone matches your timezone on your server.

```
'logtimezone' => 'UTC', 'logfile' => '/var/log/owncloud.log', 'loglevel' => '2', 'log_authfailip' => true,
```

Now make sure apache user is the owner of the logfile you set.

```bash
chown apache.apache /var/log/owncloud.log
```

Now we must configure fail2ban to read that owncloud log. First step is to create a configuration file at /etc/fail2ban/filter.d/owncloud.conf that should look like this:

```
[Definition] failregex={"reqId":".*","remoteAddr":"<HOST>","app":"core","message":"Login failed: .*","level":2,"time":".*"}
```

Now that we explained to fail2ban an entry for a failed login in the owncloud log looks like, we have to create a service definition to tie it all together. /etc/fail2ban/jail.local

```
[owncloud] enabled = true filter = owncloud port = https logpath = /var/log/owncloud.log
```

## Wrapup
So now we should have a working installation of ownCloud that is actually pretty secure. Again, I'm not a crypto expert, so please let me know if you think I should change the ciphers. Also, let me know what you think in the comments.
