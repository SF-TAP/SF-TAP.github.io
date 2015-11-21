---
layout: post
title:  "Flow Separating, and L2 Mirroring and Bridging"
date:   2015-11-21 18:22:00 +0900
categories: tutorial
---

# Tutorial of SF-TAP cell incubator

Here, suppose that we have a following FreeBSD box, which has two 10 GbE (ix0 and ix1) and four 1 GbE (igb0, igb1, igb2 and igb3) interfaces.
-r is a prefix of "RIGHT", and -t is a prefix of "TAP".

![qb01 qb01](/assets/qb/qb01.png)

## Flow Based Separating

We can use qb-separator for traffic separating as follows.
qb-separator must be executed with root privilege.

    # ./qb-separator -r ix0 -t igb0,igb1,igb2,igb3

![qb02 qb02](/assets/qb/qb02.png)

Here, qb-separator captures traffic from ix0, then separates and forwards it to
igb[0-3].
Note that it separates traffic by using the hash values of
IP addresses and port numbers of captured packets.

## Mirroring

We can also use qb-tap for traffic mirroring as follows.
qb-tap also must be executed with root privilege.

    # ./qb-tap -r ix0 -t igb0,igb1,igb2,igb3

![qb03 qb03](/assets/qb/qb03.png)

Here, qb-tap forwards all captured traffic from ix0 to igb[0-3].

## L2 Bridging

We can use qb-separator or qb-tap as a simple software L2 bridge as follows.

    # ./qb-separator -r ix0 -l ix1

-l is a prefix of "LEFT".

![qb04 qb04](/assets/qb/qb04.png)

## L2 Bridging + Flow Based Separating

We use qb-separator as a L2 bridge and traffic separator as follows.

    # ./qb-separator -r ix0 -l ix1 -t igb0,igb1,igb2,igb3

![qb05 qb05](/assets/qb/qb05.png)

Here, qb-separator separates all traffic from ix0 and ix1
and forwards it to igb[0-3].

## L2 Bridging + Mirroring

Similarly, qb-tap can work as a L2 bridge and traffic mirroring box as follows.

    # ./qb-tap -r ix0 -l ix1 -t igb0,igb1,igb2,igb3

![qb06 qb06](/assets/qb/qb06.png)

Here, all traffic caputured from ix0 and ix1 is forwarded to igb[0-3].