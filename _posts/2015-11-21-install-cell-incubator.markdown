---
layout: post
title:  "Install SF-TAP Cell Incubator on FreeBSD"
date:   2015-11-21 17:22:00 +0900
categories: installation
---
## FreeBSD Version

This document is for FreeBSD 10.1 and 10.2.

## Build netmap Enabled Kernel

First of all, you need build netmap enabled kernel to use the SF-TAP cell incubator.
Edit kernel config as follows to build the kernel.

    # cd /usr/src/sys/amd64/conf
    # cp GENERIC GENERIC.netmap
    # vi GENERIC.netmap
    
    device netmap # add device of netmap

Then, compile and install the kernel.

    # cd /usr/src
    # make -j 30 buildkernel KERNCONF=GENERIC.netmap
    # make installkernel KERNCONF=GENERIC.netmap

And reboot.

    # reboot

Finally, confirm that netmap is enabled.

    # ls /dev/netmap
    /dev/netmap

## Install Dependencies

Install git to get the source.

    # pkg install git-x.x.x

## Build Cell Incubator

Clone the souce form GitHub.

    $ git clone https://github.com/SF-TAP/sf-incubator.git

Then, build it.

    $ cd sf-incubator/src
    $ make

## Run

Before running the SF-TAP cell incubator, disable offload engines of NICs.
The shell script for disabling the engines for all NICs is included in the repository. Thus, you just need execute the script as follows.

    $ sudo ./misc/ifcap_disable.sh

Finally, you can run the SF-TAP cell incubator as follows.


    # ./qb-separator -r ix0 -t igb0,igb1,igb2,igb3

In this case, ix0 is a NIC to capture network traffic, and
igb[0-3] are NICs to which separated flows are forwarded.
