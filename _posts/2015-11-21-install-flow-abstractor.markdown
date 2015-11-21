---
layout: post
title:  "Install SF-TAP Flow Abstractor on Ubuntu Linux"
date:   2015-11-21 17:00:00 +0900
categories: installation
---
## Ubuntu Version

This document is for Ubuntu 14.10, 15.04 and 15.10.

## Install Dependencies

First of all, install build-essential, cmake, git, libevent-dev, libboost-all-dev, libpcap-dev, libre2-dev, libyaml-cpp-dev.

    $ sudo apt-get install build-essential cmake git libevent-dev \
    libboost-all-dev libpcap-dev libre2-dev libyaml-cpp-dev

## Build and Run SF-TAP Flow Abstractor

### Get Source

Then, clone the source of the SF-TAP flow abstractor from GitHub.

    $ git clone https://github.com/SF-TAP/flow-abstractor.git

### Build

Build by cmake and make.

    $ cd flow-abstractor
    $ cmake -DCMAKE_BUILD_TYPE=Release CMakeLists.txt
    $ make

If you got an eorror regarding language locale, install suitable launguage pack.

    $ sudo apt-get install language-pack-ja

### Run

Run the SF-TAP flow abstractor specifying a network interface and a config file.

    $ sudo ./src/sftap_fabs -i eth0 -c ./examples/fabs.conf