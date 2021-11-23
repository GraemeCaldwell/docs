---
slug: configure-webdav-apache-ubuntu-20-04
author:
  name: Linode Community
  email: docs@linode.com
description: 'Learn how to configure Apache to share files with WebDAV on Ubuntu 20.04. We explain how to set up Apache and secure WebDAV with digest authentication'
keywords: ['WebDAV server','WebDAV server ubuntu','Setup WebDAV server ubuntu']
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
published: 2021-11-23
modified_by:
  name: Linode
title: "How to Configure WebDAV for Apache on Ubuntu 20.04"
h1_title: "How to Configure Apache for WebDAV File Sharing on Ubuntu 20.04"
enable_h1: true
external_resources:
- '[Apache mod_dav documentation](http://www.example.com)'
- '[Apache Authentication and Authorization](https://httpd.apache.org/docs/2.4/howto/auth.html)'
---

## What is WebDAV?

WebDAV is an extension to HTTP that allows users to manage files on a server. Using a WebDAV connection, users can upload, create, delete, and move files stored in a designated directory on the server. Because WebDAV is an HTTP extension, it is natively supported by many web servers, including Apache.

This guide explains how to configure Apache to enable WebDAV for secure file sharing on Ubuntu 20.04.

## Before You Begin

1.  Familiarize yourself with our [Getting Started](/docs/getting-started/) guide and complete the steps for setting your Linode's hostname and timezone.

2.  Complete the sections of our [Securing Your Server](/docs/security/securing-your-server/) to create a standard user account, harden SSH access and remove unnecessary network services.

3.  This guide assumes you have Apache installed and running on your Ubuntu 20.04 Linode. If necessary, complete the Apache installation and configuration sections in [How to Install a LAMP Stack on Ubuntu 20.04](/docs/guides/how-to-install-a-lamp-stack-on-ubuntu-20-04/), including the steps to configure Apache virtual hosts.

4.  This guide also assumes a domain name's [DNS records are configured](/docs/guides/dns-manager/) to resolve to your Linode server's public IP.

6.  To secure connections to your WebDAV server, follow the steps in [Securing Web Traffic Using Certbot with Apache on Ubuntu 20.04](/docs/guides/enabling-https-using-certbot-with-apache-on-ubuntu/) to install a free SSL certificate.

3.  Update your system:

        sudo apt update && sudo apt upgrade

{{< note >}}
This guide is written for a non-root user. Commands that require elevated privileges are prefixed with `sudo`. If you’re not familiar with the `sudo` command, see the [Users and Groups](/docs/tools-reference/linux-users-and-groups/) guide.
{{< /note >}}

## Enable WebDAV on Apache

Apache natively supports WebDAV storage via a pair of Apache modules: `dav` and `dav_fs`.

1.  Enable the WebDAV modules:

        sudo a2enmod dav
        sudo a2enmod dav_fs

2.  Restart Apache to load the modules:

        sudo systemctl restart apache2

## Set Up The Filesystem for WebDAV

1.  Create a folder to contain the files served over WebDAV. Replace `example.com` with your domain name.

        sudo mkdir /var/www/html/example.com/webdav

2.  Change the directory's owner to `www-data` so Apache can serve files located here:

        sudo chown www-data:www-data /var/www/html/example.com/webdav/

3.  Create a directory to store Apache's WebDAV database files:

        sudo mkdir -p /usr/local/apache/var/

4.  Change the database directory owner to `www-data`:

        sudo chown www-data:www-data /usr/local/apache/var

## Configure Apache To Serve Files with WebDAV

In this section, you will edit the virtual hosts file to enable WebDAV. The following steps assume you have configured virtual hosts for your domains.

{{< note >}}
If you are using Apache's default host with an SSL certificate, replace the VirtualHost filename with `default-ssl.conf`. Use `000-default.conf` if you haven't configured an SSL certificate for your domain.
{{< /note >}}


The examples use the `vim` text editor, but you can substitute `nano` or your preferred editor.

