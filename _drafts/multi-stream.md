- multi-stream
# Basic
- switch policy
    - AS: Active-Speaker
    - RS: Receiver-Selection
    - AL: Activity Layout
- Synchronization source (SSRC)
    - Identify stream(SSRC1-720p, SSRC2-360p)
- Contributing source(CSRC)
- Sub-session Channel ID(8bit), (VID: virtual identifier)
    - AS-policy don't care the CSI, just care which stream is belong to active speaker.
    - CSI is 0, that means anyone. so receiver just bind stream with UI by the VID.
    - why RS-policy need VID?
        - It is more flexibile for client render. 
        - both AS/RS just bind VID with render. 
    - VID RTP Extension
        - ```a=extmap:1/sendrecv http://protocols.cisco.com/virtualid```
        - VID is 8 bits
        - RTP EXT ID: 4bit
        - VID Length: 4bit , 0 ~ N
        - VIDs (N*8bits) 
    
- Capture Source ID (CSI)
        - format: 32bits
            - Scene ID (24 bits): 
            - Reserved (7 bits): 
            - AV (1 bit): 
        - note
            - Identify source id,not stream id.
            - one CSI map multi-SSRC(SSRC1-720p, SSRC2-360p)
            - SCR need not take care of the SSRC. just care the CSI
# multstream header
    - Application Feedback Identifier (32 bits): 
        - Must be set to 0x4D535452 (the hexadecimal value of ‘MSTR’ in ASCII).
    - Multistream Message Type (12 bits): 
    - Version (4 bits): 
    - Sequence Number (16 bits): 

# SCR Sub-session Channel Request 
> include any number of requests, 0-reqeust is validate

- Sub-session Channel ID(8bit), (VID: virtual identifier)
- SourceID (8 bits)
- Length (16 bits) including VID & SourceID
- max-bitrate(32bits)
- preferred-bitrate(32bits)
- Policy Type (16 bits):  
- Policy Info Length (16 bits): 
    - not including the policy type and length fields
- Policy-Specific Info (N bits):
    - ActiveSpeaker
        - Priority (8 bits): 
            - 
        - ScreenGroupID (8 bits): 
            - Grouping adjacency ID (4 bits):
            - screen ID (4 bits):
        - Duplication Flag (1 bit): 
    - Activity Layout
        - Priority (8 bits): 
        - ScreenGroupID (8 bits): 
        - Duplication Flag (1 bit): 
        - Reserved (3 bits): 
        - MinimumStreams (4 bits): 
        - ActivityTimer (8 bits): 
    - ReceiverSelect
        - Capture Source ID (CSI) (32bit)
    - Composed
        - value(32bits) (Optional)
            - sitename in stream enble/disable 1bit:
            - resvered (31bits)
- Payload Type (8 bits): 
- Reserved (5 bits): 
- A (1 bit):  encoded avatar (image) 
- C (1 bit):  CVO (coordination of video orientation) 
- L (1 bit):  When set this bit requests that the sender always encode the media source in a landscape display aspect ration (e.g. 4:3 or 16:9), padding as required.  
- Payload Info Length (16 bits): 
- Payload-Specific Info (N bits): 
    - Max MBPS (32 bits): 
    - Max FS (16 bits): 
    - Max FPS (16 bits): 
    - Temporal Layers (8 bits): 
    - TLV Section (Variable Length):  
        - TLV Type (8 bits):  
            - Maximum Number of Reference Frames (1):  
            - Maximum Width (2):  
            - Maximum Height (3):  
        - TLV Length (8 bits):  
        - TLV Value (Variable Length):  
        
        
# SCA:Sub-session Channel Announce (SCA)
- Current Request (16 bits): 
- Sources Available (8 bits): 
- Max Adjacent Sources (4 bits): 
- Reserved (3 bits): 
- ACK Required flag (1 bit):
- 0 or more invalid channel warnings
    - Invalid Channel ID (8 bits): 
    - Reserved (15 bits):  
    - C Flag (1 bit): 
    - Error Code (8 bits): 
        - 0x0: Undefined Error
        - 0x20: Invalid Request
        - 0x31: Stream Temporarily Unavailable: Insufficient sources
        - 0x32: Stream Temporarily Unavailable: Current source has no media of this format
        - 0x34: Stream Temporarily Unavailable: Insufficient network bandwidth
        - 0x40: Stream Source Change Avatar: The normal source for this stream has been replaced with an encoding of an avatar.
    - Capture Source ID (32 bit): 

