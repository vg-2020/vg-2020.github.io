---
layout: post
title: "线程安全分析"
date: 2020-11-28 00:01:00 -0000
categories: base
tag: [c++,thread]
---
>引言，倒入主题

介绍
================
简单介绍概念



RTP Extensions
================
- VID
    - ```a=extmap:1/sendrecv http://protocols.cisco.com/virtualid```
    - VID is 8 bits
    - RTP Extnsion Format
        - RTP EXT ID(4bits):  = 1
        - Length(4bits): (0 ~ N) (VID counts)
        - VID0 (8bits)
        - VID1...N (N*8bits) 
- Framemarking
    - ```a=extmap:2/sendrecv http://protocols.cisco.com/framemarking```
    - RTP Extnsion Format
        - RTP EXT ID: 4bit = 2
        - Length(4bits): 0
        - D(1bit): Discardable
        - S(1bit): Switching point 
        - TID(3bits): temporal identifier, 
            - t0, t1, t2.
            - t0: indepenpendent(I-frame, LTR)
            - t1 -> t0,
            - t2 -> t1 -> t1
        - Type(4bits)
            - P=0, IDR=1, GDR=2, LTRF=3
-  Audio level
    - >A multi stream conference solution should have a level indication added to each RTP audio packet to be able to calculate active speaker information without decoding audio on conference server. 
    - ```a=extmap:3/sendrecv urn:ietf:params:rtp-hdrext:ssrc-audio-level ```


RTCP Extensions
================
- 
- RTCP-XR (RTP Control Protocol Extended Reports)

细分主题2
================


示例应用
================

参考
================
- RTP/RTCP https://tools.ietf.org/html/rfc3550
- Extended RTP Profile for RTCP https://tools.ietf.org/html/rfc4585
- RTCP-XR: https://tools.ietf.org/html/rfc3611
- Guidelines for Extending RTCP https://tools.ietf.org/html/rfc5968
- Codec Control Messages in the AVPF https://tools.ietf.org/html/rfc5104
- 
