---
layout: post
title: "实时会议系统架构介绍"
date: 2020-11-24 00:01:00 -0000
categories: media
tag: [tech, conference, realtime]
---
>实时通讯开始都是一对一的，相对简单，能p2p最好，不能p2p直连的话，就通过TURN server中转，但是当多人会议通讯后，整个架构就复杂了。

实时通讯媒体架构类型
================
- MESH
- MCU (MultiPoint Control Unit)
- SFU (Selective Forwarding Unit)
- Hybrid: SFU + Transcode + MCU

MESH
----------------
这个就是基于多人P2P形成一个网状结构,当然前提是俩俩的P2P要能通。连接数目和带宽的限制，不能人太多。(n-1)路上行加(n-1)路下行(n:总人数)，和用户数目线性增加。

考虑360p(640kbps)+audio(24kbps)=664kbps*4(5个终端) ~= 2.7mbps，
需要上下行2.7mbps，如果考虑FEC/RTX包，需要更多带宽.

![MESH basic](/assets/media_arch/media-arch-mesh-1.png){:height="50%" width="50%"}

一些场景实时用户少， 但是非实时用户多，也可以mesh方法。

![MESH enhance](/assets/media_arch/media-arch-mesh-2.png){:height="50%" width="50%"}

MCU
----------------
MCU 主要是把多个用户的流整合为一个流.
基本过流程
- decode 输入的多个流
- 合成多路流为一路流, 在encode
    - video: 按一定的layout合成后encode。 
    - audio: mix audio后encode
- 最后只要发给一个流给用户。 

![MCU](/assets/media_arch/media-arch-mcu.png){:height="50%" width="50%"}

优点是节约带宽，包括上下行，都只要一路流。
理论上可以针对每个终端带宽，encode特定流，但是考虑到费用问题，一般做聚合分组。

>例如用户带宽如下
A: 2.2mbps, B: 2.0mbps, C: 1.9 mbps
- 一种方法，encode 3个不同3路720p流. 
- 另外就是把A/B/C做为一组，encode 一路1.9mbps的720p流

缺点就是费用高，单个MCU节点连接的用户少, 节点CPU/Memory 要求高。在有encode/decode 带来额外的延时。为提高性能一般MCU Node用hardware.

SFU (+ Transcode)
----------------
每个终端发自己的流,SFU负责转发流给其它的终端。

![SFU](/assets/media_arch/media-arch-sfu.png){:height="50%" width="50%"}
明显它对server性能要求比MCU低，但是要求用户更高的带宽。
但是灵活。接收端可以做不同的处理， 而且通过订阅选择要接收那些用户。SFU可以根据订阅选择行转发.
MCU也可以，但cost很高。 因为它需要增加encode,每个新的组合要增加一次encode.

前面都是理想状态，真正的产品环境是每个终端的网络环境，性能能力是不同的。

>例如，如果其中一个终端的带宽差，不得不降低质量，这个时候最差的带宽的用户就把其它用户的体验都拉下来了。

![SFU issue](/assets/media_arch/media-arch-sfu-issue.png){:height="50%" width="50%"}

有2个解决方法。 
- 如果发送端能力强，带宽好，可以用simulcast， 发送2路不同质量的流，SFU根据其它接收端的能力做不同的转发。

![SFU+simulcast](/assets/media_arch/media-arch-sfu-simulcast.png){:height="50%" width="50%"}

- 另外也有可能能力带宽不能支持simucast，可以用transcoder，在server端做转码，这个是按需做的，总体比MCU费用低。

![SFU+transcoder](/assets/media_arch/media-arch-sfu-transcoder.png){:height="50%" width="50%"}

transcoder 还有的用户场景是codec,例如所有用户支持AV1，但是一个用户支持H264， 如果协商最后是AV1， 难transcoder 负责decode AV1 -> encode H264 对于这个特定用户。 audio transcoder 概率高点。 

Hybrid (SFU + Transcode + MCU)
----------------
真正产品上都是用混合模式，不同的需求，不同的用户场景，选择不同的架构。
- 某个低端终端，用MCU，当其它终端用SFU

![SFU+MCU](/assets/media_arch/media-arch-sfu-mcu.png){:height="50%" width="50%"}

- 录制需要合成，给只有的playback用。节省storage

![SFU+MCU](/assets/media_arch/media-arch-sfu-mcu2.png){:height="50%" width="50%"}

总结比较
----------------


| 比较角度 | MESH | MCU |SFU |
|-------|--------|---------|---------|
| 综合成本 | 低 | 高 |中 |
| 复杂度 | 低 | 中 |高 |
| 带宽要求 | 高 | 低 |高 |


随着用户带宽越来越好，SFU是主要的架构，但是针对不同的用户场景，其它做补充。
