---
date: 2018-09-18T21:07:13+08:00
title: 已发布版本
weight: 112
menu:
  main:
    parent: "introduction"
keywords:
- cilium版本信息
- cilium最新版本
- cilium release note
description : "收集Cilium各个发布版本的信息和Release Note"
---

## v1.2

发布于2018-09-21。

官方博客文章和中文翻译：

- [Cilium 1.2: DNS Security Policies, EKS Support, ClusterMesh, kube-router integration, ...](https://cilium.io/blog/2018/08/21/cilium-12)
- [Cilium 1.2发布—DNS安全性策略、EKS支持、ClusterMesh、kube-router集成等](http://www.servicemesher.com/blog/cilium1.2-dns-security-policies-eks-support-clustermesh-kube-router-integration/)

该版本引入了几个新功能实现了Cilium用户和社区成员最迫切想要的功能。

1. 其中最吸引人的功能之一是引入基于DNS 名称的安全策略，目的是保护对集群外服务的访问。
2. 另一个最受关注的问题是加入了连接和保护多个Kubernetes集群的能力。
	- 我们将ClusterMesh功能进入Alpha版本。它可以连接和保护在多个Kubernetes集群中运行的pod。
	- Kube-router与Cilium的集成同等重要。DigitalOcean团队的努力使kube-router提供BGP网络与Cilium提供的基于BPF的安全性和负载均衡相结合。

1.2版本的重要功能：

- 基于DNS/FQDN的安全策略
	- 基于FQDN/DNS命名定义网络安全规则，表示允许连接到外部服务。例如，允许访问foo.com。（处于Beta版）
- 支持AWS EKS
	- 为管理Kubernetes集成量身定制的etcd operator，消除了对需要外部kvstore的依赖。（处于Beta版）
- Clustermesh（集群间连接）
	- 跨多个Kubernetes集群的pod间网络连接。（处于Alpha版）
	- 跨集群的基于label的网络安全策略实施，例如允许cluster1中的pod foo与cluster2中的pod bar通信。
- 为支持BPF集成Kube-route
	- 与kube-router一起协作运行以启用BGP网络。
- 基于节点发现的KV存储
	- 在非Kubernetes环境中启用自动节点发现。
- 负载均衡
	- 支持一致的后端selection用于服务后端扩缩容
	- 支持基于服务label/name的策略以及L4规则
- 高效性 & 扩缩容
	- 对于大型和多集群规模的环境，安全身份认证空间从16bits增加到24bits。
	- 首次实现基于BPF的数据路径通知聚合。
	- 取得持续高效的CPU利用进展。
	- 自动检测underlying网络的MTU。
	- 禁用DSR时使用本地service ID分配以提高负载均衡可伸缩性。
- 文档
	- 新的AWS EKS安装指南。
	- 参考kubespray安装指南。
	- 新的简化安装和升级说明。