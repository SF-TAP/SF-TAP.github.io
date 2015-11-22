---
layout: post
title:  "Write Your Own Analyzers"
date:   2015-11-21 18:39:00 +0900
categories: tutorial
---

# How to Write Application Level Analyzers

In this document, we show how to implement analyzers.

This tutorial is also available on SlideShare.

<iframe src="//www.slideshare.net/slideshow/embed_code/key/jjAp6Xd5H3LNh" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/ytakano/tutorialf-of-sftap-flow-abstractor" title="Tutorial of SF-TAP Flow Abstractor" target="_blank">Tutorial of SF-TAP Flow Abstractor</a> </strong> from <strong><a href="//www.slideshare.net/ytakano" target="_blank">Yuuki Takano</a></strong> </div>

## Flow Abstraction Interfaces

The SF-TAP flowabstractor provdes flow abstraction interfaces by using UNIX domain socket.
Thus, in order to capture flows (a.k.a. TCP streams),
you must access to the interfaces.
The directory structure of the interfaces is as follows.

    $ cd /tmp/sf-tap
    $ ls -R
    loopback7=   tcp/         udp/

    ./tcp:
    default=         http=            smtp=            torrent_tracker=
    dns=             http_proxy=      ssh=             websocket=
    ftp=             irc=             ssl=

    ./udp:
    default=     dns=         torrent_dht=

## Configuration File

Before explaining how to implement analyzers,
we show a snippet of the configuration.
For example, a configuration of HTTP is given as follows.

    1 http:
    2   up:     '^[-a-zA-Z]+ .+ HTTP/1\.(0\r?\n|1\r?\n([-a-zA-Z]+: .+\r?\n)+)'
    3   down:   '^HTTP/1\.[01] [1-9][0-9]{2} .+\r?\n'
    4   proto:  TCP  # TCP or UDP
    5   if:     http # file name of UNIX domain socket
    6   format: text # format of header. text or binary
    7   body:   yes  # if specified 'no', only header is output
    8   nice:   100  # the smaller a value is, the higher a priority is

Here, regular expressions for HTTP protocol are defined by 'up' and 'down' on
line 2 and 3.
Note that there are 2 regular expressions here because TCP is connection oriented protocol.
Whereas, UDP protocols have only 1 regular expression as follows.

    1 torrent_dht: # BitTorrent DHT
    2   up:     '^d.*1:y1:[eqr].*e$'
    3   proto:  UDP
    4   if:     torrent_dht
    5   format: text
    6   nice:   100

Even though you add an item of 'down' to protocols of UDP,
it must be meaningless.

## TCP Event Abstraction

Actually, TCP handles many events (and many states),
but the SF-TAP flow abstractor abstracts it by 3 events for simplicity.
The SF-TAP flow abstractor defines 3 events which are
CREATED, DATA, and DESTROYED events.
CREATED event is invoked when TCP session is established.
More precisely, it is invoked when finished 3-way handshake and
determined the application protocol by the regular expressions.
DATA event is invoked when arriving data.
DESTROYED event is invoked when closed TCP session because of
some reasons, which are timeout, reset, and fin-close.

## Protocol of Flow Abstraction Interface

not yet

## Pseudo Code for TCP Protocols

We give pseudo code to analyze TCP protocols as follows.

    // connect to socket
    s = socket();
    connect(s, "/tmp/sf-tap/tcp/http");

    for (;;) {
        // read header
        line = readline(s);
        h = parse_header(line);
    
        // generate session ID
        sid = new sessionID(h["ip1"], h["ip2"], h["port1"], h["port2"], h["hop"]);
    
        if (h["event"] == "DATA") {
            // data is arriving
            // h["from"] and h["match"] help to determine data origin
            read(s, buf, h["len"]);
        } else if (h["event"] == "CREATED") {
            // created TCP session
        } else if (h["event"] == "DESTROYED") {
            // destroyed TCP session
        }
    }

In this code, first of all, we connect to the file of UNIX domain socket.
Then we read line, which is a header, from the socket.
We parse the line to interpret the header, and generate the session ID by using
it.
After that, we read data if the event is DATA where
the data length is specified by "len" key in the header.

The skeleton written in Python is available on [GitHub Gist](https://gist.github.com/ytakano/87fcb3377df3c29c60c3 "GitHub Gist").
