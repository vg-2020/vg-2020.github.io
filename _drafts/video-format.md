---
layout: post
title: "video 格式YUV"
date: 2020-11-28 00:01:00 -0000
categories: base
tag: [c++,thread]
---
>引言，倒入主题

介绍
================

RGB
================
linear space
gamma-corrected/gamma-compressed  non linear space
- RGB Components: defined as an (r,g,b) triple, usually between 0–1.0 (computer graphics) or 0–255 (web), identifies a colour in a RGB colour space.

sRGB image → degamma → process → gamma → sRGB image
- **linear sRGB** (often called linear RGB, or gamma-expanded values)
- **non-linear sRGB** (or gamma-compressed values) are the RGB components expressed in the RGB colour space after the application of the transfer function.

YUV
================
- Y': The Y' component, also called luma,The prime symbol (') is used to differentiate luma from a closely related value, luminance. The prime symbol is frequently omitted, but YUV color spaces always use luma, not luminance.
- Y: Luminance is derived from linear RGB values, whereas luma is derived from non-linear (gamma-corrected) RGB values.
- U/V: chroma values or color difference values, are derived by subtracting the Y value from the red and blue components of the original RGB color
    - U: Cb, subtracting the Y value from blue
    - V: Cr, subtracting the Y value from red

YUV ~= Y'UV ~= Y'CbCr



Convert
================
Y' = Wr*R + WgG + WbB
U = B - Y'
V = R - Y'


storage
================

- YUV422 storage method

|Storage format | memory layout |  storage type | planes |
| ------------- |:-------------:| --------------:|-----------:|
|YUY2: | Y0U0Y1V0Y2U1Y3V1　　| YUV422SP　　|1 planes|
|UYVY: | U0Y0V0Y1U1Y2V1Y3　　| YUV422SP　　|1 planes|
|YVYU: | Y0V0Y1U0Y2V1Y3U1　　|YUV422SP　　|1 planes|

- YUV420 storage method


|Storage format | memory layout |  storage type | planes |
| ------------- |:-------------:| --------------:|-----------:|
| IYUV: | YYYYYYYY UU VV　　　　| YUV420P　　| 3 planes |
| YV12: | YYYYYYYY VV UU　　　　| YUV420P　　| 3 planes |
| NV12: | YYYYYYYY UVUV　　　　| YUV420P　　|  2 planes| 
| NV21: | YYYYYYYY VUVU　　　　| YUV420P　　| 2 planes| 


示例应用
================

参考
================
- https://en.wikipedia.org/wiki/YUV
- https://docs.microsoft.com/en-us/windows/win32/medfound/about-yuv-video
- https://observablehq.com/@sebastien/srgb-rgb-gamma

