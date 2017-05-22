# Logstash hook for logrus <img src="http://i.imgur.com/hTeVwmJ.png" width="40" height="40" alt=":walrus:" class="emoji" title=":walrus:" /> [![Build Status](https://travis-ci.org/bshuster-repo/logrus-logstash-hook.svg?branch=master)](https://travis-ci.org/bshuster-repo/logrus-logstash-hook)
Use this hook to send the logs to [Logstash](https://www.elastic.co/products/logstash) over both UDP and TCP.

# Important notes

This a fork from [github.com/bshuster-repo/logrus-logstash-hook](https://github.com/bshuster-repo/logrus-logstash-hook.git) repo.

[ripcurld0](https://github.com/ripcurld0) going to rewrite original hook but there is no estimates when it will be ready for using in production. 
And more important is that he declines all pull requests with new features. 
So the main goal of this fork is to add some new features and use logstash hook until [ripcurld0](https://github.com/ripcurld0) finish his work.

Added features:

* [Async mode](#asyncmode). You can send log messages without blocking logic.

## Usage

```go
package main

import (
	"github.com/cheshir/logrustash"
    "github.com/sirupsen/logrus"
)

func main() {
        log := logrus.New()
        hook, err := logrustash.NewHook("tcp", "172.17.0.2:9999", "myappName")

        if err != nil {
                log.Fatal(err)
        }
        log.Hooks.Add(hook)
        ctx := log.WithFields(logrus.Fields{
          "method": "main",
        })
        ...
        ctx.Info("Hello World!")
}
```

This is how it will look like:

```ruby
{
    "@timestamp" => "2016-02-29T16:57:23.000Z",
      "@version" => "1",
         "level" => "info",
       "message" => "Hello World!",
        "method" => "main",
          "host" => "172.17.0.1",
          "port" => 45199,
          "type" => "myappName"
}
```


## Async mode

Create hook with _NewAsync..._ factory methods if you want to send logs in async mode.

Example:

```go
log := logrus.New()
hook, err := logrustash.NewAsyncHook("tcp", "172.17.0.2:9999", "myappName")
if err != nil {
        log.Fatal(err)
}
log.Hooks.Add(hook)
```


## Hook Fields
Fields can be added to the hook, which will always be in the log context.
This can be done when creating the hook:

```go

hook, err := logrustash.NewHookWithFields("tcp", "172.17.0.2:9999", "myappName", logrus.Fields{
        "hostname":    os.Hostname(),
        "serviceName": "myServiceName",
})
```

Or afterwards:

```go

hook.WithFields(logrus.Fields{
        "hostname":    os.Hostname(),
        "serviceName": "myServiceName",
})
```
This allows you to set up the hook so logging is available immediately, and add important fields as they become available.

Single fields can be added/updated using 'WithField':

```go

hook.WithField("status", "running")
```



## Field prefix

The hook allows you to send logging to logstash and also retain the default std output in text format.
However to keep this console output readable some fields might need to be omitted from the default non-hooked log output.
Each hook can be configured with a prefix used to identify fields which are only to be logged to the logstash connection.
For example if you don't want to see the hostname and serviceName on each log line in the console output you can add a prefix:

```go


hook, err := logrustash.NewHookWithFields("tcp", "172.17.0.2:9999", "myappName", logrus.Fields{
        "_hostname":    os.Hostname(),
        "_serviceName": "myServiceName",
})
...
hook.WithPrefix("_")
```

There are also constructors available which allow you to specify the prefix from the start.
The std-out will not have the '\_hostname' and '\_servicename' fields, and the logstash output will, but the prefix will be dropped from the name.


# Authors

Name         | Github    | Twitter    |
------------ | --------- | ---------- |
Boaz Shuster | ripcurld0 | @ripcurld0 |

# License

MIT.
