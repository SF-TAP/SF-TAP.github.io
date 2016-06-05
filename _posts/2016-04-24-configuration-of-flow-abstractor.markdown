---
layout: post
title:  "Configuration of SF-TAP Flow Abstractor"
date:   2016-04-24 12:51:42 +0900
categories: tutorial
---

## Configuration File

SF-TAP flow abstractor reads configuration file written in YAML when waking up.
In this document, we explain how to write the configuration file for it.

## Example Configuration

An example configuration is as follows.
We explain each section of the configuration here.

    # global configuration
    global:
      home:    /tmp/sf-tap
      timeout: 30  # close long-lived (over 30[s]) but do-nothing connections
      lru:     yes # bring the least recently used pattern to front of list
      cache:   yes # use cache for regex
      tcp_threads:   2
      regex_threads: 2
    
    pcap:
      if: pcap
    
    loopback7:
      if:     loopback7
      format: text
    
    tcp_default:
      if:     default # for every flow that wasn't matched by any rules
      proto:  TCP
      format: text
      body:   yes
    
    udp_default:
      if:     default # for every flow that wasn't matched by any rules
      proto:  UDP
      format: text
      body:   yes
    
    http:
      up:     '^[-a-zA-Z]+ .+ HTTP/1\.(0\r?\n|1\r?\n([-a-zA-Z]+: .+\r?\n)+)'
      down:   '^HTTP/1\.[01] [1-9][0-9]{2} .+\r?\n'
      proto:  TCP  # TCP or UDP
      if:     http
      format: text # text or binary
      body:   yes  # if specified 'no', only header is output
      nice:   100  # the smaller a value is, the higher a priority is
      utf8:   no   # treat data as UTF8 or latin1 (binary). used for regex
      balance: 4   # flows are balanced by 4 interfaces
    
    http_proxy:
      up:     '^(CONNECT|connect) .+ HTTP/1\.(0\r?\n|1\r?\n([-a-zA-Z]+: .+\r?\n)+)'
      down:   '^HTTP/1\.[01] 200 .+\r?\n'
      proto:  TCP  # TCP or UDP
      if:     http_proxy
      format: text # text or binary
      body:   yes  # if specified 'no', only header is output
      nice:   90   # the smaller a value is, the higher a priority is
      utf8:   no   # treat data as UTF8 or latin1 (binary)
    
    syslog_udp:
      up:     '^<([0-9]|[1-9][0-9]|1[0-8][0-9]|19[01])>'
      proto:  UDP
      if:     syslog
      format: text
      port:   514
      nice:   100
      utf8:   no
    
    dns_udp:
      proto:  UDP
      if:     dns
      port:   53,5353,5355 # 53: UDP DNS, 5353: multicast DNS, 5355: LLMNR
      format: text
      nice:   200
      utf8:   no

### global section

The global section can contain 6 subsections, which are "home", "timeout",
"lru", "cache", "tcp_threads", "regex_threads", to control
behavior of SF-TAP flow abstractor.

|home         | used to indicate the root directory to which abstraction iterfaces are located|
|timeout      | discard TCP sessions which do nothing for "timeout" seconds |
|lru          | regular expressions are internally managed by the least-recently-used manner  |
|cache        | regular expressions are cached when matching |
|tcp_threads  | the number of threads for handling TCP sessions |
|regex_threads| the number of threads for classifing flows by regular expressions |

### TCP section

TCP sections can contain following subsections.

|up      | the regular expression for up stream |
|down    | the regular expression for down stream |
|proto   | this subsection must be "TCP" for TCP |
|if      | file name of the flow abstraction interface |
|format  | spefify header format of the flow abstraction interface. this must be "text" or "binary" |
|body    | the flow abstraction interface outputs TCP payload or only headers |
|nice    | preference for classifing by regular expressions |
|utf8    | palyload is treated as binary or UTF-8 string. this subsection affects behavior of regular expressions |
|port    | classify by TCP port numbers. you can specify multiple port numbers such as 100,200,300-310 |
|balance | the number of interfaces for load-balancing |

The nice subsection indicate the preference for regular expressions.
For example, if nice values of http and http_proxy are 100 and 90,
flows are distinguished by using http_proxy's regular expressions,
and if the flows are classified as not http_proxy,
then the flows are distinguished by using http's regular expressions.

### tcp_default section

This section is for the default interface of TCP flows.
TCP flows are classified by regular expressions and port numbers
indicated in TCP sections.
The default TCP interfaces is the exit of flows which are not classified
by the TCP sections.
This section can contain following subsections.

|if      | interface name |
|proto   | must be "TCP" |
|format  | must be "text" or "binary" |
|body    | output TCP payload or not |
|balance | the number for load-balancing |

### UDP section

UDP sections are for UDP packets.
The subsections are quite similar to TCP section's,
but UDP can contain only 1 regular expression for classification,
which are indicated by "up" subsection.

|up      | the regular expression for up stream |
|proto   | this subsection must be "TCP" for TCP |
|if      | file name of the flow abstraction interface |
|format  | spefify header format of the flow abstraction interface. this must be "text" or "binary" |
|body    | the flow abstraction interface outputs TCP payload or only headers |
|nice    | preference for classifing by regular expressions |
|utf8    | palyload is treated as binary or UTF-8 string. this subsection affects behavior of regular expressions |
|port    | classify by TCP port numbers. you can specify multiple port numbers such as 100,200,300-310 |
|balance | the number of interfaces for load-balancing |


### udp_default section

This section is for the default interface of UDP.
This section can contains following subsections.

|if      | interface name |
|proto   | must be "UDP" |
|format  | must be "text" or "binary" |
|body    | output TCP payload or not |
|balance | the number for load-balancing |

### loopback7 section

This section is used for the loopback7 interface.
This section can contain "if" and "format" subsections as follows.

|if    | interface name|
|format| must be "text" or "binary"|

### pcap section

This section is for the pcap interface.
This section can contain only the "if" subsection.