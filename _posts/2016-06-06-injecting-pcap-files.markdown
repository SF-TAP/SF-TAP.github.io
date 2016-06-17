---
layout: post
title:  "Injecting pcap Files to SF-TAP Flow Abstractor"
date:   2016-06-06 05:35:48 +0900
categories: tutorial
---

## pcap Interface

SF-TAP flow abstractor provides the interface for injecting pcap files.
The injection can be done by writing pcap files to the interface as follows.

    $ ls /tmp/sf-tap/pcap
    /tmp/sf-tap/pcap
    $ cat dump01.pcap dump02.pcap | sudo nc -U /tmp/sf-tap/pcap

## Limitation

Multiple pcap files beeing injected must be same endian,
version numbers and timezone.
If you want to inject pcap files captured different environments,
inject the files one by one as follows.

    $ cat dump01.pcap | sudo nc -U /tmp/sf-tap/pcap && \
    cat dump02.pcap | sudo nc -U /tmp/sf-tap/pcap

Generally, traffic capturing is performed on singile environment,
so, you may not need care about this.