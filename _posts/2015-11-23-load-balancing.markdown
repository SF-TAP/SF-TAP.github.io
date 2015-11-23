---
layout: post
title:  "Load Balancing using Flow Abstraction Interface"
date:   2015-11-23 14:13:00 +0900
categories: tutorial
---

## Configuration for Load Balancing

The SF-TAP flow abstractor provides load balancing mechanism for
application-level analyzers by using the flow abstraction interface.
The flow abstraction interface can be divided into multiple interfaces
by specifying the 'balance' field in the configuration file as follows.

    http:
      up:      '^[-a-zA-Z]+ .+ HTTP/1\.(0\r?\n|1\r?\n([-a-zA-Z]+: .+\r?\n)+)'
      down:    '^HTTP/1\.[01] [1-9][0-9]{2} .+\r?\n'
      proto:   TCP
      if:      http
      format:  text
      body:    yes
      nice:    100
      balance: 4 # flows are balanced by 3 interfaces

In this case, 4 HTTP interfaces are provided for HTTP protocol as
follows.

    $ ls /tmp/sf-tap/tcp
    /tmp/sf-tap/tcp/http0=      /tmp/sf-tap/tcp/http2=
    /tmp/sf-tap/tcp/http1=      /tmp/sf-tap/tcp/http3=

## Effect of Multiple Processes

Thus, an HTTP analyzer can take advantage of multiple CPU cores
by connecting load balancing interfaces as follows.

    $ python3 http/sftap_http.py /tmp/sf-tap/tcp/http0=

    $ python3 http/sftap_http.py /tmp/sf-tap/tcp/http1=

    $ python3 http/sftap_http.py /tmp/sf-tap/tcp/http2=

    $ python3 http/sftap_http.py /tmp/sf-tap/tcp/http3=

Following graph shows CPU loads of the HTTP analyzer with 1, 2, and 4 processes
when generating 2500 HTTP requests per second.
The analyzer could not handle the requests with 1 single process,
but it could handle the requests with 2 or 4 processes.

![load-balancing load-balancing](/assets/load_balance.png)
