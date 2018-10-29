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

## v1.3

发布于2018-10-23。

官方博客文章：

- [Cilium 1.3: Go extensions for Envoy, Cassandra & Memcached Support](https://cilium.io/blog/2018/10/23/cilium-13-envoy-go/)

1.3 Release Highlights:

- **Go extensions for Envoy**
	- Exciting new extension API for Envoy using Go including a generic configuration and access logging API. (Beta)
- **Cassandra & Memcached protocol support**
	- New protocol parsers for Cassandra and Memcached implemented using the new Envoy Go extensions. Both parsers provide visibility and security policy enforcement on operation type and key/table names using exact matches, prefix matches, and regular expressions. (Beta)
- **Security**
	- TTLs support for DNS/FQDN policy rules
	- Introduction of well-known identities for kube-dns, coredns, and etcd-operator.
	- New security identity "unmanaged" to represent pods which are not managed by Cilium.
	- Improved security entity "cluster" which allows defining policies for all pods in a cluster (managed, unmanaged and host networking).
- **Additional Metrics & Monitoring**
	- New "cilium metrics list" command to list metrics via CLI.
	- Lots of additional metrics: connection tracking garbage collection, Kubernetes resource events, IPAM, endpoint regenerations, services, and error and warning counters.
	- New monitoring API with more efficient encoding/decoding protocol. Used by default with fallback for older clients.
- **Networking Improvements**
	- Split of connection tracking tables into TCP and non-TCP to better handle the mix of long and short-lived nature of each protocol.
	- Ability to specify the size of the connection tracking tables via ConfigMap.
	- Better masquerading behavior for traffic via NodePort and HostPort to allow pods to see the original source IP if possible.
- **Full Key-value store Resiliency**
	- Introduced ability to re-construct the kvstore contents immediately after loss of any state. Allows to restore etcd from backup or to completely wipe it for a running cluster with minimal impact. (Beta)
- **Efficiency & Scale**
	- Significant improvements in the cost of calculating policy of individual endpoints. Work continues on this subject.
	- New grace period when workloads change identity to minimize connectivity impact throughout identity change.
	- More efficient security identity allocation algorithm.
	- New generic framework to detect and ignore Kubernetes event notifications for which Cilium does not need to take action.
	- Improvements in avoiding unnecessary BPF compilations to reduce the CPU overhead caused by it. Initial work to scope BPF templating to avoid compilation altogether.
- **Kubernetes**
	- Added support for Kubernetes 1.12
	- Custom columns for the CiliumEndpoints CRD (Requires Kubernetes 1.11)
	- Removed cgo dependency from cilium-cni for compatibility with ulibc
	- Removed support for Kubernetes 1.7
- **Documentation**
	- New Ubuntu 18.04 guide
	- Coverage of latest BPF runtime features such as BTF (BPF Type Format).
	- Documentation for VM/host firewall requirements to run multi-host networking.
- **Long Term Stable (LTS) Release**
	- 1.3 has been declared an LTS release and will be supported for the next 6 months with backports.

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