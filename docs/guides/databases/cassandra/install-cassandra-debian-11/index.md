---
slug: install-cassandra-debian-11
author:
  name: Linode Community
  email: docs@linode.com
description: 'Learn how to install the NoSQL distributed database Apache Cassandra on Debian 11—an in-depth guide to Apache Cassandra installation and configuration.'
keywords: ['install cassandra debian','cassandra installation','apache cassandra','cassandra database']
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
published: 2021-11-19
modified_by:
  name: Linode
title: "How to Install Apache Cassandra on Debian 11"
h1_title: "Cassandra Installation: How to Install Apache Cassandra on Debian 11"
enable_h1: true
external_resources:
- '[Installing Cassandra](http://cassandra.apache.org/doc/latest/cassandra/getting_started/installing.html)'
- '[Apache Cassandra Documentation](https://cassandra.apache.org/doc/latest/)'
- '[The Cassandra Query Language](https://cassandra.apache.org/doc/latest/cassandra/cql/)'
- '[Cassandra Security Configuration](http://cassandra.apache.org/doc/latest/cassandra/operating/security.html)'
---

## What is Apache Cassandra?

[Apache Cassandra](https://cassandra.apache.org/_/index.html) is an open-source NoSQL database management system. Cassandra excels as a distributed database, allowing users to store large amounts of data across many servers and data centers. Cassandra provides low-latency replication to multiple nodes, and is often used to create high-availability database clusters with no single point of failure.

Apache Cassandra is a Java-based application, and in this guide you will learn how to install and configure Java, Apache Cassandra, and their dependencies on Debian 11 'bullseye'.

## Before You Begin

{{< note >}}
This guide is written for a non-root user. Commands that require elevated privileges are prefixed with `sudo`. If you’re not familiar with the `sudo` command, see the [Users and Groups](/docs/tools-reference/linux-users-and-groups/) guide.
{{< /note >}}

1.  Familiarize yourself with our [Getting Started](/docs/getting-started/) guide and complete the steps for setting your Linode's hostname and timezone.

2.  Complete the sections of our [Securing Your Server](/docs/security/securing-your-server/) guide to create a standard user account, harden SSH access and remove unnecessary network services.

3.  Update your system:

        sudo apt update && sudo apt upgrade

## Install Cassandra Dependencies

1.  Install package dependencies:

        sudo apt install apt-transport-https ca-certificates wget dirmngr gnupg software-properties-common

2.  Install Java:

        sudo apt install default-jdk

3.  Verify the correct version of Java is now installed:

        java --version

    {{< output >}}
    openjdk version "11.0.13" 2021-10-19
OpenJDK Runtime Environment (build 11.0.13+8-post-Debian-1deb11u1)
OpenJDK 64-Bit Server VM (build 11.0.13+8-post-Debian-1deb11u1, mixed mode, sharing)
    {{< /output >}}

## Install Apache Cassandra on Debian 11

1.  Download the Apache Cassandra repository GPG keys:

        curl https://downloads.apache.org/cassandra/KEYS | sudo apt-key add -

2.  Add the Apache Cassandra APT repository to the Debian sources list:

        echo "deb https://downloads.apache.org/cassandra/debian 40x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list

3.  Download the package index and install Cassandra:

        sudo apt update
        sudo apt install cassandra

## Activate Cassandra

1.  Start Cassandra and configure systemd to start it on boot:

        sudo systemctl enable cassandra
        sudo systemctl start cassandra
        sudo systemctl -l status cassandra

## Verify the Cassandra Cluster is Working as Expected

1.  Verify the Cassandra cluster's status with `nodetool`, a [CLI cluster management tool](https://cassandra.apache.org/doc/latest/cassandra/tools/nodetool/nodetool.html) for Cassandra which was installed alongside the main application:

        nodetool status

    The output should include `UN` in the first column, indicating that the cluster is working.

    {{< output >}}
    Datacenter: datacenter1
    =======================
    Status=Up/Down
    |/ State=Normal/Leaving/Joining/Moving
    --  Address    Load       Tokens  Owns (effective)  Host ID                               Rack
    UN  127.0.0.1  69.05 KiB  16      100.0%            c7d364f9-b074-4ccf-98aa-ff7a96d2771b  rack1
    {{< /output >}}

2.  If the output contains connection errors, add your server's loopback address or public IP address to the `/etc/cassandra/conf/cassandra-env.sh` file. You can [find the public IP address](https://www.linode.com/docs/guides/find-your-linodes-ip-address/) for your server in the Cloud Manager.

        sudo vim /etc/cassandra/cassandra-env.sh

3.  Search for the line that includes `-Djava.rmi.server.hostname=`. Watch our [VIM beginner's guide](https://www.linode.com/content/vim-the-beginners-guide-to-the-perfect-text-editor/) to learn how to search, edit text, and save files in Vim.

4.  Replace `<public name>` with your loopback address or public IP address and uncomment the line (remove the `#` symbol at the beginning of the line).

    {{< file "Debian /etc/cassandra/conf/cassandra-env.sh" bash >}}
# add this if you're having trouble connecting:
JVM_OPTS="$JVM_OPTS -Djava.rmi.server.hostname=<public name>"
    {{< /file >}}

5.  Restart Cassandra:

        sudo systemctl restart cassandra

6.  Re-check the node's status:

        nodetool status

## Configure Cassandra on Debian 11

### Enable Security Features

The default Apache Cassandra configuration is not secure. We will edit configuration variables to create a more secure installation. The configuration variables suggested below are sensible defaults for a development environment, but you should configure Cassandra to meet the needs of your project.

1.  Make a copy of the Cassandra configuration file. If you make a mistake, you can revert to the defaults by deleting the edited file and renaming the backup to `cassandra.yaml`.

        sudo cp /etc/cassandra/cassandra.yaml /etc/cassandra/cassandra.yaml.backup

2.  Open `cassandra.yaml` in your preferred text editor. 

        sudo vim /etc/cassandra/cassandra.yaml

3.  Find and edit the configuration variables so they match those in the following listing. Uncomment the variables if they are commented out.

    {{< file "Debian /etc/cassandra/cassandra.yaml" yaml >}}
. . .

authenticator: org.apache.cassandra.auth.PasswordAuthenticator
authorizer: org.apache.cassandra.auth.CassandraAuthorizer
role_manager: CassandraRoleManager
roles_validity_in_ms: 0
permissions_validity_in_ms: 0

. . .
    {{< /file >}}

4. Save the edited file and restart Cassandra:

        sudo systemctl restart cassandra

### Add An Administration Superuser

Cassandra ships with a default user and password, both `cassandra`. To improve Cassandra's security, we'll remove the default user's superuser permissions. Then we'll create a new superuser account with a secure password.

1.  Open the Cassandra command terminal and log in as the default user:

        cqlsh -u cassandra -p cassandra

2. Create the superuser. Replace the brackets ([superuser]) and their contents with the applicable information.

        CREATE ROLE [new_superuser] WITH PASSWORD = '[secure_password]' AND SUPERUSER = true AND LOGIN = true;

    For example:

        CREATE ROLE exampleuser WITH PASSWORD = 'examplepassword' AND SUPERUSER = true AND LOGIN = true;

3.  Log out of `cqlsh`:

        exit

4.  Log back in with the newly created superuser account. Replace the username and password with the credentials from the previous step.

        cqlsh -u exampleuser -p examplepassword

5.  Remove the elevated permissions from the `cassandra` user:

        ALTER ROLE cassandra WITH PASSWORD = 'cassandra' AND SUPERUSER = false AND LOGIN = false;
        REVOKE ALL PERMISSIONS ON ALL KEYSPACES FROM cassandra;

6.  Grant all permissions to the superuser account. As before, replace `[superuser]` with the new superuser's username.

        GRANT ALL PERMISSIONS ON ALL KEYSPACES TO [superuser];

7.  Log out of `cqlsh`:

        exit

### Edit the Console Configuration File

The `cqlshrc` file contains configuration settings that influence the Cassandra shell's behavior.

{{< note >}}
Ensure you complete the steps in this section using your limited user account. This account will need [sudo privileges](/docs/security/securing-your-server/#debian), if it does not already have them. Do not complete this section as the root user.

{{</ note >}}

1.  Create the `cqlshrc` file and the `~/.cassandra` directory if it doesn't already exist:

        sudo mkdir ~/.cassandra
        sudo vim ~/.cassandra/cqlshrc

2.  View the [sample configuration file](https://github.com/apache/cassandra/blob/trunk/conf/cqlshrc.sample) and copy variables you wish to use into your `cqlshrc`. Remove any comment markers (;) at the beginning of each variable.

    For example, your `cqlshrc` might include the following:

    {{< file "~/.cassandra/cqlshrc" aconf >}}
[ui]
;; The encoding used for characters
color = on
;; The encoding used for characters
encoding = utf8

[connection]
;; The host to connect to
hostname = 127.0.0.1
;; A timeout in seconds for opening new connections
timeout = 10
;; A timeout in seconds for executing queries
request_timeout = 10
    {{< /file >}}

3.  Save the file and change the directory and file permissions. This prevents other users from accessing the file and sensitive configuration variables.

        sudo chmod 440 ~/.cassandra/cqlshrc
        sudo chmod 700 ~/.cassandra

## Rename The Cluster

Cassandra gives clusters the default name "Test Cluster". You can display the cluster's name with `nodetool`:

    nodetool describecluster

{{< output >}}
Cluster Information:
    Name: Test Cluster
    Snitch: org.apache.cassandra.locator.SimpleSnitch
    DynamicEndPointSnitch: enabled
    Partitioner: org.apache.cassandra.dht.Murmur3Partitioner
    Schema versions:
        2207c2a9-f598-3971-986b-2926e09e239d: [127.0.0.1]
. . .
{{< /output >}}

In this section, you will change the cluster name to an appropriate name for your project.

1.  Log in to the `cqlsh` control terminal as the superuser you created earlier:

        cqlsh -u superuser

2.  Replace `[new_name]` with your preferred cluster name:

        UPDATE system.local SET cluster_name = '[new_name]' WHERE KEY = 'local';

3.  Exit `cqlsh`:

        exit

4. Search for and change the `cluster_name` variable in `cassandra.yaml` to the new name:

        sudo vim /etc/cassandra/cassandra.yaml

    {{< file "Debian /etc/cassandra/cassandra.yaml" yaml >}}
# The name of the cluster. This is mainly used to prevent machines in
# one logical cluster from joining another.
cluster_name: 'Example Cluster Name'
    {{< /file >}}

5.  Save and close the file.

6.  Flush the Cassandra system cache:

        nodetool flush system

7.  Restart Cassandra:

        sudo systemctl restart cassandra

8. Wait until Cassandra has restarted, and then verify the name change:

        nodetool describecluster

    {{< output >}}
Cluster Information:
    Name: Example Cluster Name
    Snitch: org.apache.cassandra.locator.SimpleSnitch
    DynamicEndPointSnitch: enabled
    Partitioner: org.apache.cassandra.dht.Murmur3Partitioner
    Schema versions:
        2207c2a9-f598-3971-986b-2926e09e239d: [127.0.0.1]

    {{< /output >}}

## Where to Go From Here

You have configured and secured an Apache Cassandra cluster on your Debian 11 Linode. To fully utilize Cassandra's distributed database capability, you will add more nodes. Adding nodes is called bootstrapping. The process is described in detail in [Adding nodes to an existing cluster](https://docs.datastax.com/en/archived/cassandra/3.0/cassandra/operations/opsAddNodeToCluster.html).