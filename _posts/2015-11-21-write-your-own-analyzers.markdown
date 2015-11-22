---
layout: post
title:  "Write Your Own Analyzers"
date:   2015-11-21 18:39:00 +0900
categories: tutorial
---

# How to Write Application Level Analyzers

In this document, we show how to implement analyzers for the SF-TAP flow abstractor.

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

The protocol on flow abstraction interfaces consists of chunks
of header and data.
Note that data is involved only if chunk's event is DATA.

The header is ended \n (line feed), and denoted as follows.

    ip1=192.168.24.54,ip2=216.58.221.196,port1=59547,port2=80,hop=0,l3=ipv4,l4=tcp,event=CREATED\n

This is equivalent for

    {
      "ip1":   "192.168.24.54",
      "ip2":   "216.58.221.196",
      "port1": 59547,
      "port2": 80,
      "hop":   0,
      "l3":    "ipv4",
      "l4":    "tcp",
      "event": "CREATED"
    }

in JSON. When the event is CREATED or DESTROYED, the chunk does not involve data.
Otherwise, when the event is DATA, the chunk involve data.
The header of DATA event is denoted as follows.

    ip1=192.168.24.54,ip2=216.58.221.196,port1=59547,port2=80,hop=0,l3=ipv4,l4=tcp,event=DATA,from=2,match=down,len=494\n

In this case, the header indicates that
data is coming from (ip2=216.58.221.196, port2=80) because
the header denotes from=2.
The entry of match=down indicates that the which regular expressions,
provided in the configuration file shown before.
Accordingly, you can determine whether the data is from server or client
by the 'match' filed.
The data length is indicated by len=494 in this case.

Since multiple flows are outputted to a single abstraction interface,
it is required to identify chunks by flow identifiers.
The SF-TAP flow abstractor identifies each flow by using 5-tuple as follows.

    (ip1, port1, ip2, port2, hop)

The 'hop' filed is used for loopback7 interface,
and the SF-TAP flow abstractor increments it internally.
The loopback7 is used for re-injecting flows for encapsulated flows,
such as HTTP proxy.
We do not show more details of the loopback7 interface here.

Note that CREATED and DESTROYED events are not invoked when UDP protocols
since UDP is not connection oriented.
Thus, you need handle only DATA event for UDP protocols.

## Pseudo Code for TCP Protocols

We give pseudo code to analyze TCP protocols as follows.

    // connect to socket
    s = socket();
    connect(s, "/tmp/sf-tap/tcp/http");

    loop {
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
Then we read a line, which is a header, from the socket.
We parse the line to interpret the header, and generate the session ID by using
it.
After that, we read data if the event is DATA where
the data length is specified by 'len' field in the header.

The skeleton written in Python is available on [GitHub Gist](https://gist.github.com/ytakano/87fcb3377df3c29c60c3 "GitHub Gist").

## Pseudo Code for UDP Protocols

Then, we give pseudo code to analyze UDP protocols as follows.

    // connect to socket
    s = socket();
    connect(s, "/tmp/sf-tap/udp/dns");

    loop {
        // read header
        line = readline(s);
        h = parse_header(line);
    
        if (h["event"] == "DATA") {
            // data is arriving
            // h["from"] help to determine data origin
            read(s, buf, h["len"]);
        }
    }
