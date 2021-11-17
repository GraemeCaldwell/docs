---
slug: find-large-files-linux
author:
  name: Linode
  email: docs@linode.com
description: 'A guide on how to find large or big files in Ubuntu, Debian, Fedora, CentOS Linux systems or most other Unix-like systems.'
keywords: ["linux find large files", "find largest files linux", "find big files linux"]
tags: ["linux"]
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
published: 2021-11-16
title: 'How to Find Large Files in Linux'
h1_title: 'How to Find Large Files in Linux'
enable_h1: true
external_resources:
 - '[du man page](https://man7.org/linux/man-pages/man1/du.1.html)'
 - '[find man page](https://man7.org/linux/man-pages/man1/find.1.html)'
---

In this guide, we explain how to find large files from the Linux command line. Finding large files is a common server administration task. It is often helpful for identifying large log files, media files, and other data consuming excessive disk space.

## Before You Begin

1.  Familiarize yourself with our [Getting Started](/docs/getting-started/) guide and complete the steps for setting your Linode's hostname and timezone.

2.  This guide uses `sudo` wherever possible. Complete the sections of our [Securing Your Server](/docs/security/securing-your-server/) to create a standard user account, harden SSH access and remove unnecessary network services.

## How to Find Large Files in Linux with `du`

`du` is a command-line tool for estimating disk space usage. 

1.  Use `du` to display disk space usage within a directory. Without additional options, `du` recursively displays byte counts for directories beneath the indicated directory.

    sudo du /var/log/

2.  To view sizes for files as well as directories, add the `-a` option

    sudo du -a /var/log/

3.  Display human-readable byte sizes. The `-h` option tells `du` to output human-readable file sizes in kilobytes, megabytes, and gigabytes as appropriate.

    sudo du -ah /var/log/

5.  Sort biggest files from largest to smallest. To identify the largest files, pipe the output of `du` through `sort`, which is available on all Linux distributions with GNU tools installed.

    sudo du -ah /var/log/ | sort -rh


6.  Get the ten biggest files by piping the output of `sort` through `head`.

    sudo du -ah /var/log/ | sort -rh | head -10

## How to Find Large Files in Linux with `find`

The GNU `find` command can also be used to discover large files. It is particularly useful when you're interested in large files but not directories. For example, to find all files larger than 500 MiB in the current directory and its subdirectories.

    sudo find . -type f -size +500M

Replace "." with the directory you are interested in. The `-type f` option restricts `find` to reporting on files. The `-size` option indicates the file size with greater than (+), less than (-), or exact files sizes. Available file size units include blocks (b), bytes (c), kibibytes (k), mebibytes (M), and gibibytes (G).

To prevent `find` from recursing to other filesystems, add the `-xdev` option.

    sudo find . -xdev -type f -size +500M

To display file size data in a more useful format, combine `find` with other tools such as `xargs`, `du`, and `sort`.

    sudo find . -xdev -type f -size +100M -print | xargs du -sh | sort -rh | head -5

This uses `find` to identify files larger than 100 MB in and beneath the current directory. Its output is piped to `xargs`, which runs `du` on each result. The output from `du` is then piped through `sort` and `head` to display the five biggest files sorted in descending order.
