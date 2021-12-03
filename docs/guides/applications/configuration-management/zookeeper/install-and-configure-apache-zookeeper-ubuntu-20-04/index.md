---
slug: install-and-configure-apache-zookeeper-ubuntu-20-04
author:
  name: Linode Community
  email: docs@linode.com
description: 'Learn how to install and configure Apache ZooKeeper on Ubuntu 20.04. Apache ZooKeeper is an open-source distributed configuration coordination service.'
keywords: ['apache zookeeper','install apache zookeeper','apache zookeeper ubuntu']
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
published: 2021-12-03
modified_by:
  name: Linode
title: "How to Install and Configure Apache ZooKeeper on Ubuntu 20.04"
h1_title: "How to Install and Configure Apache ZooKeeper on Ubuntu 20.04"
enable_h1: true
external_resources:
- '[ZooKeeper Overview](https://zookeeper.apache.org/doc/r3.7.0/zookeeperOver.html)'
- '[ZooKeeper Getting Started Guide](https://zookeeper.apache.org/doc/r3.7.0/zookeeperStarted.html)'
---
## What is Apache ZooKeeper?

## Before You Begin

1.  Familiarize yourself with our [Getting Started](/docs/getting-started/) guide and complete the steps for setting your Linode's hostname and timezone.

2.  This guide will use `sudo` wherever possible. Complete the sections of our [Securing Your Server](/docs/security/securing-your-server/) to create a standard user account, harden SSH access, and remove unnecessary network services.

3.  If you configured the `ufw`  firewall in the previous step, add rules to open the ports ZooKeeper uses:

        sudo ufw allow 2181
        sudo ufw allow 2888
        sudo ufw allow 3888

4.  Update your system:

        sudo apt update && sudo apt upgrade

{{< note >}}
This guide is written for a non-root user. Commands that require elevated privileges are prefixed with `sudo`. If you’re not familiar with the `sudo` command, see the [Users and Groups](/docs/tools-reference/linux-users-and-groups/) guide.
{{< /note >}}

## Install Java on Ubuntu 20.04

Apache ZooKeeper is a Java application, so the first step is to install a supported version of the JDK. The Ubuntu 20.04 repositories include OpenJDK 11, which ZooKeeper supports.

1.  Install Java:

        sudo apt install default-jdk

2.  Verify the correct version of Java is now installed:

        java --version

    {{< output >}}
    openjdk 11.0.11 2021-04-20
OpenJDK Runtime Environment (build 11.0.11+9-Ubuntu-0ubuntu2.20.04)
OpenJDK 64-Bit Server VM (build 11.0.11+9-Ubuntu-0ubuntu2.20.04, mixed mode, sharing)
    {{< /output >}}

## Create a User for ZooKeeper

Creating a dedicated user to run the ZooKeeper service improves security by limiting access to ZooKeeper files and the network service.

1.  Create a new user. In our examples, we call the user `zookeeper`, but you can use any appropriate username.

        sudo adduser zookeeper

    Choose a long, secure password when prompted. You can answer `adduser`'s other questions or leave them blank if you prefer. Once `adduser` has gathered the information it needs, it creates the user and their home directory.

    {{< output >}}
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for zookeeper
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] Y
{{< /output >}}

1.  Add `zookeeper` to the `sudo` group so it can execute commands that require root privileges. :

        sudo adduser zookeeper sudo

## Create ZooKeeper's Data Directory

ZooKeeper achieves low latencies by storing data in memory. However, it also stores data and transaction logs to disk so information persists across reboots.

1.  Create a directory for ZooKeeper to store state and data:

        sudo mkdir /var/zookeeper

2.  Transfer the directory's ownership to the `zookeeper` user:

        sudo chown zookeeper:zookeeper /var/zookeeper

## Download and Install Apache ZooKeeper

At the time of writing, the current stable version is ZooKeeper 3.6.3. The Ubuntu 20.04 repositories contain an older version (3.4). Instead of installing from the repositories, you can download and install the current version's binary files from the Apache file servers.

In this section, you will create a directory for the binary files and install Apache ZooKeeper. The [Linux Filesystem Hierarchy](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html) provides the `/opt/` directory for "add-on application software packages," so that's the ideal location for the ZooKeeper binaries.

1.  Navigate to the `opt` directory and create the `zookeeper` directory:

        cd /opt
        sudo mkdir zookeeper

