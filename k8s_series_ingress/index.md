# Kubernetes系列：Ingress


## 系列目录

[《Kubernetes系列：开篇》]()

[《Kubernetes系列：概述》]()

[《Kubernetes系列：架构》]()

[《Kubernetes系列：CRI》]()

[《Kubernetes系列：CNI》]()

[《Kubernetes系列：CSI》]()

[《Kubernetes系列：Service》]()

[《Kubernetes系列：Ingress》]()

[《Kubernetes系列：OAM》]()

***

## 1. 介绍

### 1.1 Ingress

> [Ingress](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.20/#ingress-v1beta1-networking-k8s-io) 公开了从集群外部到集群内[service](https://kubernetes.io/zh/docs/concepts/services-networking/service/)的 HTTP 和 HTTPS 路由。 流量路由由 Ingress 资源上定义的规则控制。

![](ingress-process.jpg)

可以将 Ingress 配置为服务提供外部可访问的 URL、负载均衡流量、终止 SSL/TLS，以及提供基于名称的虚拟主机等能力。 [Ingress Controller](https://kubernetes.io/zh/docs/concepts/services-networking/ingress-controllers) 通常负责通过负载均衡器来实现 Ingress，尽管它也可以配置边缘路由器或其他前端来帮助处理流量。

***

### 1.2 Ingress Controller

为了让 Ingress 资源工作，集群必须有一个正在运行的 Ingress Controller。

***

## 2. Ingress Contoller 选择









***


