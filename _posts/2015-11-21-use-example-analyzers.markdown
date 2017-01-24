---
layout: post
title:  "Use Example HTTP and DNS Analyzers"
date:   2015-11-21 19:01:00 +0900
categories: tutorial
---

# Flow Abstraction Interfaces

SF-TAP flow abstractor provides flow abstraction interfaces by using UNIX domain socket.
If you run it, there are files in a directory as follows.

    $ cd /tmp/sf-tap
    $ ls -R
    loopback7=   tcp/         udp/

    ./tcp:
    default=         http=            smtp=            torrent_tracker=
    dns=             http_proxy=      ssh=             websocket=
    ftp=             irc=             ssl=

    ./udp:
    default=     dns=         torrent_dht=

Analyzers of SF-TAP connect to these interfaces and read flows to analyze.
In this document, we show how to use example HTTP and DNS analyers provided by us.

## Get Source

First of all, you must get source codes of the analyzers form GitHub.

    $ git clone https://github.com/SF-TAP/protocol-parser.git
    $ cd protocol-parser

---

## HTTP Analyzer

An example HTTP analyzer written in Python3 connects a flow abstraction interface of HTTP, parses HTTP protocol, and outputs JSON to the standard output.
You can execute it as follows.

    $ python3 http/sftap_http.py /tmp/sf-tap/tcp/http nobody

Here, /tmp/sf-tap/tcp/http is a path to a flow abstraction interface of HTTP,
and nobody is a flag specifying not to include HTTP body in outputs.

If you access to any HTTP server when running the HTTP analyzer,
you can get information of HTTP access in JSON.

For example, if you connect to http://sf-tap.github.io/ in another prompt,

    $ curl http://sf-tap.github.io/

, then you will get JSON data from the standard output as follows.

    {
      "server":{
        "response":{
          "msg":"OK",
          "code":"200",
          "ver":"HTTP/1.1"
        },
        "ip":"103.245.222.133",
        "header":{
          "cache-control":"max-age=600",
          "content-length":"8418",
          "age":"0",
          "expires":"Sat, 21 Nov 2015 10:41:48 GMT",
          "date":"Sat, 21 Nov 2015 10:42:19 GMT",
          "x-fastly-request-id":"2fa8bb058315c7d938574efee8e564f03e4d5d08",
          "access-control-allow-origin":"*",
          "x-timer":"S1448102539.814963,VS0,VE180",
          "vary":"Accept-Encoding",
          "server":"GitHub.com",
          "x-github-request-id":"67F5E016:17E1:1286355:56504813",
          "x-cache":"HIT",
          "content-type":"text/html; charset=utf-8",
          "connection":"keep-alive",
          "via":"1.1 varnish",
          "accept-ranges":"bytes",
          "x-cache-hits":"1",
          "last-modified":"Sat, 21 Nov 2015 09:56:46 GMT",
          "x-served-by":"cache-itm7421-ITM"
        },
        "port":80
      },
      "client":{
        "method":{
          "method":"GET",
          "ver":"HTTP/1.1",
          "uri":"/"
        },
        "header":{
          "host":"sf-tap.github.io",
          "user-agent":"curl/7.43.0",
          "accept":"*/*"
        },
        "port":64755,
        "ip":"192.168.24.52"
      }
    }

---

## DNS Analyzer

An example DNS analyzer written in C++11 also analyzes DNS protocol and outputs
JSON to the standard output.
Note that this is only for UDP.

### Build and Run DNS Analyzer

Build the DNS analyzer by using cmake and make.

    $ cd protocol-parser/dns
    $ cmake -DCMAKE_BUILD_TYPE=Release CMakeLists.txt
    $ make

Then run it.

    $ ./sftap_dns /tmp/sf-tap/udp/dns

If you send any DNS queries, you will get information of DNS access in JSON
from the standard output.

For example, if you look up sf-tap.github.io in another prompt,

    $ dig sf-tap.github.io

you will see DNS queries in JSON as follows.

    {
      "src":{"ip":"192.168.24.52", "port":54459},
      "dst":{"ip":"192.168.24.1", "port":53},
      "id":60923, "qr":0, "op":0, "aa":0, "tc":0,
      "rd":1, "ra":0, "z":0, "ad":0, "cd":0, "rc":0,
      "query_count":1,
      "answer_count":0,
      "authority_count":0,
      "additional_count":0,
      "query":[{"name":"sf-tap.github.io", "type":"A", "class":1}],
      "answer":[],
      "authority":[],
      "additional":[]
    }

    {
      "src":{"ip":"192.168.24.1", "port":53},
      "dst":{"ip":"192.168.24.52", "port":54459},
      "id":60923, "qr":1, "op":0, "aa":0, "tc":0,
      "rd":1, "ra":1, "z":0, "ad":0, "cd":0, "rc":0,
      "query_count":1,
      "answer_count":2,
      "authority_count":4,
      "additional_count":4,
      "query":[{"name":"sf-tap.github.io", "type":"A", "class":1}],
      "answer":[
        {
          "name":"sf-tap.github.io", "type":"CNAME", "class":1,
          "ttl":3600, "cname":"github.map.fastly.net"
        },
        {
          "name":"github.map.fastly.net", "type":"A", "class":1,
          "ttl":30, "a":"103.245.222.133"
        }
      ],
      "authority":[
        {
          "name":"fastly.net", "type":"NS", "class":1,
          "ttl":71200, "ns":"ns1.p04.dynect.net"
        },
        {
          "name":"fastly.net", "type":"NS", "class":1,
          "ttl":71200, "ns":"ns4.p04.dynect.net"
        },
        {
          "name":"fastly.net", "type":"NS", "class":1,
          "ttl":71200, "ns":"ns2.p04.dynect.net"
        },
        {
          "name":"fastly.net", "type":"NS", "class":1,
          "ttl":71200, "ns":"ns3.p04.dynect.net"
        }
      ],
      "additional":[
        {
          "name":"ns1.p04.dynect.net", "type":"A", "class":1,
          "ttl":81318, "a":"208.78.70.4"
        },
        {
          "name":"ns2.p04.dynect.net", "type":"A", "class":1,
          "ttl":78428, "a":"204.13.250.4"
        },
        {
          "name":"ns3.p04.dynect.net", "type":"A", "class":1,
          "ttl":72003, "a":"208.78.71.4"
        },
        {
          "name":"ns4.p04.dynect.net", "type":"A", "class":1,
          "ttl":71749, "a":"204.13.251.4"
        }
      ]
    }
