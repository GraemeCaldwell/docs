---
slug: how-to-install-ghost-ubuntu-20-04
author:
  name: Linode
  email: docs@linode.com
description: 'Learn how to install Ghost on Ubuntu 20.04 LTS with NGINX as a reverse proxy. We cover installing and configuring Ghost CMS, MySQL, Node, and NGINX.'
og_description: 'Learn how to install Ghost on Ubuntu 20.04 LTS with NGINX as a reverse proxy. We cover installing and configuring Ghost CMS, MySQL, Node, and NGINX.'
keywords: ['Ghost CMS','Ghost Ubuntu 20.04','install ghost on ubuntu']
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
published: 2021-11-18
title: "How to Install Ghost on Ubuntu 20.04 LTS"
h1_title: "How to Install Ghost CMS on Ubuntu 20.04"
enable_h1: true
external_resources:
---

## What is Ghost on Ubuntu?

Ghost is an open-source content management system (CMS) for publishing blogs and email newsletters. Based on Node.js and the JavaScript programming language, Ghost is a modern CMS with advanced features that include newsletter publication, built-in analytics, and social media integrations. 

In this guide, you will install Ghost on Ubuntu 20.04 LTS alongside its dependencies, including NGNIX, MySQL, Node.js, NPM, Ghost-CLI, and Let's Encrypt. 

## Before You Begin

1.  Familiarize yourself with our [Getting Started](/docs/getting-started/) guide and complete the steps for setting your Linode's hostname and timezone. 

2.  This guide assumes that you have a valid domain name and properly configured and propagated DNS records.

3.  Complete the sections of our [Securing Your Server](/docs/security/securing-your-server/) guide to create a standard user account, harden SSH access, and remove unnecessary network services.

{{< note >}}
This guide is written for a non-root user. Commands that require elevated privileges are prefixed with `sudo`. If you’re not familiar with the `sudo` command, see the [Users and Groups](/docs/tools-reference/linux-users-and-groups/) guide.
{{< /note >}}


4.  Update your system.

        sudo apt update && sudo apt upgrade

5.  Install build-essential.

        sudo apt install build-essential

6.  Create a new user for Ghost with elevated `sudo` privileges. Do not use the username `ghost` because it may cause conflicts with the `Ghost-CLI`. The examples in this guide use `ghostexample`.

        sudo adduser ghostexample
        sudo usermod -aG sudo ghostexample

    This guide assumes that you are logged-in to your server with the user you have just created for Ghost. 

## Install the NGINX Reverse Proxy for Ghost

1.  Install NGINX. 

        sudo apt install nginx

2.  If the `ufw` firewall is enabled, add rules that allow HTTP and HTTPS connections. 

        sudo ufw allow 'Nginx Full'

    For more information about NGINX on Ubuntu 20.04, visit [Installing and Using NGINX on Ubuntu 20.04](https://www.linode.com/docs/guides/how-to-install-and-use-nginx-on-ubuntu-20-04/).

## Install and Configure MySQL

1.  Download and install MySQL.

        sudo apt install mysql-server

2.  Create a root password for MySQL. Ghost-CLI uses this password to create Ghost's database and a `ghost` database user.  

        sudo mysql

3.  Enter the following SQL at the MySQL prompt. Replace `'password'` with a suitably secure root password.

        ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';

4.  Exit MySQL.

        quit 

5.  Re-login as the user you created earlier.

        su - ghostexample

## Install Node.js and NPM

Ghost requires a newer [version of Node.js](https://ghost.org/docs/faq/node-versions/) than is available in the Ubuntu 20.04 repositories. Use the NodeSource repositories to install Node 14. 

1.  Add the NodeSource repository for Node.js 14.

        sudo curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash

2.  Install Node 14.

        sudo apt install nodejs

## Install Ghost-CLI

Ghost-CLI is a command-line tool for installing and configuring Ghost. We'll install Ghost-CLI with `npm` and then use it to install Ghost CMS.

1.  Install Ghost-CLI.

        sudo npm install ghost-cli@latest -g

## Install Ghost on Ubuntu 20.04

1.  Create a directory to install Ghost with an appropriate name for your site. Ghost must be installed in an empty directory.

        sudo mkdir -p /var/www/ghost_directory

2.  Transfer ownership of the directory to the Ghost user you created earlier. 

        sudo chown ghostexample:ghostexample /var/www/ghost_directory

3.  Configure the directory's permissions.

        sudo chmod 775 /var/www/ghost_directory

4. Navigate to the new directory. 

        cd /var/www/ghost_directory

5.  Use Ghost-CLI to install Ghost. The command-line tool asks a series of questions about your site's URL, MySQL password, and other details. It will guide you through the configuration of NGINX and set up SSL with the Let's Encrypt certificate authority.  

        ghost install

    For more information about the `ghost install` process, [visit Ghost's documentation](https://ghost.org/docs/install/ubuntu/)

## Set Up Your Ubuntu Ghost Installation

Ghost's configuration interface is now available at `http://example.com/ghost`. Visit that URL to create an account and complete your Ghost blog's initial configuration. 