# Sub-session Channel Announce Acknowledgement (SCA-ACK)
Just has header to fill Sequence Number for ACKed SCA's seq


# SDP
- params
    - multi-stream support capability
        - ```a=rtcp-fb:* ccm cisco-scr-v3``` 
        - support verison v3
    - Multi-source SDP advertisement
        - ```a=sprop-source:<source_id> <optional parameters>``` 
            - csi ```;csi=4049318657;``` 
            - policies  ```…;policies=as:1,rs:2;…  ```
            - count: how many sources send, in general media server(endpoints with multi-cameras) maybe > 1, and normal endpoint is 1
            - lrotation: ```…;lrotation=1;… ``` 
                - >The optional parameter lrotation signals support for requesting a landscape display aspect ratio (e.g. 4:3 or 16:9) of this source.  If not present then the source cannot be requested in a landscape aspect ratio.  The value of the parameter is either a 0 (not supported) or a 1 (supported).
            - simul ```;simul=1,2|1,3|1,4;```
                - this source can be simulcast in encoding 1 and either 2, 3 or 4
                - 2,2,3,4 are simulcast_id that will be define in sprop-simul
            - mavatar ```…;mavatar=1;… ```
                - >signals support for requesting an encoding of a video avatar in place of a live video feed should the live video feed become unavailable (e.g. the user mutes their video).  If not present then the source cannot generate encoded video based on an avatar.  The value of the parameter is either a 0 (not supported) or a 1 (supported)
        - example
            - endpoint
                - audio: ```a=sprop-source:0 csi=600599808;count=1```
                - video: ```a=sprop-source:0 csi=600599809;count=1;simul=100,101,102|103,104,105;lrotation=1;mavatar=1```
                - sharing ```a=sprop-source:0 csi=478737665;count=1;lrotation=1```
            - server
                - Audio: ```a=sprop-source:0 count=6;policies=as:1,al:2;lrotation=1;mavatar=1```
                - Video: 
                ```
                a=sprop-source:0 count=25;policies=as:1,al:2;lrotation=1;mavatar=1
                a=sprop-source:100 count=1;policies=rs:3;lrotation=0;mavatar=0
                ```
                - Sharing: ```a=sprop-source:0 count=1;policies=as:1,al:2;lrotation=1;mavatar=1```
    - Source encoding SDP advertisement
        - ```a=sprop-simul:<source_id> <simulcast_id> <format> <format specific parameters>```
        - example
            - endpoint
                - audio: ```a=sprop-simul:0 100 *```
                - video: 
                ```
                a=sprop-simul:0 100 117 profile-level-id=42e00c;max-mbps=7200;max-fs=396;max-fps=3000;
                a=sprop-simul:0 101 117 profile-level-id=42e014;max-mbps=27600;max-fs=920;max-fps=3000;
                a=sprop-simul:0 102 117 profile-level-id=42e014;max-mbps=108000;max-fs=3600;max-fps=3000;
                a=sprop-simul:0 103 97 profile-level-id=42e00c;max-mbps=7200;max-fs=396;max-fps=3000;
                a=sprop-simul:0 104 97 profile-level-id=42e014;max-mbps=27600;max-fs=920;max-fps=3000;
                a=sprop-simul:0 105 97 profile-level-id=42e014;max-mbps=108000;max-fs=3600;max-fps=3000;
                ```
                - sharing ```a=sprop-simul:0 100 * profile-level-id=420016;max-mbps=122400;max-fs=34560;fr-layers=500```
            - server
                - Audio: ```a=sprop-simul:0 0 *```
                - Video: 
                    ```
                    a=sprop-simul:0 0 *
                    a=sprop-simul:0 1 *
                    a=sprop-simul:0 2 *
                    a=sprop-simul:0 3 *
                    a=sprop-simul:100 100 *
                    a=sprop-simul:100 101 *
                    a=sprop-simul:100 102 *
                    ```
                - Sharing: ```a=sprop-simul:0 0 *```
    - advertise some additional encoder- and codec-independent totals for all transmitted encodings
        - ```a=sprop-total:max-pps=62668800```
