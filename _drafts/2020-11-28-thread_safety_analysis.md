---
layout: post
title: "线程安全分析"
date: 2020-11-28 00:01:00 -0000
categories: base
tag: [c++,thread]
---
>在多线程编程，线程安全总是容易出问题的地方。即便有经验的程序员也容易犯错。一般原型代码没问题，以后增强功能时候一犯错。提前在编译时候发现问题，总是比运行时发现好，而且大部分时候是潜伏在那，问题未必暴露。

介绍
=========

概念
=========


示例应用 WebRTC使用
==========


参考
=========
[clang ThreadSafetyAnalysis](https://opensource.apple.com/source/clang/clang-703.0.31/src/tools/clang/docs/ThreadSafetyAnalysis.rst.auto.html)
[gperftools]https://github.com/gperftools/gperftools