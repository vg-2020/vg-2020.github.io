---
layout: post
title: "WebRTC代码: sigslot"
date: 2020-11-23 03:00:00 -0000
categories: webrtc
tag: [tech, webrtc, sigslot]
---
>很久之前在嵌入设备写电子书阅读器时候用到QT的sigslot。那个时候的目的是把UI和逻辑分离。
读WebRTC代码时候，重新看到，有点熟悉的感觉。写个总结

sigslot是什么
===============

sigslot 其实是一个observer模式的实现。目的是让模块间减少依赖。
2个主要概念
- signal信号: 事件产生时候发出信号。
- slot槽: 接收事件，做相应的响应。

sigslot怎么实现的
===============

四个角色参与
- 消息signal: ```sigslot::signal？```
    - 维护一张slot表 item = {slot, solt::handler}
    - 事件发生后调用每个item 的handler
    - 自己析构时候把自己从所有的slot的signal表删除. 
- 处理slot: ```sigslot::has_slots<>```
    - 维护一张signal表 item = {signal}
    - 自己析构时候把自己从所有的signal的slot表删除. 
- 消息发送者: 拥有signal, 当某事件发生时候,发送消息
- 消息处理者: 接收消息，并做对应的响应。

sigslot线程
===============
- ```sigslot::single_threaded<>```

```c++
class single_threaded {
 public:
  void lock() {}
  void unlock() {}
};
```
- ```sigslot::multi_threaded_local```
    - 每个signal 自带一个锁
    - 如果signal很多的时候，有资源问题
- ```multi_threaded_global```
    - 所有signal公用一个全局锁
    - 如果signal很多时候， 有性能问题

sigslot使用
===============
- 定义signal
- 实现slot
- 连接signal和slot
- 发消息

example code >>
```c++
template <class signal_policy = sigslot::single_threaded>
class SignalProducer {
public:
    sigslot::signal1<const char*, signal_policy> signal_;
};

template <class slot_policy = sigslot::single_threaded>
    class SignalHandler : public sigslot::has_slots<slot_policy> {
    public:
        SignalHandler(const char* name):name_(name) {}
        void Handler(const char* name) {
            if (name && name == name_) {
                std::cout << "I am " << name << "! <raise hand> this=" << this << std::endl;
            }
            else {
                std::cout << "NA this=" << this << std::endl;
            }
        }
        std::string name_;
};
int main()
{
    //define slots
    SignalHandler<> signal_handler1("jack");
    SignalHandler<> signal_handler2("tom");

    //define signal
    SignalProducer<> signal_producer;

    //bind signal and handler
    signal_producer.signal_.connect(&signal_handler1, &SignalHandler<>::Handler);
    signal_producer.signal_.connect(&signal_handler2, &SignalHandler<>::Handler);

    //emit signal
    std::cout << "fire signal...." << std::endl;
    signal_producer.signal_("jack");
    std::cout << "fire signal...." << std::endl;
    signal_producer.signal_("tom");

    //disconnect
    signal_producer.signal_.disconnect(&signal_handler1);
    signal_producer.signal_.disconnect(&signal_handler2);

    std::cout << "fire signal...." << std::endl;
    signal_producer.signal_("jack");
    std::cout << "fire signal...." << std::endl;
    signal_producer.signal_("tom");
}
```

output>>
```bash
fire signal....
I am jack! <raise hand> this=0116FC24
NA this=0116FBE4
fire signal....
NA this=0116FC24
I am tom! <raise hand> this=0116FBE4
fire signal....
fire signal....
```


参考
===============
- webrtc code 
    - [sigslot.h](https://webrtc.googlesource.com/src/+/refs/heads/master/rtc_base/third_party/sigslot/sigslot.h)
    - [sigslot_unittest.cc](https://webrtc.googlesource.com/src/+/refs/heads/master/rtc_base/sigslot_unittest.cc)
- http://sigslot.sourceforge.net/