- Example
    - Endpoint example (simulcast, local composition)
        ```
        m=audio 4298 RTP/AVP 113 108 0 15
        a=rtpmap:113 MP4A-LATM/90000
        a=fmtp:113 profile-level-id=24;object=23;bitrate=96000
        a=rtpmap:108 MP4A-LATM/90000
        a=fmtp:108 profile-level-id=24;object=23;bitrate=64000
        a=rtpmap:0 PCMU/8000/1
        a=rtpmap:15 G728/8000
        a=sprop-source:1 csi=15847168
        a=sprop-simul:1 100 *
        a=sprop-simul:1 101 *
        a=rtcp-fb:* ccm cisco-scr
        a=sendrecv

        m=video 4300 RTP/AVP 126 98
        b=AS:4000
        a=rtpmap:126 H264/90000
        a=fmtp:126 profile-level-id=42e016;max-mbps=244800;max-fs=8160;packetization-mode=1
        a=rtpmap:98 H264-SVC/90000
        a=fmtp:98 profile-level-id=42e016;max-mbps=244800;max-fs=8160;packetization-mode=1;mst-mode=NI-T;uc-mode=1
        a=sprop-source:1 csi=15847169;simul=100|101,102
        a=sprop-simul:1 100 98 profile-level-id=42e016;max-mbps=244800;max-fs=8160;fr-layers=750,1500,3000
        a=sprop-simul:1 101 98 profile-level-id=42e016;max-mbps=108000;max-fs=3600;fr-layers=750,1500,3000
        a=sprop-simul:1 102 98 profile-level-id=42e016;max-mbps=108000;max-fs=3600;fr-layers=750,1500,3000
        a=rtcp-fb:* ccm cisco-scr
        a=sendrecv
        ```
    - Conferencing device example (switching of multiple, independent audio and video streams)
        ```
        m=audio 6014 RTP/AVP 113 15
        a=rtpmap:113 MP4A-LATM/90000
        a=fmtp:113 profile-level-id=24;object=23;bitrate=96000
        a=rtpmap:15 G728/8000
        a=sprop-source:1 source-count=3;policies=as:10
        a=sprop-simul:1 1 *
        a=rtcp-fb:* ccm cisco-scr
        a=sendrecv
        m=video 6016 RTP/AVP 126 98
        b=AS:8000
        a=rtpmap:126 H264/90000
        a=fmtp:126 profile-level-id=42e016;max-mbps=244800;max-fs=8160;packetization-mode=1
        a=rtpmap:98 H264-SVC/90000
        a=fmtp:98 profile-level-id=42e016;max-mbps=244800;max-fs=8160;packetization-mode=1;mst-mode=NI-T;uc-mode=1
        a=sprop-source:1 policies=as:10,rs:11
        a=sprop-simul:1 1 98 profile-level-id=42e016;max-mbps=244800;max-fs=8160;fr-layers=1500,3000
        a=sprop-source:2 source-count=5; policies=as:10,rs:11
        a=sprop-simul:2 2 98 profile-level-id=42e016;max-mbps=108000;max-fs=3600;fr-layers=1500,3000
        a=rtcp-fb:* ccm cisco-scr
        a=sendrecv
        ```

    - H265-capable endpoint example (simulcast of H264 and H265)
        ```
        m=video 4300 RTP/AVP 126 98 104
        b=AS:4000
        a=rtpmap:126 H264/90000
        a=fmtp:126 profile-level-id=42e016;max-mbps=244800;max-fs=8160;packetization-mode=1
        a=rtpmap:98 H264-SVC/90000
        a=fmtp:98 profile-level-id=42e016;max-mbps=244800;max-fs=8160;packetization-mode=1;mst-mode=NI-T;uc-mode=1
        a=rtpmap:104 H265/90000
        a=fmtp:104 example-caps=1080p30;packetization-mode=1
        a=sprop-source:1 csi=15847169;simul=100|101,102
        a=sprop-simul:1 1 98 profile-level-id=42e016;max-mbps=244800;max-fs=8160;fr-layers=750,1500,3000
        a=sprop-simul:1 2 104 example-caps=1080p30
        a=sprop-total:max-pps=62668800
        a=rtcp-fb:* ccm cisco-scr
        a=sendrecv
        ```
    - WME
        ```
        o=wme-mac-10.11.4 0 1 IN IP4 127.0.0.1
        s=-
        c=IN IP4 127.0.0.1
        b=TIAS:13800000
        t=0 0
        a=cisco-mari:v1
        a=cisco-mari-rate
        a=cisco-mari-rtx:v0
        a=cisco-mari-hybrid-resilience:v0
        a=ice-options:cisco+tls+fqdn
        m=audio 52022 RTP/SAVPF 101 9 102 8 18 103 104 0 13 100 111
        c=IN IP4 10.79.98.48
        b=TIAS:600000
        a=content:main
        a=sendrecv
        a=rtpmap:101 opus/48000/2
        a=fmtp:101 maxplaybackrate=48000;maxaveragebitrate=64000;stereo=0
        a=rtpmap:9 G722/8000
        a=rtpmap:102 iLBC/8000
        a=rtpmap:8 PCMA/8000
        a=rtpmap:18 G729/8000
        a=fmtp:18 annexb=yes
        a=rtpmap:103 G7221/16000
        a=fmtp:103 bitrate=24000
        a=rtpmap:104 G7221/16000
        a=fmtp:104 bitrate=32000
        a=rtpmap:0 PCMU/8000
        a=rtpmap:13 CN/8000
        a=rtpmap:100 telephone-event/8000
        a=fmtp:100 0-15
        a=rtpmap:111 x-ulpfecuc/8000
        a=fmtp:111 max_esel=1400;max_n=255;m=8;multi_ssrc=1;FEC_ORDER=FEC_SRTP;non_seq=1;feedback=0
        a=rtcp-fb:* ccm cisco-asn
        a=extmap:1/sendrecv http://protocols.cisco.com/virtualid
        a=extmap:2/sendrecv urn:ietf:params:rtp-hdrext:ssrc-audio-level
        a=extmap:3/sendrecv urn:ietf:params:rtp-hdrext:toffset
        a=extmap:4/sendrecv http://protocols.cisco.com/timestamp#100us
        a=crypto:1 AEAD_AES_128_GCM inline: ****** WSH=1024
        a=crypto:2 AEAD_AES_256_GCM inline: ****** WSH=1024
        a=crypto:3 AES_CM_128_HMAC_SHA1_80 inline: ****** WSH=1024
        a=crypto:4 AES_CM_256_HMAC_SHA1_80 inline: ****** WSH=1024
        a=crypto:5 AES_CM_128_HMAC_SHA1_32 inline: ****** WSH=1024
        a=rtcp-mux
        a=sprop-source:0 csi=600599808;count=1
        a=sprop-simul:0 100 *
        a=rtcp-fb:* ccm cisco-scr
        a=ice-ufrag:KeLg
        a=ice-pwd: ******
        a=candidate:1 1 UDP 2113935615 10.79.98.48 52022 typ HOST
        a=candidate:1 2 UDP 2113935614 10.79.98.48 52023 typ HOST
        a=candidate:5 1 TCP 2113935359 10.79.98.48 52032 typ HOST
        a=candidate:5 2 TCP 2113935358 10.79.98.48 52033 typ HOST
        a=candidate:9 1 xTLS 2113935103 10.79.98.48 9 typ HOST
        a=candidate:9 2 xTLS 2113935102 10.79.98.48 9 typ HOST
        a=candidate:13 1 UDP 2113934847 192.168.2.3 52012 typ HOST
        a=candidate:13 2 UDP 2113934846 192.168.2.3 52013 typ HOST
        a=candidate:17 1 TCP 2113934591 192.168.2.3 52036 typ HOST
        a=candidate:17 2 TCP 2113934590 192.168.2.3 52037 typ HOST
        a=candidate:21 1 xTLS 2113934335 192.168.2.3 9 typ HOST
        a=candidate:21 2 xTLS 2113934334 192.168.2.3 9 typ HOST
        a=rtcp-xr:rcvr-rtt=all voip-metrics
        m=video 52122 RTP/SAVPF 117 97 111
        c=IN IP4 10.79.98.48
        b=TIAS:5000000
        a=content:main
        a=sendrecv
        a=rtpmap:117 H264/90000
        a=fmtp:117 profile-level-id=420014;packetization-mode=1;max-mbps=108000;max-fs=3600;max-fps=3000;max-br=1500;max-dpb=11520;level-asymmetry-allowed=1
        a=rtpmap:97 H264/90000
        a=fmtp:97 profile-level-id=420014;packetization-mode=0;max-mbps=108000;max-fs=3600;max-fps=3000;max-br=1500;max-dpb=11520;level-asymmetry-allowed=1
        a=rtpmap:111 x-ulpfecuc/8000
        a=fmtp:111 max_esel=1400;max_n=255;m=8;multi_ssrc=1;FEC_ORDER=FEC_SRTP;non_seq=1;feedback=0
        a=rtcp-fb:* nack pli
        a=rtcp-fb:* ccm tmmbr
        a=rtcp-fb:* ccm msync
        a=rtcp-fb:* ccm fir
        a=extmap:1/sendrecv http://protocols.cisco.com/virtualid
        a=extmap:2/sendrecv http://protocols.cisco.com/framemarking
        a=extmap:3/sendrecv urn:ietf:params:rtp-hdrext:toffset
        a=extmap:4/sendrecv http://protocols.cisco.com/timestamp#100us
        a=extmap:5/sendrecv http://protocols.cisco.com/av1
        a=crypto:1 AEAD_AES_128_GCM inline: ****** WSH=1024
        a=crypto:2 AEAD_AES_256_GCM inline: ****** WSH=1024
        a=crypto:3 AES_CM_128_HMAC_SHA1_80 inline: ****** WSH=1024
        a=crypto:4 AES_CM_256_HMAC_SHA1_80 inline: ****** WSH=1024
        a=crypto:5 AES_CM_128_HMAC_SHA1_32 inline: ****** WSH=1024
        a=rtcp-mux
        a=sprop-source:0 csi=600599809;count=1;simul=100,101,102|103,104,105;lrotation=1;mavatar=1
        a=sprop-simul:0 100 117 profile-level-id=42e00c;max-mbps=7200;max-fs=396;max-fps=3000;
        a=sprop-simul:0 101 117 profile-level-id=42e014;max-mbps=27600;max-fs=920;max-fps=3000;
        a=sprop-simul:0 102 117 profile-level-id=42e014;max-mbps=108000;max-fs=3600;max-fps=3000;
        a=sprop-simul:0 103 97 profile-level-id=42e00c;max-mbps=7200;max-fs=396;max-fps=3000;
        a=sprop-simul:0 104 97 profile-level-id=42e014;max-mbps=27600;max-fs=920;max-fps=3000;
        a=sprop-simul:0 105 97 profile-level-id=42e014;max-mbps=108000;max-fs=3600;max-fps=3000;
        a=rtcp-fb:* ccm cisco-scr
        a=ice-ufrag:z+jH
        a=ice-pwd: ******
        a=candidate:1 1 UDP 2113935615 10.79.98.48 52122 typ HOST
        a=candidate:1 2 UDP 2113935614 10.79.98.48 52123 typ HOST
        a=candidate:5 1 TCP 2113935359 10.79.98.48 52178 typ HOST
        a=candidate:5 2 TCP 2113935358 10.79.98.48 52179 typ HOST
        a=candidate:9 1 xTLS 2113935103 10.79.98.48 9 typ HOST
        a=candidate:9 2 xTLS 2113935102 10.79.98.48 9 typ HOST
        a=candidate:13 1 UDP 2113934847 192.168.2.3 52182 typ HOST
        a=candidate:13 2 UDP 2113934846 192.168.2.3 52183 typ HOST
        a=candidate:17 1 TCP 2113934591 192.168.2.3 52182 typ HOST
        a=candidate:17 2 TCP 2113934590 192.168.2.3 52183 typ HOST
        a=candidate:21 1 xTLS 2113934335 192.168.2.3 9 typ HOST
        a=candidate:21 2 xTLS 2113934334 192.168.2.3 9 typ HOST
        a=rtcp-xr:rcvr-rtt=all
        m=video 52198 RTP/SAVPF 117 97 111 112
        c=IN IP4 10.79.98.48
        b=TIAS:8000000
        a=content:slides
        a=sendrecv
        a=rtpmap:117 H264/90000
        a=fmtp:117 profile-level-id=420016;packetization-mode=1;max-mbps=122400;max-fs=34560;max-fps=3000;max-br=4000;max-dpb=51840;level-asymmetry-allowed=1
        a=rtpmap:97 H264/90000
        a=fmtp:97 profile-level-id=420016;packetization-mode=0;max-mbps=122400;max-fs=34560;max-fps=3000;max-br=4000;max-dpb=51840;level-asymmetry-allowed=1
        a=rtpmap:111 x-ulpfecuc/8000
        a=fmtp:111 max_esel=1400;max_n=255;m=8;multi_ssrc=1;FEC_ORDER=FEC_SRTP;non_seq=1;feedback=0
        a=rtpmap:112 mari-rtx/90000
        a=fmtp:112 RTX_ORDER=RTX_SRTP;rtx-time=3000
        a=rtcp-fb:* nack pli
        a=rtcp-fb:* ccm tmmbr
        a=rtcp-fb:* ccm msync
        a=rtcp-fb:* ccm fir
        a=rtcp-fb:* nack
        a=extmap:1/sendrecv http://protocols.cisco.com/virtualid
        a=extmap:2/sendrecv http://protocols.cisco.com/framemarking
        a=extmap:3/sendrecv urn:ietf:params:rtp-hdrext:toffset
        a=extmap:4/sendrecv http://protocols.cisco.com/timestamp#100us
        a=extmap:5/sendrecv http://protocols.cisco.com/av1
        a=crypto:1 AEAD_AES_128_GCM inline: ****** WSH=1024
        a=crypto:2 AEAD_AES_256_GCM inline: ****** WSH=1024
        a=crypto:3 AES_CM_128_HMAC_SHA1_80 inline: ****** WSH=1024
        a=crypto:4 AES_CM_256_HMAC_SHA1_80 inline: ****** WSH=1024
        a=crypto:5 AES_CM_128_HMAC_SHA1_32 inline: ****** WSH=1024
        a=rtcp-mux
        a=sprop-source:0 csi=478737665;count=1;lrotation=1
        a=sprop-simul:0 100 * profile-level-id=420016;max-mbps=122400;max-fs=34560;fr-layers=500
        a=rtcp-fb:* ccm cisco-scr
        a=ice-ufrag:zgWg
        a=ice-pwd: ******
        a=candidate:1 1 UDP 2113935615 10.79.98.48 52198 typ HOST
        a=candidate:1 2 UDP 2113935614 10.79.98.48 52199 typ HOST
        a=candidate:5 1 TCP 2113935359 10.79.98.48 52100 typ HOST
        a=candidate:5 2 TCP 2113935358 10.79.98.48 52101 typ HOST
        a=candidate:9 1 xTLS 2113935103 10.79.98.48 9 typ HOST
        a=candidate:9 2 xTLS 2113935102 10.79.98.48 9 typ HOST
        a=candidate:13 1 UDP 2113934847 192.168.2.3 52100 typ HOST
        a=candidate:13 2 UDP 2113934846 192.168.2.3 52101 typ HOST
        a=candidate:17 1 TCP 2113934591 192.168.2.3 52168 typ HOST
        a=candidate:17 2 TCP 2113934590 192.168.2.3 52169 typ HOST
        a=candidate:21 1 xTLS 2113934335 192.168.2.3 9 typ HOST
        a=candidate:21 2 xTLS 2113934334 192.168.2.3 9 typ HOST
        a=rtcp-xr:rcvr-rtt=all
        m=audio 52006 RTP/SAVPF 101 9 102 8 18 103 104 0 13 111
        c=IN IP4 10.79.98.48
        b=TIAS:200000
        a=content:slides
        a=sendrecv
        a=rtpmap:101 opus/48000/2
        a=fmtp:101 maxplaybackrate=48000;maxaveragebitrate=64000;stereo=0
        a=rtpmap:9 G722/8000
        a=rtpmap:102 iLBC/8000
        a=rtpmap:8 PCMA/8000
        a=rtpmap:18 G729/8000
        a=fmtp:18 annexb=yes
        a=rtpmap:103 G7221/16000
        a=fmtp:103 bitrate=24000
        a=rtpmap:104 G7221/16000
        a=fmtp:104 bitrate=32000
        a=rtpmap:0 PCMU/8000
        a=rtpmap:13 CN/8000
        a=rtpmap:111 x-ulpfecuc/8000
        a=fmtp:111 max_esel=1400;max_n=255;m=8;multi_ssrc=1;FEC_ORDER=FEC_SRTP;non_seq=1;feedback=0
        a=extmap:1/sendrecv http://protocols.cisco.com/virtualid
        a=extmap:2/sendrecv urn:ietf:params:rtp-hdrext:ssrc-audio-level
        a=extmap:3/sendrecv urn:ietf:params:rtp-hdrext:toffset
        a=extmap:4/sendrecv http://protocols.cisco.com/timestamp#100us
        a=crypto:1 AEAD_AES_128_GCM inline: ****** WSH=1024
        a=crypto:2 AEAD_AES_256_GCM inline: ****** WSH=1024
        a=crypto:3 AES_CM_128_HMAC_SHA1_80 inline: ****** WSH=1024
        a=crypto:4 AES_CM_256_HMAC_SHA1_80 inline: ****** WSH=1024
        a=crypto:5 AES_CM_128_HMAC_SHA1_32 inline: ****** WSH=1024
        a=rtcp-mux
        a=sprop-source:0 csi=478737664;count=1
        a=sprop-simul:0 100 *
        a=rtcp-fb:* ccm cisco-scr
        a=ice-ufrag:S33c
        a=ice-pwd: ******
        a=candidate:1 1 UDP 2113935615 10.79.98.48 52006 typ HOST
        a=candidate:1 2 UDP 2113935614 10.79.98.48 52007 typ HOST
        a=candidate:5 1 TCP 2113935359 10.79.98.48 52034 typ HOST
        a=candidate:5 2 TCP 2113935358 10.79.98.48 52035 typ HOST
        a=candidate:9 1 xTLS 2113935103 10.79.98.48 9 typ HOST
        a=candidate:9 2 xTLS 2113935102 10.79.98.48 9 typ HOST
        a=candidate:13 1 UDP 2113934847 192.168.2.3 52018 typ HOST
        a=candidate:13 2 UDP 2113934846 192.168.2.3 52019 typ HOST
        a=candidate:17 1 TCP 2113934591 192.168.2.3 52008 typ HOST
        a=candidate:17 2 TCP 2113934590 192.168.2.3 52009 typ HOST
        a=candidate:21 1 xTLS 2113934335 192.168.2.3 9 typ HOST
        a=candidate:21 2 xTLS 2113934334 192.168.2.3 9 typ HOST
        a=rtcp-xr:rcvr-rtt=all voip-metrics
        ```
    - media-server
    ```
        o=linus 0 1 IN IP4 10.79.38.25
        s=-
        c=IN IP4 10.79.38.25
        b=TIAS:13800000
        t=0 0
        a=ice-lite
        a=cisco-mari:v1
        a=cisco-mari-rate
        a=cisco-mari-rtx:v0
        m=audio 5004 RTP/SAVPF 98 9 8 0 127 101
        c=IN IP4 10.79.38.25
        b=TIAS:600000
        a=content:main
        a=sendrecv
        a=rtpmap:98 opus/48000/2
        a=fmtp:98 maxplaybackrate=48000;maxaveragebitrate=64000;stereo=1
        a=rtpmap:9 G722/8000
        a=rtpmap:8 PCMA/8000
        a=rtpmap:0 PCMU/8000
        a=rtpmap:127 x-ulpfecuc/8000
        a=fmtp:127 max_esel=1400;max_n=255;m=8;multi_ssrc=1;FEC_ORDER=FEC_SRTP;non_seq=1;feedback=0
        a=rtpmap:101 telephone-event/8000
        a=fmtp:101 0-15
        a=extmap:1/sendrecv http://protocols.cisco.com/virtualid
        a=extmap:3/sendrecv urn:ietf:params:rtp-hdrext:toffset
        a=extmap:4/sendrecv http://protocols.cisco.com/timestamp#100us
        a=crypto:1 AEAD_AES_128_GCM inline: ****** WSH=1024
        a=rtcp-mux
        a=label:100
        a=sprop-source:0 count=6;policies=as:1,al:2;lrotation=1;mavatar=1
        a=sprop-simul:0 0 *
        a=rtcp-fb:* ccm cisco-scr
        a=ice-ufrag:QKNRKT3a
        a=ice-pwd: ******
        a=candidate:0 1 UDP 2130706431 10.79.38.25 5004 typ host
        a=candidate:1 1 TCP 1962934271 10.79.38.25 5004 typ host tcptype passive
        m=video 5004 RTP/SAVPF 108 107 127
        c=IN IP4 10.79.38.25
        b=TIAS:5000000
        a=content:main
        a=sendrecv
        a=rtpmap:108 H264/90000
        a=fmtp:108 profile-level-id=420014;packetization-mode=1;max-mbps=108000;max-fs=3600;max-fps=3000;max-br=1500;max-dpb=891;level-asymmetry-allowed=1
        a=rtpmap:107 H264/90000
        a=fmtp:107 profile-level-id=420014;packetization-mode=0;max-mbps=108000;max-fs=3600;max-fps=3000;max-br=1500;max-dpb=891;level-asymmetry-allowed=1
        a=rtpmap:127 x-ulpfecuc/8000
        a=fmtp:127 max_esel=1400;max_n=255;m=8;multi_ssrc=1;FEC_ORDER=FEC_SRTP;non_seq=1;feedback=0
        a=rtcp-fb:* nack pli
        a=rtcp-fb:* ccm fir
        a=rtcp-fb:* ccm msync
        a=rtcp-fb:* ccm tmmbr
        a=extmap:1/sendrecv http://protocols.cisco.com/virtualid
        a=extmap:2/sendrecv http://protocols.cisco.com/framemarking
        a=extmap:3/sendrecv urn:ietf:params:rtp-hdrext:toffset
        a=extmap:4/sendrecv http://protocols.cisco.com/timestamp#100us
        a=crypto:1 AEAD_AES_128_GCM inline: ****** WSH=1024
        a=rtcp-mux
        a=label:200
        a=sprop-source:0 count=25;policies=as:1,al:2;lrotation=1;mavatar=1
        a=sprop-source:100 count=1;policies=rs:3;lrotation=0;mavatar=0
        a=sprop-simul:0 0 *
        a=sprop-simul:0 1 *
        a=sprop-simul:0 2 *
        a=sprop-simul:0 3 *
        a=sprop-simul:100 100 *
        a=sprop-simul:100 101 *
        a=sprop-simul:100 102 *
        a=rtcp-fb:* ccm cisco-scr
        a=ice-ufrag:gjVqd7Gq
        a=ice-pwd: ******
        a=candidate:0 1 UDP 2130706431 10.79.38.25 5004 typ host
        a=candidate:1 1 TCP 1962934271 10.79.38.25 5004 typ host tcptype passive
        m=video 5004 RTP/SAVPF 118 117 127 126
        c=IN IP4 10.79.38.25
        b=TIAS:8000000
        a=content:slides
        a=sendrecv
        a=rtpmap:118 H264/90000
        a=fmtp:118 profile-level-id=420016;packetization-mode=1;max-mbps=122400;max-fs=34560;max-fps=3000;max-br=4000;max-dpb=3037;level-asymmetry-allowed=1
        a=rtpmap:117 H264/90000
        a=fmtp:117 profile-level-id=420016;packetization-mode=0;max-mbps=122400;max-fs=34560;max-fps=3000;max-br=4000;max-dpb=3037;level-asymmetry-allowed=1
        a=rtpmap:127 x-ulpfecuc/8000
        a=fmtp:127 max_esel=1400;max_n=255;m=8;multi_ssrc=1;FEC_ORDER=FEC_SRTP;non_seq=1;feedback=0
        a=rtpmap:126 mari-rtx/90000
        a=fmtp:126 RTX_ORDER=RTX_SRTP;rtx-time=1000
        a=rtcp-fb:* nack pli
        a=rtcp-fb:* ccm fir
        a=rtcp-fb:* ccm msync
        a=rtcp-fb:* nack
        a=rtcp-fb:* ccm tmmbr
        a=extmap:1/sendrecv http://protocols.cisco.com/virtualid
        a=extmap:2/sendrecv http://protocols.cisco.com/framemarking
        a=extmap:3/sendrecv urn:ietf:params:rtp-hdrext:toffset
        a=extmap:4/sendrecv http://protocols.cisco.com/timestamp#100us
        a=crypto:1 AEAD_AES_128_GCM inline: ****** WSH=1024
        a=rtcp-mux
        a=label:400
        a=sprop-source:0 count=1;policies=as:1,al:2;lrotation=1;mavatar=1
        a=sprop-simul:0 0 *
        a=rtcp-fb:* ccm cisco-scr
        a=ice-ufrag:OOURueR1
        a=ice-pwd: ******
        a=candidate:0 1 UDP 2130706431 10.79.38.25 5004 typ host
        a=candidate:1 1 TCP 1962934271 10.79.38.25 5004 typ host tcptype passive
        m=audio 5004 RTP/SAVPF 102 127
        c=IN IP4 10.79.38.25
        b=TIAS:200000
        a=content:slides
        a=sendrecv
        a=rtpmap:102 opus/48000/2
        a=fmtp:102 maxplaybackrate=48000;maxaveragebitrate=64000;stereo=1
        a=rtpmap:127 x-ulpfecuc/8000
        a=fmtp:127 max_esel=1400;max_n=255;m=8;multi_ssrc=1;FEC_ORDER=FEC_SRTP;non_seq=1;feedback=0
        a=extmap:1/sendrecv http://protocols.cisco.com/virtualid
        a=extmap:3/sendrecv urn:ietf:params:rtp-hdrext:toffset
        a=extmap:4/sendrecv http://protocols.cisco.com/timestamp#100us
        a=crypto:1 AEAD_AES_128_GCM inline: ****** WSH=1024
        a=rtcp-mux
        a=label:300
        a=sprop-source:0 count=1;policies=as:1,al:2;lrotation=1;mavatar=1
        a=sprop-simul:0 0 *
        a=rtcp-fb:* ccm cisco-scr
        a=ice-ufrag:8bGMLQiB
        a=ice-pwd: ******
        a=candidate:0 1 UDP 2130706431 10.79.38.25 5004 typ host
        a=candidate:1 1 TCP 1962934271 10.79.38.25 5004 typ host tcptype passive
    ```

参考
================
[CiscoLive: multi-stream](https://www.ciscolive.com/c/dam/r/ciscolive/us/docs/2016/pdf/BRKUCC-2325.pdf)