1.  Open the VirtualHost file. If the virtual host has an SSL certificate:

        sudo vim /etc/apache2/sites-available/example.com-le-ssl.conf

    If the virtual host does not have an SSL certificate:

        sudo vim /etc/apache2/sites-available/example.com.conf

    In the next steps, you will add directives to this file. When you're done, it will look like the following file listing. Your file may also contain comments, but we've removed them for clarity.

    {{< file "/etc/apache2/sites-available/example.com-le-ssl.conf" aconf >}}
DavLockDB /usr/local/apache/var/DavLock
<IfModule mod_ssl.c>
<VirtualHost *:443>

    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/example.com/public_html
    ServerName example.com
    ServerAlias www.example.com
    ServerAdmin webmaster@localhost

    Alias /webdav /var/www/html/example.com/webdav

    <Directory /var/www/webdav>
        DAV On
    </Directory>

    ErrorLog /var/www/html/example.com/logs/error.log
    CustomLog /var/www/html/example.com/logs/access.log combined

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    Include /etc/letsencrypt/options-ssl-apache.conf
    SSLCertificateFile /etc/letsencrypt/live/example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem
</VirtualHost>
</IfModule>
    {{< /file >}}

2.  Add a `DavLockDB` directive at the top of the file:

    {{< file "/etc/apache2/sites-available/example.com-le-ssl.conf" aconf >}}
DavLockDB /usr/local/apache/var/DavLock
    {{< /file >}}

3.  Add an `Alias` directive inside the `<VirtualHost>` block:

    {{< file "/etc/apache2/sites-available/example.com-le-ssl.conf" aconf >}}
Alias /webdav /var/www/html/example.com/webdav
    {{< /file >}}


4.  Add a `Directory` block inside the `<VirtualHost>` block:

    {{< file "/etc/apache2/sites-available/example.com-le-ssl.conf" aconf >}}
<Directory /var/www/html/example.com/webdav>
    DAV On
</Directory>
    {{< /file >}}

5.  Save the file and restart Apache:

        sudo systemctl restart apache2

## Adding WebDAV Authentication

WebDAV is now up-and-running, but the default configuration is insecure. Anyone can log in and view or delete files on your server. To improve security, configure Apache to use digest authentication. Digest authentication stores usernames and hashed passwords in a file. Only users listed in the file can log in.

1.  Create a `webdav-users` file in the database directory you created in previous steps:

        sudo touch /usr/local/apache/var/webdav-users

2.  Change the file's owner to `www-data`:

        sudo chown www-data:www-data /usr/local/apache/var/webdav-users

3.  Use `htdigest` to add users and hashed passwords to this file.

        sudo htdigest /usr/local/apache/var/webdav-users webdav example-user

    Replace `example-user` with your preferred username. The `webdav` in this command after the filename is the realm to which the user is added. We'll use it in subsequent steps to configure authentication. You can replace it with a name you prefer, but be aware that users can see it when they log in.

4.  `htdigest` requests a password and then adds the authentication credentials to the `webdav-users` file.

    {{< output >}}
Adding user example-user in realm webdav
New password:
Re-type new password:
    {{< /output >}}

5.  Edit the virtual host file to tell Apache to use digest authentication.

        sudo vim /etc/apache2/sites-available/example.com-le-ssl.conf

    Edit the `<Directory>` block you added earlier so that it includes the following directives. These directives will also tell Apache not to display a directory listing to unauthorized users who open `https://example.com/webdav` in a browser.

    {{< file "/etc/apache2/sites-available/example.com-le-ssl.conf" aconf >}}
<Directory /var/www/html/example.com/webdav>
    DAV On
    AuthType Digest
    AuthName "webdav"
    AuthUserFile /usr/local/apache/var/webdav-users
    Require valid-user
</Directory>
{{< /file >}}

6.  Enable Apache's `auth_digest` module and restart Apache.

        sudo a2emod auth_digest
        sudo systemctl restart apache2

7.  Test the authentication by attempting to log in with (1) the user you added to `webdav-users` and (2) a different user. The authentication attempt should succeed for the authorized user and fail for any other user.