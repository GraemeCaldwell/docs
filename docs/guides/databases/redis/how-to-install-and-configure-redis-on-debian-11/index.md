---
slug: how-to-install-and-configure-redis-on-debian-11
author:
  name: Linode Community
  email: docs@linode.com
description: 'Learn how to install Redis on Debian 11 "bullseye". Redis is an open source database typically used as an in-memory data store and cache.'
keywords: ['redis server','install redis','install redis debian 11']
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
published: 2021-12-01
modified_by:
  name: Linode
title: 'How to Install and Configure Redis on Debian 11'
h1_title: 'How to Install and Configure Redis on Debian 11 "Bullseye"'
enable_h1: true
external_resources:
- '[Redis Quick Start](https://redis.io/topics/quickstart)'
- '[Access Control Lists](https://redis.io/topics/acl)'
- '[Redis Persistance](https://redis.io/topics/persistence)'
- '[Redis Setup Hints and Administration](https://redis.io/topics/admin)'
---

## What is Redis?

Redis is an open-source in-memory data store that is often used as a cache for web applications. Other uses include as a distributed database and a message broker. Redis is a key-value store. It stores values of several data types—including strings, lists, and sets—which are accessed via a key. By default, Redis stores data in memory and periodically writes it to permanent storage for data persistence.

## Redis Advantages and Disadvantages

