---
date: 2018-08-09T21:07:13+08:00
title: 介绍
weight: 100
---

![](images/cilium-logo.svg)

## Cilium是什么？

Cilium是一个开源软件，用于透明地提供和保护使用Kubernetes，Docker和Mesos等Linux容器管理平台部署的应用程序服务之间的网络和API连接。

![](images/cilium-connectivity.jpg)

Cilium基于一种名为BPF的新Linux内核技术，它可以在Linux内部动态插入强大的安全性，可见性和网络控制逻辑。 除了提供传统的网络级安全性之外，BPF的灵活性还可以在API和进程级别上实现安全性，以保护容器或容器内的通信。由于BPF在Linux内核中运行，因此可以应用和更新Cilium安全策略，而无需对应用程序代码或容器配置进行任何更改。

![](images/cilium-ecosystem.jpg)

## 为什么用Cilium？

现代数据中心应用程序的开发已经转向面向服务的体系结构，通常称为**微服务**，其中大型应用程序被分成小型独立服务，这些服务使用HTTP等轻量级协议通过API相互通信。微服务应用程序往往是高度动态的，使用单个容器，随着应用程序扩展/收缩而启动或销毁，以适应负载变化，以及滚动更新，作为持续交付的一部分而部署。

这种向高度动态微服务的转变在保护微服务之间的连接方面提出了挑战和机遇。传统的Linux网络安全方法（例如，iptables）过滤IP地址和TCP/UDP端口，但IP地址在动态微服务环境中经常流失。容器高度不稳定的生命周期导致这些方法难以与应用程序并排扩展，因为负载平衡表和访问控制列表包含需要以不断增长的频率更新的数十万条规则。出于安全目的，协议端口（例如，用于HTTP流量的TCP端口80）不再能够用于区分应用流量，因为端口用于跨服务的各种消息。

另一个挑战是提供准确可见性的能力，因为传统系统使用IP地址作为主要识别工具，在微服务架构中可能大大缩短，仅仅几秒钟的寿命。

通过利用Linux BPF，Cilium保留了透明地插入安全可视性+强制执行的能力，但这种方式基于service/pod/container标识（与传统系统中的IP地址识别相反），并且可以在应用层（例如HTTP）进行过滤。因此，通过将安全性与寻址分离，Cilium不仅可以在高度动态的环境中应用安全策略，而且除了提供传统的第3层和第4层分割之外，还可以通过在HTTP层操作来提供更强的安全隔离。。

BPF的使用使Cilium能够以高度可扩展的方式实现所有这一切，即使对于大规模环境也是如此。

## 功能概述

### 透明地保护和加密API

能够保护现代应用程序协议，如REST/HTTP，gRPC和Kafka。传统防火墙在第3层和第4层运行。在特定端口上运行的协议要么完全受信任，要么完全被阻止。Cilium提供了过滤各个应用程序协议请求的功能，例如：

- 允许所有使用方法`GET`和路径`/public/.*`的HTTP请求。拒绝所有其他请求。
- 允许`service1`在Kafka主题`topic1`生成和`service2`在`topic1`上消费。拒绝所有其他Kafka消息。
- 要求HTTP标头`X-Token：[0-9] +`出现在所有REST调用中。

### 基于身份的服务间安全通信

现代分布式应用程序依赖于诸如应用程序容器之类的技术来促进部署中的敏捷和按需扩展。这导致在短时间内启动大量应用容器。典型的容器防火墙通过过滤源IP地址和目标端口来保护工作负载。这个概念要求在集群中的任何位置启动容器时都要操作所有服务器上的防火墙。

为了避免这种限制扩展的情况，Cilium为共享相同安全策略的应用程序容器组分配安全标识。然后，该标识与应用程序容器发出的所有网络数据包相关联，从而允许在接收节点处验证身份。使用键值存储执行安全身份管理。

### 安全访问外部服务

基于标签的安全性是集群内部访问控制的首选工具。为了保护对外部服务的访问，支持传统基于CIDR的入口和出口的安全策略。这允许将对应用程序容器的访问限制在特定IP范围内。

### 简单网络

一个简单的扁平3层网络能够跨越多个集群连接所有应用程序容器。使用主机范围分配器可以简化IP分配。这意味着每个主机可以在主机之间没有任何协调的情况下分配IP。

支持以下多节点网络模型：

- **Overlay:** 基于封装的虚拟网络，产生所有主机。目前VXLAN和Geneve已经完成，但可以启用Linux支持的所有封装格式。

	何时使用此模式：此模式具有最小的基础设施和集成要求。它几乎适用于任何网络基础设施，因为唯一的要求是主机之间的IP连接，这通常已经给出。

- **Native Routing:** 使用Linux主机的常规路由表。网络必须能够路由应用程序容器的IP地址。

	何时使用此模式：此模式适用于高级用户，需要了解底层网络基础结构。此模式适用于：

	- 原生 IPv6 网络
	- 与云网络路由器配合使用
	- 如果您已经在运行路由守护进程

### 负载均衡

应用程序容器之间和到外部服务的流量分布式负载均衡。负载均衡使用BPF，使用高效hashtables，允许几乎无限制的规模，并且如果未在源主机上执行负载均衡操作，则支持直接服务器返回（DSR）。

### 监控和故障排除

获得可见性和解决问题的能力是任何分布式系统运维的基础。虽然我们学会了诸如tcpdump和ping这样的工具，虽然他们总会在我们心中找到一个特殊的地方，但我们会努力为故障排除提供更好的工具。这包括提供以下工具：

- 使用元数据进行事件监视：当丢弃数据包时，该工具不仅报告数据包的源IP和目标IP，该工具还提供发送方和接收方的完整标签信息以及许多其他信息。
- 策略决策跟踪：为什么丢弃数据包或拒绝请求。策略跟踪框架允许跟踪策略决策过程，包括运行中的工作负载和基于任意标签的定义。
- 通过Prometheus导出指标：通过Prometheus导出关键指标，以便与现有仪表板集成。

### 集成

- 网络插件集成: [CNI](https://github.com/containernetworking/cni), [libnetwork](https://github.com/docker/libnetwork)
- 容器运行时事件: [containerd](https://github.com/containerd/containerd)
- Kubernetes: [NetworkPolicy](https://kubernetes.io/docs/concepts/services-networking/network-policies/), [Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/), [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/), [Service](https://kubernetes.io/docs/concepts/services-networking/service/)
- 日志: syslog, [fluentd](http://www.fluentd.org/)

> 备注: 以上内容翻译自cilium官方文档 [Introduction to Cilium](https://cilium.readthedocs.io/en/v1.2/intro/#intro)