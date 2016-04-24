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

    # global configuration
    global:
      home:    /tmp/sf-tap
      timeout: 30  # close long-lived (over 30[s]) but do-nothing connections
      lru:     yes # bring the least recently used pattern to front of list
      cache:   yes # use cache for regex
      tcp_threads:   2
      regex_threads: 2
    
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
