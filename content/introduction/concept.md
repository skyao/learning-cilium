---
date: 2018-09-18T21:07:13+08:00
title: 概念
weight: 102
menu:
  main:
    parent: "introduction"
---

本文档的目标是描述Cilium架构的组件，以及在数据中心或云环境中部署Cilium的不同模型。 它侧重于运行完整的Cilium部署所需的更高级别的理解。然后，您可以使用更详细的安装指南来了解设置Cilium的详细信息。

## 组件概况

![](images/cilium-arch.png)

Cilium的部署包括以下组件，运行在容器集群中的每个Linux容器节点上：

- **Cilium Agent (Daemon):** 用户空间守护程序，通过插件与容器运行时和编排系统（如Kubernetes）交互，以便为在本地服务器上运行的容器设置网络和安全性。提供用于配置网络安全策略，提取网络可见性数据等的API。
- **Cilium CLI Client:** 用于与本地Cilium Agent通信的简单CLI客户端，例如，用于配置网络安全性或可见性策略。
- **Linux Kernel BPF:** 集成Linux内核的功能，用于接受内核中在各种钩子/跟踪点运行的已编译字节码。Cilium编译BPF程序，并让内核在网络堆栈的关键点运行它们，以便可以查看和控制进出所有容器中的所有网络流量。
- **容器平台网络插件:** 每个容器平台（例如，Docker，Kubernetes）都有自己的插件模型，用于外部网络平台集成。对于Docker，每个Linux节点都运行一个进程（cilium-docker）来处理每个Docker libnetwork调用，并将数据/请求传递给主要的Cilium Agent。

除了在每个Linux容器主机上运行的组件之外，Cilium还利用键值存储在不同节点上运行的Cilium代理之间共享数据。目前支持的键值存储是：

- etcd
- consul


### Cilium Agent

Cilium代理（cilium-agent）在每个Linux容器主机上运行。 在高级别，代理接受描述服务级别网络安全性和可见性策略的配置。然后，它会监听容器运行时中的事件，以了解容器何时启动或停止，并创建自定义BPF程序，Linux内核使用这些程序来控制进出这些容器的所有网络访问。更详细地说，代理：

- 公开API以允许运维/安全团队配置安全策略（见下文）来控制集群中容器之间的所有通信。这些API还公开了监视功能，以进一步获取网络转发和过滤行为的可见性。
- 收集有关每个被创建的新容器的元数据。特别是，它查询的身份元数据，如容器/pod标签，这些会用于标识Cilium安全策略中的Endpoint。
- 与容器平台网络插件交互以执行IP地址管理（IPAM），IPAM控制为每个容器分配IPv4和IPv6地址。IPAM由代理管理，代理在所有插件之间的共享池中，这意味着Docker和CNI网络插件可以并行分配单个地址池。
- 结合容器标识和地址的知识与已配置的安全性和可见性策略，生成高效的BPF程序，这些程序适用于每个容器的网络转发和安全行为。
- 使用[clang/LLVM](https://clang.llvm.org/)将BPF程序编译为字节码，并将它们传递给Linux内核，以便为进出容器虚拟以太网设备中的所有数据包而运行

### Cilium CLI Client

Cilium CLI Client（cilium）是一个与Cilium Agent一起安装的命令行工具。它提供了一个命令行界面，可与Cilium Agent API的所有方面进行交互。包括检查有关每个网络端点（如容器）的Cilium状态，配置和查看安全策略以及配置网络监视行为。

### Linux Kernel BPF

Berkeley Packet Filter （BPF）是最初用于过滤网络数据包的Linux内核字节码解释器，例如， tcpdump和套接字过滤器。它已经扩展了其他数据结构，如hashtable和数组，以及支持数据包修改，转发，封装等的其他操作。内核验证程序确保BPF程序可以安全运行，然后JIT编译器转换字节码到CPU体系结构特定指令，以求原生执行效率。BPF程序可以在内核中的各个钩子点运行，例如传入数据包，传出数据包，系统调用，kprobes等。

每个新的Linux版本，BPF都在不断发展并获得更多功能。Cilium利用BPF执行核心数据路径过滤，修改，监控和重定向，并且需要Linux内核版本4.8.0或更高版本的BPF功能。在4.8.x已被宣布为生命周期结束且4.9.x被提名为稳定版本的基础上，我们建议至少运行内核4.9.17（截至本文撰写时，最新的当前稳定Linux内核为4.10.x）。

Cilium能够探测Linux内核的可用功能，并在检测到它们时自动使用更新的功能。

专注于成为容器运行时的Linux发行版（例如，CoreOS，Fedora Atomic）通常已经发布了比4.8更新的内核，但即使是最新版本的通用操作系统（如Ubuntu 16.10）也提供了相当新的内核。一些Linux发行版仍然发布较旧的内核，但其中许多允许从单独的内核包仓库中安装最新的内核。

### Key-Value Store

键值（KV）存储用于以下状态：

- 策略标识：标签列表<=>策略标识标识符
- 全局服务：VIP协会的全局服务ID（可选）
- 封装VTEP映射（可选）

为了简化更大部署中的内容，可以与容器编排器使用相同的键值存储（例如，Kubernetes使用etcd）。

### 保证

如果Cilium失去与KV-Store的连接，它可以保证：

- 正常的网络操作将继续;
- 如果启用了策略实施，现有端点仍将强制执行其策略，但您将无法添加其他容器，这些容器属于安全身份，但是在节点上无法获知;
- 如果启用了服务，您将无法添加其他服务/负载均衡器;
- 当连接恢复到KV-Store时，Cilium最多可能需要5分钟才能与KV-Store重新同步不同步的状态。

即使与KV-Store不同步，Cilium也会继续运行。

如果Cilium崩溃/或DaemonSet被意外删除，则有以下保证：

- 当使用标准安装指南文档中提供的规范文件运行Cilium作为DaemonSet/容器时，已运行的端点/容器不会损失任何连接，并且它们将使用在Cilium意外停止之前加载的策略继续运行。
- 以不同的方式运行Cilium时，只需确保bpf fs被装载 [Mounting the BPF FS](https://cilium.readthedocs.io/en/v1.2/kubernetes/install/standard/#admin-mount-bpffs)。

## 术语

TBD

> 备注：以上内容来自cilium官方文档 [Concepts](https://cilium.readthedocs.io/en/v1.2/concepts/)