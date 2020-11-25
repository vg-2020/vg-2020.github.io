---
layout: post
title: "video codec 概念"
date: 2020-11-28 00:01:00 -0000
categories: media
tag: [webrtc, video]
---
> 做video相关的开发的时候，未必是要是codec专家，但是要知道一些video codec 基本的概念

编码帧类型(frame type)
=======================
类型
----------
- I-Frame: 可独立decode成功，不依赖其它frame
- P-Frame: 需要参考前面的I或P frame 才能decode成功
- B-Frame: 及参考前面的I/P frame，也参考后面的I/P frame, 压缩效率高。但是一般有实时要求的场景下不用。

帧参考模型
----------

Macroblocks
==================
每帧图片被分为最小单位是宏块(MacroBlock),一般大小是16x16, 8x8, 4x4，越小越能保持细节，但压缩效率下降


Slices
========================
多个宏块组成Slice.编码不是基于宏块，在某个slice内部做运动检测等。这样可以并行，提高编码速度.
但是slice多太多，降低编码效率。

每个slice被生成NAL,所以slice个数也受packetization-mode影响，如果packetization-mode是0，不支持FU(fragmentation unit),一个NAL必须能放到一个包里， 这样必须限制NAL大小,必须小于MTU(maximum transmission unit)，这样限制slice大小，数目就会多。 压缩效率低。

和frame 类型对应
slide也有I-Slice，P-Slice，B-Slice




Group of pictures
========================
M=3, N=12 >> IBBPBBPBBPBBI
M=5, N=15 >> IBBBBPBBBBPBBBBI


video sequence
========================
SPS: Sequence Parameter Set
PPS: Picture Parameter Set


Temporal Layer
========================



References
=========================
- https://en.wikipedia.org/wiki/Video_compression_picture_types
- https://en.wikipedia.org/wiki/Group_of_pictures