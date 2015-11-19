---
layout: post
title: "Log Rotation for Rails"
date: 2015-11-19T11:04:40-10:00
author: Stephane Liu
categories:
- sysadmin
- maintenance
---

Leveraing system utilities on a unix operating system simplifies log rotation for Rails applications. There are only a few configurations necessary to get this up and running.

The first step is to create a file in /etc/logrotate.d directory. Giving it the name of your application is a good idea. Then, open the file in vim or nano and add the following.

```bash
/home/deploy/APPNAME/shared/log/*.log {
  daily
  missingok
  rotate 365
  compress
  delaycompress
  notifempty
  copytruncate
  dateext
}
```
Options:

* **daily** - Rotate the log files each day. Alternatively, weekly or monthly could be used
* **missingok** - Ignore if the log file does not exist
* **rotate 365** - Only keep 365 days of logs around
* **compress** - GZip the log file on rotation
* **delaycompress** - Rotate the file one day, then compress it the next day. This minimize interference with Rails server.
* **notifempty** - Don't rotate the file if the logs are empty
* **copytruncate** - Copy the log file then empty it. This insures that the log file Rails is writing to always exists because the file being modified does not change. If this is not done, you will need to add a post action to restart the Rails server.
* **dateext** - Append the date at the end of the file name instead of an the default incrementing number. (production.log-20151118). If you are testing log rotation, you will need to disable this option in order to generate multiple logrotation files on the same day.

`logrotate` runs daily via cron by default. View `/etc/cron.daily/logrotate` for more information.
