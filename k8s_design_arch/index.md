# K8S设计系列：架构


## 系列目录

[《K8S设计系列：开篇》](../k8s_design)

[《K8S设计系列：架构》](../k8s_design_arch)

***

## 1. 介绍

本篇要介绍的是k8s的整体架构，以及为什么要这么设计。

> Kubernetes是生产级的开源基础架构，用于跨主机集群部署，扩展，管理和组合应用程序容器。但是，Kubernetes不仅仅是一个“容器编排程序”，它旨在消除协调物理/虚拟计算，网络和存储基础架构的负担，并使应用程序运营商和开发人员完全专注于以容器为中心的自助服务操作原语。Kubernetes还提供了一个稳定，可移植的基础（平台），用于构建自定义工作流和更高级别的自动化。

Kubernetes主要针对由多个容器组成的应用程序。因此，它使用吊舱和标签将容器分组为紧密耦合和松散耦合的形式，以便于管理和发现。

***

### 1.1 范围

Kubernetes提供了容器运行时，容器编排，以容器为中心的基础架构编排，自我修复机制（例如运行状况检查和重新调度）以及服务发现和负载平衡。

Kubernetes渴望成为一个可扩展的，可插入的，构建模块的OSS平台和工具包。因此，在架构上，我们希望将Kubernetes构建为可插入组件和层的集合，能够使用其他调度程序，控制器，存储系统和分发机制，并且我们正在朝着这个方向发展其当前代码。此外，我们希望其他人能够在不修改核心Kubernetes源代码的情况下扩展Kubernetes功能，例如具有更高级别的PaaS功能或多集群层。因此，其API不仅（或者甚至主要是主要）针对最终用户，还针对工具和扩展开发人员。其API旨在作为工具，自动化系统和更高级别API层的开放生态系统的基础。因此，没有“内部”组件间API。所有API都是可见且可用的，包括调度程序，节点控制器，复制控制器管理器，Kubelet的API等所使用的API。没有困难可言-为了处理更复杂的用例，人们可以以完全透明，可组合的方式访问较低级别的API。

***

### 1.2 目标

该项目致力于以下（理想的）设计理想：

* *Portable*，Kubernetes以一致的行为在任何地方运行-公共云，私有云，裸机，笔记本电脑。这样，应用程序和工具就可以在整个生态系统以及开发和生产环境之间移植。
* *General-purpose*，Kubernetes应该运行所有主要类别的工作负载，以使您可以在单个基础架构，无状态和有状态，微服务和整体，服务和批处理，未开发的环境和遗留的基础架构上运行所有工作负载。
* *Meet users partway*，Kubernetes不仅迎合纯天然云原生应用程序，也需要满足所有用户的需求。它着重于微服务和云原生应用程序的部署和管理，但提供了一些机制来促进整体和遗留应用程序的迁移。
* *Flexible*，可以单点使用Kubernetes功能，并且（在大多数情况下）Kubernetes不会阻止您使用自己的解决方案来代替内置功能。
* *Extensible*，Kubernetes使您可以通过暴露内置功能所使用的相同接口，将其集成到您的环境中并添加所需的其他功能。
* *Automatable*，Kubernetes旨在极大地减少手动操作的负担。它通过通过其API指定用户期望的意图来支持声明式控制，以及支持更高级别的编排和自动化的命令式控制。声明式方法是系统自我修复和自主功能的关键。
* *Advance the state of the art*，尽管Kubernetes打算支持非云原生应用程序，但它也希望推动云原生和DevOps技术的发展，例如让应用程序参与其自身的管理。但是，这样做时，我们努力不强迫应用程序将自己锁定在Kubernetes API中，这就是例如为什么我们偏爱向下API中的配置优先于约定的原因。此外，Kubernetes不受它所依赖的系统的最低公分母的约束，例如容器运行时和云提供商。我们突破可实现范围的一个例子是其IP Pod网络模型。

***

## 2. 分层

kubernetes将自己的架构设计分为下面几层：

