# PlanB: a distributed HTTP and websocket proxy

[![Build Status](https://travis-ci.org/tsuru/planb.svg?branch=master)](https://travis-ci.org/tsuru/planb)

## What Is It?

PlanB is a HTTP and websocket proxy backed by Redis and inspired by
[Hipache](https://github.com/dotcloud/hipache).

It aims to be fully compatible with Hipache when Redis is used as a backend.
The same format is used for all keys stored in Redis so migrating from Hipache
to PlanB should be completely seamless. The process should be as simple as
replacing Hipache's executable for PlanB.

## Start-up flags

The following flags are available for configuring PlanB on start-up:

- ``--listen/-l``: the address to which PlanB will bind. Default value is
  ``0.0.0.0:8989``.
- ``--read-redis-host``: Redis host of the server which contains application
  addresses. Default value is ``localhost``.
- ``--read-redis-port``: Redis port of the server which contains application
  addresses. Default value is ``6379``.
- ``--write-redis-host``: Redis host of the server which PlanB will use for
  publishing dead backends. Default value is ``localhost``.
- ``--write-redis-port``: Redis port of the server which which PlanB will use
  for publishing dead backends. Default value is ``6379``.
- ``--access-log``: File path where access log will be written. If value equals
  ``syslog`` log will be sent to local syslog. Default value is
  ``./access.log``.
- ``--request-timeout``: Total backend request timeout in seconds. Default
  value is ``30``.
- ``--dial-timeout``: Dial backend request timeout in seconds. Default value is
  ``10``.
- ``--dead-backend-time``: Time in seconds a backend will remain disabled after
  a network failure. Default value is ``30``.
- ``--flush-interval``: Time in milliseconds to flush the proxied request.
  Default value is ``10``.

## Features

* Load-Balancing
* Dead Backend Detection
* Dynamic Configuration
* WebSocket

## VHOST Configuration

The configuration is managed by **Redis** that makes possible
to update the configuration dynamically and gracefully while
the server is running, and have that state shared across workers
and even across instances.

Let's take an example to proxify requests to 2 backends for the hostname
`www.tsuru.io`. The 2 backends IP are `192.168.0.42` and `192.168.0.43` and
they serve the HTTP traffic on the port `80`.

`redis-cli` is the standard client tool to talk to Redis from the terminal.

Follow these steps:

### Create the frontend:

```
$ redis-cli rpush frontend:www.tsuru.io mywebsite
(integer) 1
```

The frontend identifer is `mywebsite`, it could be anything.

### Add the 2 backends:

```
$ redis-cli rpush frontend:www.tsuru.io http://192.168.0.42:80
(integer) 2
$ redis-cli rpush frontend:www.tsuru.io http://192.168.0.43:80
(integer) 3
```

### Review the configuration:

```
$ redis-cli lrange frontend:www.tsuru.io 0 -1
1) "mywebsite"
2) "http://192.168.0.42:80"
3) "http://192.168.0.43:80"
```

While the server is running, any of these steps can be
re-run without messing up with the traffic.

## Links

* Repository & Issue Tracker: https://github.com/tsuru/planb
* Talk to us on Gitter: https://gitter.im/tsuru/tsuru
