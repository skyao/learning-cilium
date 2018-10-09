---
date: 2018-09-08T21:07:13+08:00
title: 前言
weight: 1
keywords:
- cilium
- BPF
- eBPF
- 学习笔记
description : "介绍Cilium学习笔记的基本资料和访问方式"
---

![](introduction/images/cilium-logo.svg)

### 内容介绍

Cilium是一个开源软件，用于透明地提供和保护使用Kubernetes，Docker和Mesos等Linux容器管理平台部署的应用程序服务之间的网络和API连接。

![](introduction/images/cilium-connectivity.jpg)

Cilium基于一种名为BPF的新Linux内核技术，它可以在Linux内部动态插入强大的安全性，可见性和网络控制逻辑。 除了提供传统的网络级安全性之外，BPF的灵活性还可以在API和进程级别上实现安全性，以保护容器或容器内的通信。由于BPF在Linux内核中运行，因此可以应用和更新Cilium安全策略，而无需对应用程序代码或容器配置进行任何更改。

### 访问方式

这是个人学习Cilium的笔记，请点击下面的链接阅读:

- [在线阅读](https://skyao.io/learning-cilium/)：hugo格式，界面清爽。托管于腾讯云香港节点，速度快，偶尔抽风
- [@github](https://github.com/skyao/learning-cilium/)：源码托管于github，如有谬误或需讨论，请提issue，欢迎提交PR

### 版权申明

本笔记内容可以任意转载，但请注明来源并提供链接，**请勿用于商业出版**。