1. **Nucleus** ，提供标准化的API和执行机制，包括基本的REST机制，安全性，单个Pod，容器，网络接口和存储卷管理，所有这些都可以通过定义明确的接口进行扩展。Nucleus是非可选的，有望成为系统中最稳定的部分。
2. **Application Management Layer**，应用程序管理层提供基本的部署和路由，包括自我修复，扩展，服务发现，负载平衡和流量路由。这通常称为编排和服务结构。提供了所有功能的默认实现，但允许进行兼容的替换。
3. **Governance Layer**，它提供了更高级别的自动化和策略实施，包括单租户和多租户，指标，智能自动扩展和配置以及授权，配额，网络和存储策略表达与实施的方案。
4. **Interface Layer**，它提供了用于与Kubernetes API交互的常用库，工具，UI和系统。
5. **Ecosystem**，其中包括与Kubernetes相关的所有其他内容，并且实际上根本不是Kubernetes的“一部分”。这是大多数开发发生的地方，并且包括CI / CD，中间件，日志记录，监视，数据处理，PaaS，无服务器/ FaaS系统，工作流，容器运行时，映像注册表，节点和云提供商管理以及许多其他功能。

![](arch-roadmap-1.png)

***

### 2.1 Nucleus: API and Execution

基本的API和执行机制。

由上游Kubernetes代码库实现的这些API和功能，包括构建系统更高层所需的最少的功能和概念集。这些部分已得到详细说明和记录，每个容器化的应用程序都将使用它们。开发人员可以放心地假设它们存在。

他们最终应该变得稳定和“无聊”。但是，Linux在其25年的生命周期中一直在不断发展，并且发生了重大变化，包括NPTL（需要重建所有应用程序）和启用容器的功能（正在改变所有应用程序的运行方式）已在过去10年中添加。Kubernetes也需要一段时间才能稳定下来。

#### API与集群控制平面

Kubernetes集群提供了由Kubernetes API server公开的类似REST API的集合，主要支持（主要是）持久性资源上的CRUD操作。这些API充当其控制平面的集线器。

遵循Kubernetes API约定（路径约定，标准元数据等）的REST API可以自动受益于共享API服务（授权，身份验证，审核日志记录），并且通用客户端代码可以与它们(CLI和UI可发现性，UI和CLI中的常规状态报告，编排工具的常规等待约定，监视，标签选择)交互。

系统的最低层还需要支持添加较高层提供的功能所需的扩展机制。此外，该层必须适合在单一用途群集和租户群集中使用。核心应提供足够的灵活性，以便更高级别的API可以引入新的作用域（资源集）而不会损害群集的安全模型。

没有这种基本的API机制和语义，Kubernetes就无法运行，其中包括：

