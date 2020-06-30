---
layout: post
title:  "Logging into Syslog"
tags:
  - DevOps
  - Go
  - Linux
aside: true
---

Logging is so simple that it is very hard to implement it right. In a project I started to
clear up logging and I noticed that there are many places one can introduce syslog as a
logging target, and there are a lot of things can get just too messy. I want to show
how I started with syslog, how I separated the different layers of logging.

To go on I suppose you have `rsyslog` already installed on your Linux.

### What is syslog?

[Syslog](https://www.rsyslog.com/doc/v8-stable/index.html) is simple and fast logging
system built in the Posix operating systems. You can think that it is a logging hub,
there are several inputs it can consume, it can emit logs to multiple outputs and
also it has a rich rule engine.

When an application logs it can choose which syslog facilities wants to send log messages.
There are some builtin facilities like `KERN`, `USER`, `DAEMON`, `CRON` and there are
custom facilities for local applications from `LOCAL0` to `LOCAL7` (at least).

### Let the application log

So at first our application should connect to those facilities.

```go
package main

import (
	"flag"
	stdlog "log"
	"log/syslog"
)

func main() {
	logger, err := syslog.NewLogger(syslog.LOG_LOCAL0|syslog.LOG_INFO, stdlog.Lshortfile)
	if err != nil {
		panic(err)
	}

	audit, _ = syslog.NewLogger(syslog.LOG_LOCAL1|syslog.LOG_INFO, stdlog.Lshortfile)
}

```

That is pretty simple we made a generic logger which writes to the first local facility
at info level, so we can refer to this action as `local0.info` later. The second log
we will write rarely, it will be an audit log syslog.

It is better to use a logging framework, I am using the `gommon/log` framework, pretty
straightforward to set up. So let us connect the `logger` to the log framework.

```go
package main

import (
	"flag"
	stdlog "log"
	"log/syslog"

	"github.com/labstack/gommon/log"
)

func main() {
	logger, err := syslog.NewLogger(syslog.LOG_LOCAL0|syslog.LOG_INFO, stdlog.Lshortfile)
	if err != nil {
		panic(err)
	}

    log.DisableColor()
    log.SetOutput(logger.Writer())
    log.SetHeader("${time_rfc3339_nano} ${level} ${short_file}:${line} ${prefix}")

	audit, _ = syslog.NewLogger(syslog.LOG_LOCAL1|syslog.LOG_INFO, stdlog.Lshortfile)

    log.Info("This is an info log message")
    log.Warn("This is a warning")
}
```

Let us run this app, what happens? In the `/var/log/messages` you will see two lines

```
Jun 30 08:49:31 myhost /home/richard/logtest[53676]: \
  2020-06-30T08:49:31.701738949Z INFO main.go:27 - This is an info log message
Jun 30 08:49:31 myhost /home/richard/logtest[53676]: \
  2020-06-30T08:49:31.701852547Z WARN main.go:29 - This is a warning
```

Why?

### rsyslog.conf

In default the rsyslog writes every info level message to that file. If you check the
`/etc/rsyslog.conf` you can see this line.

```
*.info;mail.none;authpriv.none;cron.none    /var/log/messages
```

It means that `*.info` so `local0.info` also written to the messages log. You can see in
the code that we also added the `LOG_INFO` when we made the syslog loggers.

It would be nice to separate those logs from the system messages, right? So at first we
need to redirect them to a different file, and also we need to remove from the messages
log. The first is a piece of cake, just add a `/etc/rsyslog.d/my-app.conf` with this
content.

```conf
$FileCreateMode 0644
local0.*        /var/log/my-app.log
$FileCreateMode 0600
```

At first we set the file create mode in order that everybody can read the log file.
Let us suppose that not the root user will run our app, so it would be nice if that
user can read the application logs. The setting is global, so from now on this file
mask will be effective. That is why we set it back to `0600`.

Let us restart the `rsyslog.service`.

Well, we are half-ready, we need to remove the `local0.*` from messages. We need to
change the messages rule to this

```conf
*.info;mail.none;authpriv.none;cron.none;local0.none    /var/log/messages
```

So from the `local0` log facility nothing will be written to the messages.

### Formatting log messages

We are almost done, I just don't want to see two log message timestamps in the logs. One
is put by the Go logging framework, and the other is done by syslog. I trust syslog, so
in the application we can remove the timestamp generating for the log messages.

```go
    log.SetHeader("${level} ${short_file}:${line} ${prefix}")
```

That is nice but the default syslog timestamp is pretty vague. We need to explain syslog
to use a more precise time format. The RFC3339 is with microsecond precision, most of
the cases is good enough.

```conf
$FileCreateMode 0644

$template offerFormat,"%TIMESTAMP:::date-rfc3339% %HOSTNAME% %msg%\\n"
local0.*		/var/log/my-app.log;offerFormat

$FileCreateMode 0600
```

Log lines should be like this

```
2020-06-30T10:54:31.693715+00:00 myhost WARN main.go:29 - This is a warning
```

You can setup the audit log similarly, to go all the audit logs to a different file.
If you don't need the

So that is it, I needed to collect the different information fragements from different
sources, I hope it will help you to setup some clear logging of your applications.
