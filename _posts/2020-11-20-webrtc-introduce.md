---
layout: post
title: "WebRTC介绍"
date: 2020-11-20 02:00:00 -0000
categories: webrtc
tag: [tech, webrtc]
---
>大部分音视频厂商都是基于WebRTC(Web Real-Time Communication)提供音视频实时应用，到底WebRTC 提供了什么呢？

概述
===============

先载录下[webrtc.org](https://webrtc.org/)上的描述
>With WebRTC, you can add real-time communication capabilities to your application that works on top of an open standard. It supports video, voice, and generic data to be sent between peers, allowing developers to build powerful voice- and video-communication solutions. The technology is available on all modern browsers as well as on native clients for all major platforms. The technologies behind WebRTC are implemented as an open web standard and available as regular JavaScript APIs in all major browsers. For native clients, like Android and iOS applications, a library is available that provides the same functionality. The WebRTC project is open-source and supported by Apple, Google, Microsoft and Mozilla, amongst others. This page is maintained by the Google WebRTC team.

总结下WebRTC提供了什么
- 一套标准的JavaScript APIs，这个可以用在浏览器里做实时音视频通讯功能，但不用安装任何plug-in。所有不同的浏览器提供统一的API
- 它是建立在一系列已有标准媒体协议基础上的。
- 在API下层是具体的实现，它是open-source的，Google WebRTC team 维护。
- 不仅是web app，也可以用在native的app。mobile 和 desktop
    > web基于标准Javascript API，在灵活性上受到限制相当是黑盒子要没法自己做特定优化。即便底层native支持,web app也不能用，要只能等javascript API支持。大部分厂家都是直接基于Native(Windows/Mac/Linux/Android/iOS)，web app只是补充。

音视频通用处理流程
==============
音视频处理流程基本都是类似，不管是WebRTC，还是自己实现的MediaSDK. 理解基本流程，对于理解整个WebRTC有帮助，就像一个地图一样，知道模块作用及其在流水线的哪个位置。

总的来说媒体流就像游泳池池道一样，有不同的media(audio，video), 各不相关，但是在某些时候又要联系起来(例如带宽分配，lip-sync等)

能力协商
--------------
这部分主要是生成Local-SDP([Session Description Protocol](https://tools.ietf.org/html/rfc4566),发送Local-SDP 给Peer(其它终端，或MediaServer), 收到Peer的Remote-SDP, Local-SDP和Remote-SDP做协商。

关于SDP不做细节讨论，SDP包括的信息大致包括
- 基础信息
    - SDP版本，终端名，会话名等
    - 终端网络类型，地址类型，地址等
        - 如果是用ICE并且是session多路复用，那就是包含ICE信息
        - 如果不是多路复用，ICE在每个session里定义
- 之后就是多个media-session段
- 具体每个media-session 信息
    - session 类型 ```m=``` + ```a=content``` (audio-main, video-main, audio-slide, video-slide, 其它app)
        >   
            - audio-main:     麦克风音频
            - video-main:     摄像头视频
            - audio-slide:    PC机器播放音频
            - video-slide:    屏幕共享
            - other app:      其它应用media,例如BFCP, 白板等.
    - 支持的codec列表
    - 支持的Format/Payload type
    - ICE信息
    - 支持的打包模式
    - 支持的RTP/RTCP扩展
    - 支持的方向，send, recv sendrecv,
    - 状态active/inactive
    - 等其它


Signaling的本质要做的事情是让所有的终端交换SDP，通过两端的SDP里的参数信息，协商出一致的Codec，打包模式， RTP扩展等，及其连接信息建立连接。


媒体处理流程
-------------- 
大部分复杂的逻辑在音视频的处理流水线里。这里包括
- 音视频设备枚举,管理和采集
- 音视频处理
    - 语音: 回音消除,降噪,PLC(Packet loss concealment,中文得怎么翻译？) 等
    - 视频: 颜色空间转换,旋转，去噪等
- Encode/Decode
- SRTP/SRTCP Pack/Uppack
- QoS/带宽评估分配/FEC/RTX
- 传输(STUN/TURN/ICE/UDP/TCP/DTLS)
- 平滑发送/JitterBuffer
- A/V lipsync/Smooth play
- Render & Play

基本处理流如图
![webrtc-pipeline](/assets/webrtc/webrtc-pipeline.svg){:height="100%" width="100%"}

Web API
===============
Web API只是对接口的统一和抽象，作为标准达到不同的厂商间的接口兼容 
- PeerConnection
    它代表的是一个完整的P2P的媒体连接，这个是WebRTC的js统一接口，提供了大部分的建立音视频会话的接口和事件.
    > 本质上多人会和1:1会都是P2P。1:1是2个终端的P2P。 多人会(MCU/SFU)是每个终端和媒体服务器的P2P。
    - 维护建立媒体连接的状态和调用流程。
    - 作为容器包容其它的对象，及其建立对象之间的关联。
- MediaDevices
    所有的媒体的起源就是先要有媒体源，不管是来自设备，还是文件，或其它。 MediaDevices就是管理音视频设备及其采集音视频数据
    - enumerateDevices
    - devicechange
    - getUserMedia(constraints)
        - 采集麦克风和摄像头，返回MediaStream
        - constraints
            ```javascript
            const gdmOptions = {
                video: {
                    cursor: "always"
                },
                audio: {
                    echoCancellation: true,
                    noiseSuppression: true,
                    sampleRate: 44100
                }
            }
            ```
    - getDisplayMedia(constraints)
        - 采集屏幕和计算机输出音频，返回MediaStream
        - constraints example
            ```javascript
            const gdmOptions = {
                video: {
                    cursor: "always"
                },
                audio: {
                    echoCancellation: true,
                    noiseSuppression: true,
                    sampleRate: 44100
                }
            }
            ```

- MediaStream: 代表从同一个源来的媒体，
    - 由一个或多个MediaStreamTrack组成.
    - 在同一MediaStream的MediaStreamTrack需要同步(例如lip-sync)
    - MediaStreamTrack: 
        - 代表某一类型的媒体，例如来自摄像头的video-main，来自屏幕的video-slide，来自麦克风audio-main，来自扬声器的audio-slide等。
        - 有接收MediaStreamTrack,和发送的MediaStreamTrack

- RTCRtpTransceiver: 代表一个在SDP里的一类(```m=``` + ```a=content:```)media content (audio-main, video-main, audio-slide, video-slide)
    - RTCRtpSender:     某一类media content的发送pipeline
        - 有一对应的MediaStreamTrack,
        - codecs, encode-params, rtp-externsions, transport等
    - RTCRtpReceiver:   某一类media content的接收pipeline
        - 每路MediaStreamTrack对应一个RTCRtpReceiver
        - codecs, encode-params, rtp-externsions, transport等


总的来说WebAPI提供有限制的对象和接口以及设置对象参数等. 一些更底层的media相关的优化做不到。这就是为什么大部分厂商用直接用native webrtc做native客户端，web app只是做补充。 

对象关系图
![webrtc-webapi](/assets/webrtc/webrtc-webapi.svg){:height="100%" width="100%"}

Native 实现
===============
如果Web API只是对接口的统一和抽象，那Native就包括具体的细节实现


参考
===============
- https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API
- https://w3c.github.io/mediacapture-main