* [Authentication](https://kubernetes.io/docs/admin/authentication/):身份验证方案是服务器和客户端都必须同意的关键功能。 API服务器支持基本身份验证（用户名/密码）（注意：我们最终可能会弃用基本身份验证。），X.509客户端证书，OpenID Connect令牌和承载令牌，其中的任何（但不是全部）都可能被禁用。客户端应支持kubeconfig支持的所有形式。第三方身份验证系统可以实现TokenReview API并配置身份验证Webhook来调用它，尽管选择非标准身份验证机制可能会限制可用客户端的数量。
  * tokenReview API（与钩子相同的架构）启用外部身份验证检查，例如通过Kubelet
  * 通过“服务帐户”提供Pod身份。
    * ServiceAccount API，包括通过控制器创建默认的ServiceAccount秘密和通过准入控制器进行注入。
* [Authorization](https://kubernetes.io/docs/admin/authorization/):第三方授权系统可以实现SubjectAccessReview API并配置授权Webhook来调用它。
  * SubjectAccessReview（与挂钩相同的架构），LocalSubjectAccessReview和SelfSubjectAccessReview API启用外部权限检查，例如通过Kubelet和其他控制器进行检查
* REST语义，监视，持久性和一致性保证，API版本控制，默认设置和验证。
  * NIY：需要解决的API缺陷：
    * [Confusing defaulting behavior](https://github.com/kubernetes/kubernetes/issues/34292)
    * [Lack of guarantees](https://github.com/kubernetes/kubernetes/issues/30698)
    * [Orchestration support](https://github.com/kubernetes/kubernetes/issues/34363)
    * [Support for event-driven automation](https://github.com/kubernetes/kubernetes/issues/3692)
    * [Clean teardown](https://github.com/kubernetes/kubernetes/issues/4630)
* NIY: 内置的准入控制语义，同步的准入控制钩子和异步资源初始化。分销供应商，系统集成商和集群管理员必须强加其他策略和自动化。
* NIY: API注册和发现，包括API聚合，用于注册其他API，找出受支持的API并获取受支持的操作，有效负载和结果模式的详细信息
* NIY: ThirdPartyResource和ThirdPartyResourceData API（或其后继），以支持第三方存储和扩展API
* NIY：/componentstatuses API的可扩展且兼容HA的替代品，以确定群集是否已完全启动并正常运行：ExternalServiceProvider（组件注册）
* 组件注册，API服务器端点的自发布和HA集合以及应用程序层目标发现所需的Endpoints API（及其将来的发展）
* 命名空间API（用于确定用户资源的范围）和命名空间生命周期（例如，批量删除）
* 事件API，这是报告重大事件（例如状态更改和错误以及事件垃圾收集）的方式
* NIY: 级联删除垃圾收集器，完成和孤立
* NIY：我们需要一个内置的加载项管理器（与静态pod清单不同，而是在集群级别），以便我们可以向集群自动添加自托管组件和动态配置，因此我们可以从正在运行的集群中现有组件中排除功能。它的核心是一个基于拉式的声明式协调器，由当前的加载项管理器提供，并在白盒应用程序管理文档中进行了描述。一旦在API中应用了支持，这将变得更加容易。
  * 附加组件应该是作为群集的一部分进行管理的群集服务，并且应提供与群集相同程度的多租户服务。
  * 它们可能但不是必须在kube-system命名空间中运行，但是需要选择选定的命名空间，以免与用户的命名空间冲突。
* API服务器充当群集的网关。根据定义，客户端必须可以从集群外部访问API服务器，而节点，当然还有pod，可能不是。客户端使用/proxy和/ portforward API对API服务器进行身份验证，并将其用作堡垒以及到节点和pod（和服务）的代理/隧道。
* TBD：CertificateSigningRequest API，用于启用凭据生成，尤其是对薄荷Kubelet凭据的生成。

理想情况下，此核API服务器将仅支持所需的最低限度的API，并且将通过聚合，挂钩，初始化程序和其他扩展机制添加其他功能。

请注意，集中式异步控制器（例如垃圾收集）当前由称为“控制器管理器”的单独进程运行。

/healthz和/metrics endpoint可以用于集群管理机制，但是不被视为受支持的API界面的一部分，并且通常不应被客户端使用，以检测集群的存在。应该使用/version endpoint。

API服务器依赖于以下外部组件：

* 永久状态存储（etcd或等效文件；也许有多个实例）

API服务器可能依赖于：

- Certificate authority
- Identity provider
- TokenReview API implementer
- SubjectAccessReview API implementer

#### 执行

Kubernetes中最重要，最杰出的控制器是Kubelet，它是驱动容器执行层的Pod和Node API的主要实现者。没有这些API，Kubernetes将只是一个由键值存储支持的面向CRUD的REST应用程序框架（API最终可能会作为一个独立的项目被剥离出来）

Kubernetes将隔离的应用程序容器作为其默认的本机执行模式执行。 Kubernetes提供了Pod，它可以托管多个容器和存储卷作为其基本执行原语。

Kubelet API表面和语义包括：

* Pod API，Kubernetes执行原语，包括：
  * 基于Pod API中策略的基于Pod可行性的准入控制（资源请求，节点选择器，Node / Pod亲和力和反亲和力，污点和容忍度）。API准入控制可能会拒绝Pod或向其添加其他调度约束，但是Kubelet是Pod可以或不能在给定节点（而不是调度程序或DaemonSet）上运行的最终仲裁器。
  * 容器和卷的语义和生命周期
  * Pod IP地址分配（每个Pod需要一个可路由的IP地址）
  * 一种将Pod绑定到特定安全范围的机制（即ServiceAccount）
  * 卷来源:
    * emptyDir
    * hostPath
    * secret
    * configMap
    * downwardAPI
    * NIY: [Container and image volumes](http://issues.k8s.io/831) (and deprecate gitRepo)
    * NIY: Claims against local storage, so that complex templating or separate configs are not needed for dev vs. prod application manifests
    * flexVolume (which should replace built-in cloud-provider-specific volumes)
  * 子资源：绑定，状态，执行，日志，附加，portforward，代理。
* NIY：API资源检查点的可用性和引导
* 容器映像和日志生命周期
* Secret API和启用第三方秘密管理的机制
* ConfigMap API，用于组件配置以及Pod引用
* Node API，托管于Pod
  * 在某些配置中可能仅对集群管理员可见
* 节点和Pod网络及其控制器（路由控制器）
* 节点清单，运行状况和可达性（节点控制器）
  * 特定于云提供商的节点清单功能应划分为特定于提供商的控制器。
* 终止容器垃圾收集
* volume控制器
  * 特定于云提供商的附加/分离逻辑应分为特定于提供商的控制器。需要一种从API提取提供程序特定的卷源的方法。
* PersistentVolume API
  * 如上所述，至少由本地存储支持
* PersistentVolumeClaim API

同样，集中的异步功能（例如终止的Pod垃圾收集）由Controller Manager执行。

控制器管理器和Kubelet当前呼出到“云提供商”界面，以从基础结构层查询信息和/或管理基础结构资源。但是，我们正在努力将这些接触点（问题）提取到外部组件中。预期的模型是无法满足的应用程序/容器/操作系统级别的请求（例如Pods，PersistentVolumeClaims）可作为外部“动态预配置”系统的信号，这将使基础架构可用以满足这些请求，并使用基础架构资源（例如，节点，PersistentVolumes）在Kubernetes中表示它们，以便Kubernetes可以将请求和基础架构资源绑定在一起。

Kubelet依赖于以下外部组件：

* Image registry
* Container Runtime Interface implementation
* Container Network Interface implementation
* FlexVolume implementations ("CVI" in the diagram)

还可能依赖于：

- NIY: Cloud-provider node plug-in, to provide node identity, topology, etc.
- NIY: Third-party secret management system (e.g., Vault)
- NIY: Credential generation and rotation controller

接受的分层违规：

* 明确的服务链接
* API服务器的kubernetes服务

***

### 2.2 应用分层： 部署和路由

应用程序管理和组合层，提供自我修复，扩展，应用程序生命周期管理，服务发现，负载平衡和路由-也称为编排和服务结构。

这些API和功能对于Kubernetes的任何发行版都是必需的。Kubernetes应该为这些API提供默认实现，但是只要一致性测试通过，就可以替换任何或所有这些功能的实现。没有这些，大多数容器化的应用程序将无法运行，并且几乎没有任何已发布的示例可用。绝大多数的容器化应用程序将使用其中的一个或多个。

Kubernetes的API提供类似IaaS的以容器为中心的原语，以及生命周期控制器，以支持所有主要类别的工作负载的编排（自我修复，扩展，更新，终止）。这些应用程序管理，组合，发现和路由API和功能包括：

* 默认调度程序，在Pod API中实现调度策略：资源请求，nodeSelector，节点和pod关联/反关联，污点和容忍度。调度程序在群集上或群集外作为单独的进程运行。
* NIY：重新计划程序，以积极主动的方式删除已计划的Pod，以便可以将其替换并重新计划到其他节点。
* 连续运行的应用程序：些应用程序类型都应通过声明性更新，级联删除和孤立/采用来支持转出（和回滚）。除DaemonSet以外，所有都应支持水平缩放。
  * Deployment API，协调无状态应用程序的更新，包括子资源（状态，规模，回滚）。
    * ReplicaSet API，用于简单的可替代/无状态应用程序，尤其是特定版本的Deployment Pod模板，包括子资源（状态，规模）
  * DaemonSet API，用于集群服务，包括子资源（状态）
  * StatefulSet API，用于有状态的应用程序，包括子资源（状态，规模）
  * PodTemplate API，DaemonSet和StatefulSet用于记录更改历史记录。
* 终止批处理应用程序：这些应包括对终止作业的自动剔除（NIY）的支持。
  * The Job API ([GC discussion](https://github.com/kubernetes/kubernetes/issues/30243))
  * The CronJob API
* 发现，负载平衡和路由
  * 服务API，包括群集IP的分配，服务分配图的修复，通过kube-proxy或等效代理进行的负载平衡以及服务的自动端点生成，维护和删除。NIY：LoadBalancer服务支持是可选的，但是如果支持，则必须通过一致性测试。如果/当添加它们时，仅当发行版支持LoadBalancer服务时，才应提供对LoadBalancer和LoadBalancerClaim API的支持。
  * Ingress API，包括内部L7（NIY）
  * 服务DNS。使用正式的Kubernetes模式的DNS是必需的。

应用程序层可能取决于：

* Identity provider (to-cluster identities and/or to-application identities)
* NIY: Cloud-provider controller implementation
* Ingress controller(s)
* Replacement and/or additional schedulers and/or reschedulers
* Replacement DNS service
* Replacement for kube-proxy
* Replacement and/or [auxiliary](https://github.com/kubernetes/kubernetes/issues/31571) workload controllers, especially for extended rollout strategies

***

### 2.3 治理层：自动化和策略执行

策略执行和更高级别的自动化。

这些API和函数对于运行的应用程序应该是可选的，并且应该可以通过其他解决方案来实现。

每个受支持的API /功能都应适用于企业的大部分操作，安全性和/或治理方案。

必须能够配置和发现集群的默认策略（也许类似于Openshift的新项目模板机制，但支持多个策略模板，例如系统名称空间与用户名称空间的模板）至少支持以下用例：

* Is this a: (source: [multi-tenancy working doc](https://docs.google.com/document/d/1IoINuGz8eR8Awk4o7ePKuYv9wjXZN5nQM_RFlVIhA4c/edit?usp=sharing))
  - Single tenant / single user cluster
  - Multiple trusted tenant cluster
  - Production vs. dev cluster
  - Highly tenanted playground cluster
  - Segmented cluster for reselling compute / app services to others
* Do I care about limiting:
  - Resource usage
  - Internal segmentation of nodes
  - End users
  - Admins
  - DoS

自动化API和功能：

- The Metrics APIs (needed for H/V autoscaling, scheduling TBD)
- The HorizontalPodAutoscaler API
- NIY: The vertical pod autoscaling API(s)
- [Cluster autoscaling and/or node provisioning](https://git.k8s.io/contrib/cluster-autoscaler)
- The PodDisruptionBudget API
- Dynamic volume provisioning, for at least one volume source type
  - The StorageClass API, implemented at least for the default volume type
- Dynamic load-balancer provisioning
- NIY: The [PodPreset](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/service-catalog/pod-preset.md) API
- NIY: The [service broker/catalog](https://github.com/kubernetes-incubator/service-catalog) APIs
- NIY: The [Template](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/apps/OBSOLETE_templates.md) and TemplateInstance APIs

策略API和功能：

* [Authorization](https://kubernetes.io/docs/admin/authorization/): The ABAC and RBAC authorization policy schemes.
  - RBAC, if used, is configured using a number of APIs: Role, RoleBinding, ClusterRole, ClusterRoleBinding
* The LimitRange API
* The ResourceQuota API
* The PodSecurityPolicy API
* The ImageReview API
* The NetworkPolicy API

管理层可能取决于：

* Network policy enforcement mechanism
* Replacement and/or additional horizontal and vertical pod autoscalers
* [Cluster autoscaler and/or node provisioner](https://git.k8s.io/contrib/cluster-autoscaler)
* Dynamic volume provisioners
* Dynamic load-balancer provisioners
* Metrics monitoring pipeline, or a replacement for it
* Service brokers

***

### 2.4 接口层： 库和工具

建议为分发使用这些机制，并且用户也可以独立下载和安装这些机制。它们包括由Kubernetes官方项目开发的常用库，工具，系统和UI，尽管其他工具也可以用来完成相同的任务。公开的示例可能会使用它们。

在一些Kubernetes拥有的GitHub组织下开发的常用库，工具，系统和UI。

* Kubectl -- We see kubectl as one of many client tools, rather than as a privileged one. Our aim is to make kubectl thinner, by [moving commonly used non-trivial functionality into the API](https://github.com/kubernetes/kubernetes/issues/12143). This is necessary in order to facilitate correct operation across Kubernetes releases, to facilitate API extensibility, to preserve the API-centric Kubernetes ecosystem model, and to simplify other clients, especially non-Go clients.
* Client libraries (e.g., client-go, client-python)
* Cluster federation (API server, controllers, kubefed)
* Dashboard
* Helm

这些组件可能取决于：

* Kubectl extensions (discoverable via help)
* Helm extensions (discoverable via help)

***

### 2.5 生态系统

这些东西根本不是Kubernetes的真正“一部分”。

我们已经在许多领域为Kubernetes定义了明确的边界。

虽然Kubernetes必须提供部署和管理容器化应用程序通常所需的功能，但通常来说，我们会在补充Kubernetes通用编排功能的区域中保留用户的选择，特别是在拥有自己竞争环境的区域中，这些领域由众多满足各种需求和偏好的解决方案组成。 Kubernetes可以为此类解决方案提供插件API，或者可以公开可由多个后端实现的通用API，也可以公开此类解决方案可以定位的API。有时，该功能可以与Kubernetes完美组合而无需显式接口。

此外，要被视为Kubernetes的一部分，组件必须遵循Kubernetes的设计约定。例如，其主要接口是特定于域的语言（例如Puppet，Open Policy Agent）的系统与以Kubernetes API为中心的方法不兼容，并且可以很好地与Kubernetes一起使用，但不会被认为是Kubernetes的一部分。同样，旨在支持多种平台的解决方案可能不会遵循Kubernetes API约定，因此不会被视为Kubernetes的一部分。

* 内部容器映像：对于容器映像的内容，Kubernetes并没有意见-如果它位于容器映像内部，则位于Kubernetes外部。例如，这包括特定于语言的应用程序框架。

* 在Kubernetes之上

  * 持续集成和部署：Kubernetes在源到映像空间中不受限制。它不会部署源代码，也不会构建您的应用程序。持续集成（CI）和持续部署工作流是不同用户和项目有其自己的要求和偏好的领域，因此，我们旨在促进在Kubernetes上分层CI / CD工作流，但并没有规定它们应该如何工作。
  * 应用程序中间件：Kubernetes不提供应用程序中间件（例如消息队列和SQL数据库）作为内置基础结构。但是，它可以提供诸如服务代理集成之类的通用机制，以使提供，发现和访问此类组件变得更加容易。理想情况下，此类组件只能在Kubernetes上运行。
  * 日志记录和监视：Kubernetes不提供日志记录聚合，全面的应用程序监视，遥测分析和警报系统，尽管此类机制对于生产集群至关重要。
  * 数据处理平台：Spark和Hadoop是众所周知的示例，但是有很多这样的系统。
  * 特定于应用程序的操作员：Kubernetes支持针对常见类别的应用程序进行工作负载管理，但不支持特定应用程序。
  * 平台即服务：Kubernetes为众多专注，自以为是的PaaS（包括DIY）奠定了基础。
  * 服务即功能：类似于PaaS，但FaaS还会侵占容器和特定于语言的应用程序框架。
  * 工作流程编排：“工作流程”是一个非常广泛的领域，其解决方案通常针对特定用例（数据流图，数据驱动的处理，部署管道，事件驱动的自动化，业务流程执行，iPaaS）量身定制。以及特定的输入和事件源，通常需要使用任意代码来评估条件，操作和/或故障处理。
  * 配置DSL：特定于域的语言不利于分层高级API和工具，它们通常具有有限的可表达性，可测试性，熟悉度和文档记录，可以促进复杂的配置生成，它们倾向于损害互操作性和可组合性，使依赖关系管理复杂化，并且经常使用颠覆抽象和封装。
  * Kompose：Kompose是一个项目支持的适配器工具，可促进从Docker Compose迁移到Kubernetes，并支持简单的用例，但不遵循Kubernetes约定，它基于手动维护的DSL。
  * ChatOps：也是适配器工具，用于多种聊天服务。

* 基础Kubernetes：

  * 容器运行时：Kubernetes不提供其自己的容器运行时，但提供了用于插入您选择的容器运行时的接口。
  * 映像注册表：Kubernetes将容器映像拉到节点。
  * 集群状态存储：Etcd
  * 网络：与容器运行时一样，我们支持促进可插拔性的接口（CNI）。
  * 文件存储：本地文件系统和网络连接的存储。
  * 节点管理：Kubernetes既不提供也不采用任何全面的机器配置，维护，管理或自我修复系统，通常在不同的公共/私有云中，针对不同的操作系统，对于可变与不可变的基础架构，对于已经在其Kubernetes集群之外使用工具的商店等进行不同的处理。
  * 云提供商：IaaS设置和管理。
  * 集群创建和管理：社区开发了许多工具，例如minikube，kubeadm，bootkube，kube-aws，kops，kargo，kubernetes-wherewhere等。从工具的多样性可以看出，没有一种千篇一律的集群部署和管理解决方案（例如，升级）。有很多可能的解决方案，每种解决方案都有不同的权衡。也就是说，常见的构建基块（例如，安全的Kubelet注册）和方法（尤其是自托管）将减少此类工具所需的自定义编排量。

  我们希望看到生态系统能够构建和/或集成解决方案来满足这些需求。

  最终，大多数Kubernetes开发都应该属于生态系统。

***

**参考**

[https://github.com/kubernetes/community/blob/master/contributors/design-proposals/architecture/architecture.md](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/architecture/architecture.md)

[https://github.com/kubernetes/community/blob/master/contributors/design-proposals/architecture/architectural-roadmap.md](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/architecture/architectural-roadmap.md)
