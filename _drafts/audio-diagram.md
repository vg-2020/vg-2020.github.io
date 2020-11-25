---
layout: post
title: "音频图"
date: 2021-01-01 00:01:00 -0000
categories: base
tag: [audio,media]
---
在做音频分析总是有各种图形表示，它们都代表什么。 

- 时域图
    - 横轴是时间(t), 
    - 纵轴是声音幅值(dB，或Pa)
>这个是最基本的，
- 频域图
    - 幅度谱
        - 横轴是频率
        - 纵轴是声音幅值(dB，或Pa)
    - 相位谱
        - 横轴是频率
        - 纵轴是相位差(-Pi ~ +Pi)
- 语谱图(Spectrogram)
    - 横轴是时间(t)
    - 纵轴是频率
    - 窄带语谱图 vs 宽带语谱图
> 生成过程是，首先是按时间分帧(例如时间段:20ms,或采样数:128个采样数),对于每帧生成幅度频域图，并把幅度映射到一个灰度级表示，以此做为此帧语谱图基本图，以时间为横轴生成一个二维图但表示三维(时间， 频率，和声音幅值<灰度级>)。



References
=========================
- https://en.wikipedia.org/wiki/Spectrogram