Redis supports concurrency since all single-command operations in Redis are atomic. Redis contains a large [library of commands](https://redis.io/commands), but it does not have a query language or relational capabilities. Since Redis is an in-memory database, it runs quickly with low latency. However, you can only store as much data as memory allows. Redis provides persistence methods to either write the current database state to disk or record all transactions in a log.

Redis requires a robust, stable hosting environment to function properly. To store large amounts of data, we recommend hosting Redis on a [*High Memory Linode*](https://www.linode.com/products/high-memory/).

## Before You Begin

1.  Familiarize yourself with our [Getting Started](/docs/getting-started/) guide and complete the steps for setting your Linode's hostname and timezone.

2.  This guide will use `sudo` wherever possible. Complete the sections of our [Securing Your Server](/docs/security/securing-your-server) to create a standard user account, harden SSH access, remove unnecessary network services, and enable the `ufw` firewall.

3.  Update your system:

        sudo apt update && sudo apt upgrade

{{< note >}}
This guide is written for a non-root user. Commands that require elevated privileges are prefixed with `sudo`. If you’re not familiar with the `sudo` command, see the [Users and Groups](/docs/tools-reference/linux-users-and-groups/) guide.
{{< /note >}}

## Installing Redis

Redis 6.0 is available in the default Debian 11 repositories. However, at the time of writing the stable release is Redis 6.2.6. Instead of installing from the default repositories, you can get an up-to-date version from the Redis Labs PPA.

1.  Install the `gpg` package, which is needed to add the repository's GPG signature.

        sudo apt install gpg

2.  Download and install the Redis PPA's OpenPGP key:

        curl https://packages.redis.io/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/redis-keyring.gpg > /dev/null

3.  Add the repository to Debian's package sources:

        echo "deb [arch=amd64 signed-by=/usr/share/keyrings/redis-keyring.gpg] https://packages.redis.io/deb bullseye main" | sudo tee /etc/apt/sources.list.d/redis.list

4.  Download the source files:

        sudo apt update

5.  Install Redis with `apt`:

        sudo apt install redis

    The `redis` package installs `redis-server` and `redis-tools`.

### Install Redis from Source

If you would rather not use the PPA method, you can download and install Redis without the package manager.

1.  Install the `build-essential` package, which contains tools needed to build software:

        sudo apt install build-essential

2.  Download the most recent stable version, which you can find on the [Redis download page](https://redis.io/download):

        curl https://download.redis.io/releases/redis-6.2.6.tar.gz --output redis-6.2.6.tar.gz

3.  Extract the tarball and navigate into the source directory:

        tar xzf redis-6.2.6.tar.gz
        cd redis-6.2.6

4.  Build the software. This make take some time, depending on your Linode's CPU resources:

        make

5.  Install `redis-server`, `redis-cli`, and related tools:

        sudo make install

6.  Verify that the expected version of Redis is installed:

        redis-server -v

    {{< output >}}
Redis server v=6.2.6 sha=00000000:0 malloc=jemalloc-5.1.0 bits=64 build=63022d674f8c4bb2
{{< /output >}}

## Enable and Run Redis on Debian 11

If you installed Redis from the PPA, it should already be configured for the `systemd` system control utility, so you can skip Step 1 below. If you installed from source, you may need to edit `/etc/redis/redis.conf`—Redis's main configuration file—to control Redis via the `systemctl` command.

1.  Open `/etc/redis/redis.conf` in a text editor:

        sudo vim /etc/redis/redis.conf

    Search for the line that begins "supervised" and edit the value to read "systemd".

    {{< file "/etc/redis/redis.conf" redis >}}
supervised systemd
{{< /file >}}

2.  Use `systemctl` to start the Redis Server service:

        sudo systemctl restart redis-server

3.  The `redis-cli` tool allows you to control and interact with the Redis server. Enter the interactive Redis command CLI.

        redis-cli

4.  Ping the server to verify the connection:

        PING

    If Redis is running, you should receive a reply:

    {{< output >}}
PONG
{{< /output >}}

5.  To further test the Redis server, you can use the `SET` command to store a key-value pairing:

        SET server:name "Linode"

    {{< output >}}
OK
{{< /output >}}

6.  Retrieve the stored value with the `GET` command:

        GET server:name

    {{< output >}}
"Linode"
{{< /output >}}

## Secure Redis on Debian 11

Recent versions of Redis maintain multi-user security with *Access Control Lists* (ACL). *ACLs* allow administrators to limit the keys a connection can access and and the commands it can execute. When a client connects, it authenticates with a username and password. That user's *ACL* configuration then determines the connection's access rights.

In the default configuration, any connection can access every key and call every command. There are two methods to improve security. The first is to set a password for the default user. This method is suitable for a single-user instance. The more secure alternative for multi-user instances is to configure an Access Control List with multiple users with restricted permissions.

### Configure a Password for Redis

This method creates a password for the default user. Every connection is required to authenticate with the same password. On a multi-user system, we recommend using *ACLs* instead.

1.  Edit the `requirepass` directive in `/etc/redis/redis.conf`.

        sudo vim /etc/redis/redis.conf

    Search for `requirepass`. Uncomment the existing line, and change the default password ("foobared") to a secure password of your choice.

    {{< file "/etc/redis/redis.conf" redis >}}
requirepass yoursecurepassword
{{< /file >}}

2.  Restart Redis:

        sudo systemctl restart redis-server

3.  Verify that the password change is active. Open `redis-cli`:

        redis-cli

    Attempt to set a key without authenticating.

        SET server:name "Linode Again"

    Redis refuses to execute the command and returns an authentication error.

    {{< output >}}
(error) NOAUTH Authentication required.
{{< /output >}}

4.  Log in using the `AUTH` command and the password you chose. Redis returns `OK` if you authenticate successfully.

        AUTH "yoursecurepassword"

5.  Repeat the `SET` command. Because you are authenticated, Redis should execute the command and return an `OK` response.

        SET server:name "Linode Again"

### Configure an Access Control List (ACL) for Redis

An *Access Control List* allows administrators to control who can access data and execute commands. We recommend using *ACLs* in multi-user environments because they allow for more precise data access and command execution permissions.

You can add users in in the `redis.conf` file. User directives consist of a username, an `on` or `off` keyword, a set of command permissions, a set of access permissions, and a password. If you are following this guide, you should already have one user configured: the default user. You can see currently configured users by running the `ACL LIST` command in `redis-cli`.

        ACL LIST

{{< output >}}

1)  "user default on #0b955ad2dbd415df6a2b11780d129e1ac5e3094573da76c313e2fade359af381 ~*&* +@all"
{{< /output >}}

When you added a password for all connections in the previous section, it was the default user you configured. The `AUTH "password"` command is equivalent to `AUTH default "password"`. The default user is currently able to use all commands (+@all), all keys (~\*), and all pub/sub channels(&\*).

In this section, you will add two users. The first has access to all keys and commands. The second is similar, but with a restriction that disallows use of the `SET` command. We cannot provide a complete explanation of *ACLs* in this guide; to learn more, visit the [Redis ACL page](https://redis.io/topics/acl).

1.  Edit `redis.conf`:

        sudo vim /etc/redis/redis.conf

    You can add `user` directives anywhere in the file, but the main `redis.conf` includes a *Security* section with documentation and `user` directive examples.

    {{< file "/etc/redis/redis.conf" redis >}}
user user2 +@all allkeys on >user2pass
user user3 +@all -SET allkeys on >user3pass
{{< /file >}}

    Replace `user2pass` and `user3pass` with long, secure passwords.

2.  Restart the Redis server:

        sudo systemctl restart redis-server

3.  Re-open `redis-cli` and authenticate with the `AUTH` command using user2's credentials.

        redis-cli
        AUTH user2 user2pass
        SET server:name "A String Value"

    Because *user2* has permission to set values, Redis returns `OK`.

4.  Exit and re-open `redis-cli` once again. This time, authenticate with user3:

        redis-cli
        AUTH user3 user3pass
        SET server:name "A String Value"

    Because `user3` does not have permission to use the `SET` command, Redis returns a permission error.

5.  However, this user can retrieve the value set by user2 because they have permission to execute the `GET` command.

        GET server:name

{{< note >}}
You can also create users through the `redis-cli` interface using the `ACL SET USER` command, or through a separate ACL file. See the [*Redis ACL page*](https://redis.io/topics/acl) for more information.

In earlier versions of Redis, administrators could rename and therefore hide powerful commands using `rename-command` directives in `redis.conf`. With the introduction of the ACL feature, this directive is no longer recommended. However, it is still available for backward compatibility.
{{< /note >}}

## Configure Redis Persistence

Redis stores all of its data in memory, so in the event of a crash or a system reboot everything is lost. If you want to permanently save your data, you must configure some form of data persistence. Redis supports two persistence options:

-  *Redis Database File* (RDB) persistence takes snapshots of the database at intervals corresponding to the `save` directives in the `redis.conf` file. The `redis.conf` file contains three default intervals. RDB persistence generates a compact file for data recovery. However, any writes since the last snapshot are lost.

-  *Append Only File* (AOF) persistence appends every write operation to a log. Redis replays these transactions at startup to restore the database state. You can configure AOF persistence in the `redis.conf` file with the `appendonly` and `appendfsync` directives. This method is more durable and results in less data loss. Redis frequently rewrites the file so it is more concise, but AOF persistence results in larger files, and it is typically slower than the RDB approach.

1.  To change the RDB snapshot intervals, edit the `save` directives in `redis.conf`. A directive consisting of `save 30 100` means Redis continues to take a snapshot every 30 seconds provided at least 100 keys have changed. Multiple snapshot thresholds can be configured.

    {{< file "/etc/redis/redis.conf" >}}
save 900 1
save 300 10
save 60 10000
{{< /file >}}

1.  To enable AOF persistence, edit `redis.conf` and change the value of the `appendonly` directive to `yes`. Then you can set the `appendfsync` directive to any one of the following of your choice.

    -  `always` - sync upon every new command
    -  `everysec` - sync one time per second
    -  `no` - let Ubuntu manage the sync.

   The default of `everysec` is a good compromise for most implementations.
    {{< file "/etc/redis/redis.conf" >}}
appendonly yes
appendfsync everysec
{{< /file >}}

1.  Restart the redis server after making any changes to the Redis persistence directives.

        sudo systemctl restart redis-server

{{< note >}}

Some of the AOF persistence settings are complicated. Consult the [*Redis Persistence Documentation*](https://redis.io/topics/persistence) for more advice about this option.
{{< /note >}}

## Optimize Redis for Better Performance

Redis recommends several additional optimizations for the best performance. In addition to the following advice, Redis makes several recommendations regarding persistence and replication. Consult the [*Redis Administration Information*](https://redis.io/topics/admin) for more information.

1.  Set the overcommit memory setting to `1` in `sysctl.conf`. You must reboot the node for this setting to take effect.
    {{< file "/etc/sysctl.conf" >}}
vm.overcommit_memory = 1
{{< /file >}}
    {{< note >}}
Enter the command `sysctl vm.overcommit_memory=1` to apply this setting immediately.
{{< /note >}}
1.  Disable the transparent huge pages feature as this adversely affects Redis latency.

        echo never > /sys/kernel/mm/transparent_hugepage/enabled
1.  Specify an explicit maximum memory value (in bytes) in `redis.conf`. This value must be at least somewhat less than your available system memory. Restart Redis after making this change.
    {{< file "/etc/redis/redis.conf" >}}
maxmemory 2147483648
{{< /file >}}
1.  Create some swap space in the system to prevent Redis from crashing if it consumes too much memory. The following commands set up a 2GB swap file.

        sudo mkdir /swapdir/
        sudo dd if=/dev/zero of=/swapdir/swapfile bs=1MB count=2048
        sudo chmod 600 /swapdir/swapfile
        sudo mkswap /swapdir/swapfile
        sudo swapon /swapdir/swapfile