2.  Find the current stable version from the [ZooKeeper downloads page](https://downloads.apache.org/zookeeper/stable/).

3.  Download the binary tarball. This will have a filename with the format `apache-zookeeper-<STABLE_VERSION>-bin.tar.gz`. Replace the version number in this command with the current stable version.

        sudo wget https://downloads.apache.org/zookeeper/stable/apache-zookeeper-3.6.3-bin.tar.gz

4.  Download the associated SHA512 digest file. This will have a filename with the format `apache-zookeeper-<STABLE_VERSION>-bin.tar.gz.sha512`. As before, replace the version number in this command with the current stable version.

        sudo wget https://downloads.apache.org/zookeeper/stable/apache-zookeeper-3.6.3-bin.tar.gz.sha512

5.  To verify that the ZooKeeper files were downloaded correctly, compare the SHA512 digest you downloaded with a digest generated from the tarball:

        sudo sha512sum -c apache-zookeeper-3.6.3-bin.tar.gz.sha512

    If the digest is valid, the output is as follows:

    {{< output >}}
apache-zookeeper-3.6.3-bin.tar.gz: OK
{{< /output >}}

    If the message digest generated from the downloaded file does not match, the ZooKeeper files may have have been corrupted or modified.

    {{< output >}}
apache-zookeeper-3.6.3-bin.tar.gz: FAILED
sha512sum: WARNING: 1 computed checksum did NOT match
{{< /output >}}

7.  Extract the binaries from the tarball and then delete it:

        sudo tar -xvf apache-zookeeper-3.6.3-bin.tar.gz
        sudo rm apache-zookeeper-3.6.3-bin.tar.gz

8.  Give the `zookeeper` user ownership of the files so it can execute them:

        sudo chown zookeeper:zookeeper -R apache-zookeeper-3.6.3-bin

9.  Create a symbolic link to the ZooKeeper directory and assign ownership to the `zookeeper` user. This will allow you to use a shorter directory name and allow the directory to remain consistent when you update ZooKeeper.

        sudo ln -s apache-zookeeper-3.6.3-bin zookeeper
        sudo chown -h zookeeper:zookeeper zookeeper

## Configure ZooKeeper on Ubuntu 20.04

Next, you will configure ZooKeeper as a standalone installation and tell it where to store persistence files. The examples use the `vim` text editor, but you can use `nano` or an editor of your choice.

1.  Create and edit a `zoo.cfg` file in `/opt/zookeeper/conf/`:

        sudo vim /opt/zookeeper/conf/zoo.cfg

2.  Add the following configuration directives and save the file:

    {{< file "/opt/zookeeper/conf/zoo.cfg" >}}
tickTime=2000
dataDir=/var/zookeeper
clientPort=2181
{{< /file >}}

    These are the recommended settings for a minimal configuration. The `tickTime` directive sets the length of time in milliseconds between heartbeats, a measure ZooKeeper uses for timing its operations. You can learn more about the ZooKeeper configuration file in the [ZooKeeper Administrator's Guide](https://zookeeper.apache.org/doc/r3.7.0/zookeeperAdmin.html#sc_requiredSoftware).

## Start ZooKeeper and Test the Installation

1.  Change to the ZooKeeper user:

        su zookeeper

2.  Execute the `zkServer.sh` command to start ZooKeeper:

        /opt/zookeeper/bin/zkServer.sh start
    {{< output >}}
/usr/bin/java
ZooKeeper JMX enabled by default
Using config: /opt/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
{{< /output >}}

3.  Connect to the ZooKeeper server:

        /opt/zookeeper/bin/zkCli.sh -server 127.0.0.1:2181

    When you successfully connect, ZooKeeper outputs status information and gives you a prompt with the label `CONNECTED`.

    To see a list of possible commands, type `help` at the prompt. To exit the prompt, type `quit`. When you have finished testing the installation, stop ZooKeeper:

        /opt/zookeeper/bin/zkServer.sh stop

ZooKeeper is now configured and running in standalone mode. Standalone mode is suitable for testing and development, but, in production ZooKeeper is typically used as a multi-node cluster. The remainder of this guide will detail how to configure ZooKeeper to work with Ubuntu's `systemd` service manager and how to create a multi-node cluster with three Linodes.

## Configure ZooKeeper to Work With Systemd

The systemd init system and the `systemctl` command are used to manage system processes. With `systemctl`, you can start, stop, and check the status of services. While it's possible to manage ZooKeeper with its `zkCli.sh` script, `systemctl` is more convenient. It also works with the init system so ZooKeeper is automatically started when a node reboots.

To manage ZooKeeper with `systemctl,` you need to create a `.service` file.

1.  Create a file called `zookeeper.service` in `/etc/systemd/system/`.

        sudo vim /etc/systemd/system/zookeeper.service

2.  Add the following to the file and save it:

    {{< file "/etc/systemd/system/zookeeper.service" >}}
[Unit]
Description=ZooKeeper
Documentation=<http://zookeeper.apache.org>
Requires=network.target
After=network.target

[Service]
Type=forking
WorkingDirectory=/opt/zookeeper
User=zookeeper
Group=zookeeper
ExecStart=/opt/zookeeper/bin/zkServer.sh start /opt/zookeeper/conf/zoo.cfg
ExecStop=/opt/zookeeper/bin/zkServer.sh stop /opt/zookeeper/conf/zoo.cfg
ExecReload=/opt/zookeeper/bin/zkServer.sh restart /opt/zookeeper/conf/zoo.cfg
TimeoutSec=30
Restart=on-failure

[Install]
WantedBy=default.target
{{< /file >}}

    The file specifies the user ZooKeeper should be started by and links `systemctl` commands to the equivalent `zkServer.sh` commands. It also tells `systemd` when to start ZooKeeper in the boot process. 

3.  Start ZooKeeper with `systemctl`:

        sudo systemctl start zookeeper

4.  Check the status of the ZooKeeper service:

        sudo systemctl status zookeeper

    {{< output >}}
● zookeeper.service - ZooKeeper
     Loaded: loaded (/etc/systemd/system/zookeeper.service; disabled; vendor preset: enabled)
     Active: active (running) since Fri 2021-12-03 17:52:09 UTC; 49s ago
       Docs: <http://zookeeper.apache.org>
    Process: 6832 ExecStart=/opt/zookeeper/bin/zkServer.sh start /opt/zookeeper/conf/zoo.cfg (code=exited, status=0/SUCCESS)
   Main PID: 6857 (java)
      Tasks: 30 (limit: 1071)
     Memory: 43.1M
     CGroup: /system.slice/zookeeper.service
             └─6857 java -Dzookeeper.log.dir=/opt/zookeeper/bin/../logs -Dzookeeper.log.file=zookeeper-zookeeper-server-localhost.log>
{{< /output >}}

## Configure a Multi-Node ZooKeeper Cluster

In production, ZooKeeper is typically deployed as a quorum distributed on several servers. A quorum must have an odd number of nodes because ZooKeeper uses a majority-based consensus system; a majority of nodes must agree on a transaction before it is accepted.

In this section, we outline the basic procedure for setting up a three-node cluster—the smallest cluster that can achieve a majority.

1.  Deploy two additional Linodes and follow the set-up process outlined in the previous sections on each.

2.  Ensure that ZooKeeper is stopped on all nodes.

        sudo systemctl stop zookeeper

3.  To configure a multi-node cluster, the nodes need to know where to find the other members of the quorum. The details are configured in the `zoo.cfg` file on each node. Edit all three nodes' configuration files so they contain the following—all nodes should have identical `zoo.cfg` files.

    Replace `node_1`, `node_2`, and `node_3` with the IP address or fully qualified domain name of your Linodes.

    {{< file "/etc/systemd/system/zookeeper.service" >}}
tickTime=2000
dataDir=/var/zookeeper
clientPort=2181
initLimit=10
syncLimit=5
server.1=node_1:2888:3888
server.2=node_2:2888:3888
server.3=node_3:2888:3888
{{< /file >}}

    As you can see, the first three lines are identical to the standalone configuration. To create a multi-node cluster, we add `initLimit` to specify how long the initial synchronization can take and `syncLimit` to specify how much time can expire between requests and their acknowledgment. Finally, we add the connection details for each node in the cluster.

3.  In `zoo.cfg`,  we defined three nodes called `server.1`, `server.2`, etc. To complete the cluster configuration, we must also tell each node which of these represents itself. This is achieved by creating a `myid` file in each node's data directory.

        sudo vim /var/zookeeper/myid

    {{< file "/var/zookeeper/myid" >}}
1
{{< /file >}}

    On the first server, the file contains "1". On the second, it contains "2", and so on.

4.  When all the files are edited, start ZooKeeper on each node. Log in over SSH and change to the `zookeeper` user:

        su -c zookeeper

5.  Navigate to the ZooKeeper directory on each node and start the node with `systemctl` or the following command:

        cd /opt/zookeeper
        java -cp zookeeper.jar:lib/*:conf org.apache.zookeeper.server.quorum.QuorumPeerMain conf/zoo.cfg

Your ZooKeeper nodes are now configured to run as part of a multi-node cluster. To learn more about multi-node configuration and security, visit the [ZooKeeper Administrator's Guide](https://zookeeper.apache.org/doc/r3.7.0/zookeeperAdmin.html#sc_zkMulitServerSetup).
