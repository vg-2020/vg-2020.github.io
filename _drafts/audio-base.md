---
layout: post
title: "Audio 基础"
date: 2020-11-28 00:01:00 -0000
categories: base
tag: [audio,media]
---

音频基本概念
=========================
- 采样率(Sample Rate)
    - 8 kHz (phone quality)
    - 16 kHz (for common speech recognition)
    - 22 kHZ (普通FM广播的音质)
    - 44.1 kHz (CD quality)
    - 48 kHz通常用于音像制作，
    - 96 kHz
    - 192 kHz
    - 人耳可听的声音20Hz~20kHz
    - 其中1kHz～4kHz赫兹是人耳最敏感的区域，MP3品质为8kHz，就已经可以基本满足收听音乐的需求了
- 位深度(Bit Depth)
    - 8-bit  (uint8 with range: 0 ~ 255)
    - 16-bit (int16 with range: 0 ~ 65536)
    - 24bit (uint24 with rang:  0 ~ 16777216)
    - 32bit (uint32 with rang:  0 ~ 4294967296)
- 通道数(Number of Channels)
    - Mono: 1 channel
    - Stereo: 2 channels
- 比特率（Bit Rate): 这是每单位时间的位数
    - 比特率 = 采样频率 * 采样位数 * 通道数
    - 16(bit/samples) * 44100(samples/s) * 2(channels) = 1411200(bits/s) = 1411.2(kbps)
- 音响
    - 声压(Sound pressure)
        ![声压](/assets/audio/sound_pressure.png){:height="50%" width="50%"}
        - 
    - 分贝(decibel) dB
        - 无量纲,它描述的是一个比值. dB经常用作为表征声压级SPL（Sound Pressure Level）的大小
            ![dB](/assets/audio/db.jpg){:height="50%" width="50%"}
        - 人耳可听的声压幅值波动范围为2×10^-5Pa(20μPa)~20Pa,这个差距是10^6, 如果用图形线性表示的话，小的幅值很难体现出来。对数的特点可以表示小的数字的细节。所以用幅值dB表示对应的分贝数为0~120dB
        - 0db = 20μPa, 120db = 20Pa
- 音调
    - 人的听觉频率范围是20Hz～20kHz，其中1kHz～4kHz赫兹是人耳最敏感的区域。
- 音色
    - 波形决定，频率+幅值
    - 基波+多个谐波
        - 基波: 分频后， 频率最小的波
        - 谐波: 除基波外其它的频率波
        - 最终的声音波形是，无数谐波在基波的合成
    - 

- 存储格式
    - uncompressed
        - PCM (Pulse Code Modulation)/LPCM (Linear Pulse Code Modulation)
            - [wav](https://en.wikipedia.org/wiki/WAV)
            - AIFF (Audio Interchange File Format)
        - PDM(Pulse-density modulation)
            - DSD (Direct Stream Digital)
    - lossless
        - FLAC (Free Lossless Audio Codec)
        - ALAC (Apple Lossless Audio Codec)
        - WMAL (Windows Media Audio Lossless)
    - lossy
        - MP3 (MPEG-1 Audio Layer 3)
        - AAC (Advanced Audio Coding)
        - WMA (Windows Media Audio)
        - iLBC (Internet Low Bit Rate Codec)
        - Opus
    - Ogg

- HD Audio(High-Resolution audio/High-Definition Audio/HD audio)
    - 96kHz/24bit,192kHz/24bit

- MQA(Master Quality Authenticated)


References
=========================
- https://en.wikipedia.org/wiki/Sound_pressure
- https://zh.wikipedia.org/wiki/%E5%A3%B0%E5%8E%8B
- https://en.wikipedia.org/wiki/List_of_codecs