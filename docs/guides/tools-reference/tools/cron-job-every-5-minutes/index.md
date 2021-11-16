---
slug: cron-job-every-5-minutes
author:
  name: Linode
  email: docs@linode.com
description: 'A guide on how to set up a crontab to run a cron job every five minutes and test it, including links to other cron articles in the KB.'
keywords: ["cron", "crontab", "cron job every five minutes", "test cron job"]
tags: ["linux","cron"]
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
published: 2021-11-16
title: 'How to Run a cron Job Every 5 Minutes'
h1_title: "How to Run a cron Job Every 5 Minutes"
enable_h1: true
external_resources:
 - '[Cron Man Page](https://man7.org/linux/man-pages/man8/cron.8.html)'
---

## What is Cron?

Cron is a command-line job scheduler for Linux and other Unix-like operating systems. A cron job runs a command or script on a schedule. For example, a user might schedule a cron job to download data from an API every five minutes. They create a script with the commands to download the data and use cron to execute that script five-minute intervals.

## Before You Begin

1. Familiarize yourself with our [Getting Started](/docs/getting-started/) guide and complete the steps for setting your Linode's hostname and timezone.

2. This guide uses `sudo` wherever possible. Complete the sections of our [Securing Your Server](/docs/security/securing-your-server/) to create a standard user account, harden SSH access and remove unnecessary network services.

## Crontab Basics

Cron jobs are scheduled in the crontab (cron table) file, which is edited with the `crontab` command. The crontab file consists of multiple lines, each with six fields separated by white space. The first five fields are date and time fields, and the sixth field is the command to be run.
The date and time fields tell cron when to run the command. From left to right, they indicate:

* Minute of the hour (0-59)
* Hour of the day (0-23)
* Day of month (1-31 or mon, tue, etc)
* Month of the year (1-12 or jan, feb, etc)
* Day of the week (0-6 or sun, mon, tue, etc)

For example,

    10 5 * * * COMMAND

This crontab entry indicates the 10th minute of the 5th hour. Fields marked with an asterisk (\*) tell cron to run the job at every relevant interval. In this case, every day of every month at ten minutes past 5 a.m.

The crontab syntax also has an optional field before the final field to indicate the user the command is executed as.

  10 5 * * * USERNAME COMMAND

The crontab file syntax is flexible. In addition to a number or day/month name, it also allows ranges (1-10), lists (1,2,3,5), and combinations of both. Additionally, to run a cron job every five minutes, we need the step value syntax.

Step values are used with ranges to tell cron to skip a certain number of intervals. For example, `0-12/2` in the "hour of the day" field indicates every two hours between midnight and midday. Put another way, it tells cron to run the command at 00:00, skip two hours, run the command at 02:00, and so on.

You can learn more about cron and the crontab syntax in [Schedule Tasks with Cron](https://www.linode.com/docs/guides/schedule-tasks-with-cron/)

## How to Schedule a Cron Job for Every Five Minutes

1. To schedule a cron job to run every five minutes, use the `crontab` command to add a new line to the crontab file. `crontab` opens or creates a crontab file for the current user. Ensure that your user has the necessary permissions to run the command or script you would like to schedule.

Open crontab for editing in the text editor specified in your environment's VISUAL or EDITOR environmental variable.

    crontab -e

2. Add the following to the crontab file on a new line:

    */5 * * * * COMMAND

Replace COMMAND with the command or script you would like to run. For example, if you would like to download data from a URL every five minutes, the crontab line might read:

    */5 * * * * curl http://example.com

3. Save the file and close the editor. The `crontab` tool automatically installs and activates the new crontab file.

This basic structure can easily be adapted to other time intervals.

To run a cron job every 30 minutes:

    */30 * * * * COMMAND


## How to Test the Cron Job

Cron does not alert you if a cron job fails to run. If a cron job is important to your application or business, you may want to test the job and the command.

### Verify Cron is Running

Cron jobs do not run if the cron service is deactivated.

To verify the cron service is active on Debian and Ubuntu:

    service cron status

On CentOS:

    service crond status

### Testing the Command

Testing the command is straightforward. Run the command or script in an interactive shell and verify that it performs as expected. For example, if the command is intended to download data and store it in a specific location, run it manually to ensure that it does so.

### Testing the Cron Job

To test the cron job, you can verify that cron is running the command by observing the results immediately after the scheduled time. If the expected results do not occur---the data doesn't appear in the designated directory, for example---there may be a problem with the cron job's schedule, user permissions, or with cron itself.

To verify the cron job is running on time, add a logging command to the script or to the crontab that appends a line to a log file when the cron job runs.

    date >> /home/USERNAME/cronjob.log

This appends the date and time to a log file whenever it runs.

For more sophisticated logging, capture a command or script's standard output and standard error and redirect them to a log file.

    */5 * * * * COMMAND > /home/USERNAME/cronjob.log 2>&1
