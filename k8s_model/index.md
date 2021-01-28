# [译]Kubernetes成熟度模型


原文链接：[kubernetes maturity model](https://www.fairwinds.com/kubernetes-maturity-model)

**水平有限，本文不免存在遗漏或错误之处。如有疑问，请查阅原文。**

***

## 前言

Kubernetes有很多好处。 同时，当组织采用云原生技术时，它可能变得复杂。 Kubernetes成熟度模型的存在可帮助您确定自己在迁移到原生云的过程中所处的位置，无论您是Kubernetes的新手还是有部署经验的人。 这是一个重要的工具，可帮助您自我确定您所处的阶段，了解环境中的差距并获得有关增强和改善Kubernetes技术栈的见解。



![](model.png)

### 如何使用Kubernetes成熟度模型

Kubernetes和您的工作负载在不断变化。 使用此成熟度模型时，请知道，如果确实达到某个阶段，则可能仍需要重新访问以前的阶段。 另外，请注意，Kubernetes的成熟并非一朝一夕就能完成，而是需要时间。 Kubernetes成熟度模型应用作工具，以帮助您了解在迁移到云原生过程中需要集中精力或需要帮助的地方。

***

## 1. 准备阶段

从哪里开始？如何证明k8s的价值？谁可以信任？

在该阶段，你将学习/精通以下内容：

云原生和k8s将如何帮助推动业务和技术目标。它将耗费什么？并就整个组织的目标达成共识。

* 明白云原生、容器、以及k8s的价值
* 能够向企业领导者描述该价值
* 得到团队，领导和整个组织的支持

***

### 必要条件

**明白你的问题**

为什么要使用kubernetes？想要通过kubernetes解决什么问题？

**同意使用OSS**

转换到kubernetes需要你明白开源软件(OSS)在云原生生态中的角色和能量

**接受投资未来**

kubernetes的旅程将会耗费大量的时间和金钱。你需要面向未来投资。

***

### 介绍

在采用Kubernetes时，第一步是准备工作。在这里，理解并能够阐明云原生和Kubernetes为什么对组织很重要至关重要。一些核心概念包括理解云原生计算，容器和Kubernetes的价值和影响。在较高的层次上，我们在这里每个定义。

**Cloud Native** 

> 云原生技术有利于各组织在公有云、私有云和混合云等新型动态环境中，构建和运行可弹性扩展的应用。云原生的代表技术包括容器、服务网格、微服务、不可变基础设施和声明式API。
>
> 这些技术能够构建容错性好、易于管理和便于观察的松耦合系统。结合可靠的自动化手段，云原生技术使工程师能够轻松地对系统作出频繁和可预测的重大变更。
>
> 云原生计算基金会（CNCF）致力于培育和维护一个厂商中立的开源生态系统，来推广云原生技术。我们通过将最前沿的模式民主化，让这些创新为大众所用。

Source: [CNCF definition](https://github.com/cncf/toc/blob/master/DEFINITION.md).

云原生计算的好处包括更快的发布速度，易于管理，通过容器化和云标准降低了成本，能够构建更可靠的系统，避免了供应商锁定以及改善了客户应用体验。

**Container**

> 一个打包代码及其所有依赖项的标准软件单元，使得应用程序可以从一个计算环境快速可靠地运行到另一个计算环境

 Source: [Docker](https://www.docker.com/resources/what-container)

> 在k8s中，你运行的每个容器都是可重复的；通过包含依赖项实现标准化意味着无论您在哪里运行它，都可以得到相同的行为。容器将应用程序与基础主机基础结构分离。这使得在不同的云或OS环境中的部署更加容易

Source: [CNCF concepts](https://kubernetes.io/docs/concepts/containers/).

**Kubernetes**

> Kubernetes是一个可移植的，可扩展的开源平台，用于管理容器化的工作负载和服务，可促进声明式配置和自动化。 它拥有一个庞大且快速增长的生态系统。 Kubernetes的服务，支持和工具广泛可用。Kubernetes为您提供了一个可弹性运行分布式系统的框架。 它负责应用程序的扩展和故障转移，提供部署模式等。

 Source: [CNCF What is Kubernetes](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/).

**Infrastructure as code (IAC)** 

使用配置语言配置和管理基础结构的能力。 IAC为网络，负载平衡器，虚拟机，Kubernetes集群和监视等基础架构的管理带来了现代软件开发的可重复性，透明性和测试。 IAC的主要目标是减少错误和配置漂移，同时允许工程师将时间花在更高价值的任务上。 IAC定义了基础架构的最终状态，而不是定义要执行的一系列步骤-像Terraform这样的IAC工具可以针对您的基础架构多次运行，从而产生相同的预期结果。

使用云用户界面创建托管的Kubernetes集群是一个相对简单的过程，但是使用基础结构作为代码有助于标准化集群配置并管理集群节点的附件，例如网络策略，维护窗口以及身份和访问管理（IAM） 和工作量。

***

### 公司和文化的改变

云原生实践，容器和Kubernetes的采用可以有助于实现战略业务目标，包括通过适当的功能来帮助最大程度地节省成本，适应需求，降低和分散风险。

它还代表了整个组织的文化变化。在准备进行转型时，您需要团队，领导层和整个组织的支持，因为这将需要不同的技能，新的团队结构和改变的意愿。

***

### 挑战

在准备阶段，可能面临的挑战：

1. 从何处开始或谁信任和倾听的不确定性。
2. 不确定您的用例是否可行或可转换到云原生/ Kubernetes世界
3. 需要证明业务价值/领导成本

***

### 输出

* 您的技术计划与业务/领导力支持相辅相成，以取得成功。
* 您接受并交流文化和跨职能工作流程的变化。
* 您接受这一旅程将需要大量的投资和对新技术的信任。
* 您了解开源软件（OSS）在云原生生态系统中的作用和功能。
* 您已经考虑并确认了希望获得的价值
* 你知道你的业务重点。
* 您了解有关安全性，效率和可靠性的要求。

***

### 学习资源

[Deep Dive into Core Concepts](https://kubernetes.io/docs/concepts/)

[Kubernetes Basic Training](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

[Why Infrastructure as Code and Kubernetes](https://www.fairwinds.com/blog/why-infrastructure-as-code-kubernetes)

[How to Create, View and Destroy a Pod in Kubernetes](https://www.fairwinds.com/blog/how-to-create-view-and-destroy-a-pod-in-kubernetes)

***

## 2. 改造阶段

如何设置Kubernetes基础架构并转移工作负载？

在该阶段，你将学习/精通以下内容：

Kubernetes的基础知识以及采用和转换现有思维方式，工作流程和实践到平台的能力。

* 你将精通kubernetes术语，并能够将现有技术映射到云原生环境
* 你将容器化你的应用并把工作负载转移到kubernetes上
* 你将清理你的技术债务并避免把它带到kubernetes上

***

### 必要条件

**理解核心概念**

你将为采用kubernetes做花费时间理解核心概念的准备。

**全组织范围接受**

您可以在整个组织中进行购买，以投资Kubernetes概念证明或将Kubernetes用于所有新应用程序。

**优先级工作负载**

您了解使用分阶段方法计划迁移到Kubernetes的工作负载

***

### 介绍

转变是您迁移到Kubernetes的阶段。在该阶段，你将通过部署你的第一个集群和负载来验证您的基础知识和理解。在改造阶段，您应该对基础知识有所准备，但同时可能缺乏完成该阶段所需的专业知识。

你将花费大量的时间在该阶段。进行一些关键活动时，它涵盖了您的初始实施，迁移和学习曲线。当您采用Kubernetes时，不要被“启动并运行”的文章所迷惑。在设置集群和准备生产之间存在功能差距。您可能会发现在此阶段进行Kubernetes概念验证或与Kubernetes专家合作以确保您正在设置第一个集群以满足工作负载的需求很有帮助。

***

### 采用新的语言和架构体系

当你要采用kubernetes时，你不只是做简单的准备，你将开始练习和理解新的语言、架构和负载需求。

**术语学习**

在准备阶段，你可能已经明白了一些术语，但在改造阶段期间，你将需要使用所有的kubernetes术语。一些必要的概念将包含在内，包括node、pod、namespace、ReplicaSet、控制器等等。尽管事先进行培训很重要，但是在实践中您会接触到每个功能时，您会更好地理解它们。

**应用架构和理解需求**

你需要将应用架构映射到新的云原生环境中。这样做将帮助您发现需求并发现应用程序的依赖关系，以便您可以成功拥抱容器。它还将使您能够重新查看以前的历史假设和决策。例如：

| Old World                               | New World                                   |
| --------------------------------------- | ------------------------------------------- |
| SSH to server                           | Deploy by code and immutable infrastructure |
| Dedicated workloads                     | Self-healing and ephemeral workloads        |
| Add server to scale                     | Add a pod or nod to scale                   |
| Configuration management for deployment | Containerization for deployment             |

**工作负载理解**

你将理解运行在集群中内不同类型的工作负载：Deployments、Jobs、CronJobs、ReplicaSet、DaemonSets和Pods

****

### 技术改造

在第二阶段，你将开始在kubernetes上运行工作负载。为了成功应对技术改造，有10个主要的步骤需要执行。这是您将要执行的每个步骤的简要概述。准备在此过程的每个步骤上花费大量时间。

1. **深入理解和项目计划** - 无论您是在本地，在数据中心还是已经迁移到云，您的第一步是深入研究现有技术栈。您需要调查技术栈的各个方面，从基础网络，基础结构，配置，密钥管理到如何部署应用程序及其依赖项。迁移到Kubernetes时，您需要确定对技术的要求。该步骤帮助你避免遗漏重要的依赖。在这次深入研究的基础上，您可以制定一个项目计划，这是您进行迁移的路线图。
2. **应用容器化** - 您的应用程序可能已经被容器化了，在这种情况下，您可以继续执行第三步了。如果不是，则需要根据 [twelve-factor app](https://12factor.net/) 方法细分应用程序。这非常重要，因为您将需要使应用程序能够生存下去（您的容器随时可能被杀死）。您需要能够干净地站立您的应用程序和容器。在此步骤中，我们建议您从构建工件中提取密钥和配置。 Kubernetes是短暂的，因此，您可以维护标准和安全性，并在容器运行时简单地注入密钥和配置。
3. **建立云基础设施** - 你需要确认你的云服务提供商：AWS、GCP、Azure或者像EKS、GKS和AKS的kubernetes管理服务。如果你选择了kubernetes管理服务，你就不需要额外的工作用于构建你自己的kubernetes基础设施。此步骤的一部分，您需要设置基础云配置，VPC，安全组，身份验证和授权等。
4. **构建Kubernets基础设施** - 在该步骤，需要考虑一些设计注意事项，以避免做出可能需要耗时的集群重建或网络和成本影响的选择。类似的注意事项有：您应该在哪些区域拥有多少个群集，并具有多少个可用区（AZ）？需要多少个单独的环境，集群和命名空间？服务之间应如何相互通信和发现？安全性是在VPC，集群还是Pod级别上？您的重点应该放在可重复性上。您将需要利用基础架构作为代码（IaC），以便可以以一种重复的方式构建集群。在此步骤中，请谨慎使用第一步中的深度研究/项目计划中的配置选项，以确保您不会错过应用程序要求。
5. **编写YAML或者Helm charts** - 在Fairwinds，我们将此步骤称为Kubernating。在这里定义Kubernetes资源以将其放入集群中。可以在此处编写Kubernetes资源YAML文件，但是现在大多数都使用Helm charts将应用程序部署到Kubernetes中。您将专门为您的容器镜像，配置映射模板，密钥或任何特殊应用程序要求编写YAML或Helm charts。
6. **深入了解外部云依赖性** - 你的应用可能会有外部的依赖，例如key store，库、数据库或者其他资产。Kubernetes不是这些依赖生存的好地方。您将需要在Kubernetes之外管理您的有状态依赖关系。例如，您可以使用Amazon RDS之类的工具中的独立数据库，然后将其放入Kubernetes中。然后，您的应用程序可以在Kubernetes的pod中运行，并与这些依赖项对话。
7. **定义Git工作流程** - Kubernetes的一个主要优点是能够以可重复的方式部署代码而无需人工干预。您通常会通过Git将代码提交到代码仓库，这将启动事件并与将这些更改移动到非生产集群的分支合并。然后，您将对代码进行测试和质量检查，然后合并到master中。这会将您的代码部署到登台或生产中。在此阶段，您只需定义Git工作流的外观即可，即当开发人员推送代码时，Kubernetes会发生什么？
8. **构建CI/CD管道** - 定义Git工作流程后，您将使用Jenkins或CircleCI等自动化工具来设置CI / CD平台。这会将您定义的工作流程转变为实际的构建管道。
9. **非生产测试** - 完成单片式应用程序或微服务架构的步骤1-8之后，您将部署到非生产环境。在这里，您将使用该应用程序以确保其运行，具有足够的资源和限制，测试您的机密信息是否正确以及人们可以访问该应用程序。您将测试杀死pod会发生什么情况。从本质上讲，您将在开始生产之前踢一下轮胎。如果您运行的是整体应用程序，则可以更快地完成此阶段。如果要部署微服务应用程序体系结构，则将为每个服务完成步骤1-8，然后将其部署到非生产环境。一旦所有服务都存在，您就可以看到它们如何协同工作以确保您的应用程序一旦投入生产，应用是正常工作的。
10. **生产推广** - 最后，一旦您在非生产环境中对应用程序进行了全面测试并对此感到满意，只要您构建的生产环境与登台相同，就可以部署流量并将其发送到您的应用程序。在这里，您只需更改负载平衡器或DNS。使用DNS，您可以根据需要进行故障回复。

***

### 债务清理

在实施Kubernetes时，您将有机会清理现有系统中产生的技术债务。 技术债务可以概括为持续吸引人们注意力以完成任务的任何工作流，流程，代码或硬件。 这可能包括已推迟的升级或更新，已解决的代码错误，旧版本依赖性或错误的配置。

自然地，当您迁移到Kubernetes时，您可以评估该技术债务的存在位置，从而避免在新环境中复制它。

***

### 生产力增加与损失

Kubernetes提供了机会来更改和改进团队协作以及交付应用程序或服务的方式。随着您的转变，您的团队也会发生变化。

Kubernetes是一个巨大的变化。您将采用一种新的工作方式，团队中的每个人都会以不同的方式学习该技术。开始时，随着您和您的团队对技术的适应，生产率将会受到影响。在此阶段要有耐心，因为长期的生产率收益将超过短期的损失。

***

### 工具

您需要决定如何做出有关工具的决定，包括：

* 您如何确定哪些问题需要工具解决方案
* 谁负责做出该决定？
* 您如何审核工具？

对于开源工具，您需要评估每个工具是否定期更新，以及是否有足够的社区支持来避免工具过时。在此阶段花时间与使用Kubernetes的开发人员合作回答这些问题。

***

### 在控制与最终灵活性之间取得平衡

Kubernetes的一个好处是开发人员无需操作团队即可提交代码的能力。 所需要的是好的政策，以确保安全性，可靠性和效率。 挑战在于，在设置第一个集群时，需要在为开发人员提供最终灵活性或控制基础架构之间取得平衡。 您是否会在一开始就向开发人员开放Kubernetes，以免影响生产力？ 还是会锁定系统？ 您将使用哪些配置来限制更改？ 您必须考虑如何平衡开发人员需求和公司政策。

***

### 挑战

在转换期间，您可能会面临以下挑战：

* 被复杂性淹没
* 庞大的Kubernetes生态系统导致决策瘫痪
* 缺乏内部专业知识或人才流失
* Kubernetes最佳实践/执行的不确定性

这些挑战是任何IT环境的常见疑问。使用Kubernetes时也是如此，但是在这一领域的人才或专业知识甚至更少。如果出现这些挑战，您可以从托管的Kubernetes服务寻求帮助。

另外，您的团队中存在现有的行为。花时间了解哪些行为至关重要，哪些行为可以更改，哪些历史不再需要，何处存在可以简化的复杂性。

***

### 输出

* 您将在转换阶段投入大量时间并建立您的知识库
* 您将确定在Kubernetes中使用哪些应用程序
* 您将考虑整体解决方案与开放源代码和专有技术的拼凑而成
* 您将了解Kubernetes的基础知识，优点和缺点
* 您将成功进行Kubernetes的概念验证，因此您将能够决定是否继续

***

### 学习资源

[Why Managed Services with EKS, GKE or AKS](https://www.fairwinds.com/what-fairwinds-adds-to-kubernetes-engines-eks-gke-aks)

[Strengths and Weaknesses of AKS, EKS and GKE](https://www.fairwinds.com/blog/strengths-and-weaknesses-of-aks-eks-and-gke)

[Intro to Kubernetes + How to Deploy a Multi-tiered Web Application](https://www.fairwinds.com/blog/kubernetes-intro-how-deploy-multi-tiered-web-app)

***

## 3. 部署阶段

实施流程，CI / CD，赋予开发人员权力并引入一些监控和可观察性。

在该阶段，你将学习/精通以下内容：

Kubernetes实现了基线理解。您正在通过进行基本的开发，部署，管理和故障排除来练习技能。你将：

* 为您的Kubernetes工作负载奠定坚实的基础
* 准备在您的组织中部署Kubernetes
* 通过CI / CD实施构建和部署过程，并引入了一些有限的监控和可观察性。
* 使开发人员能够自助

***

### 必要条件

**应用容器化/kubernetes设置**

您将采用容器并成功运行Kubernetes POC或建立基础架构。

**在预发环境运行工作负载**

您将在预生产中拥有工作负载，或者可能具有某些生产级部署。

**建立的过程**

您将建立初始的Git工作流程，建立CI / CD管道并建立适当的测试流程。

***

### 介绍

当您达到此阶段时，您和您的团队将掌握基本知识。现在，一个应用程序或服务正在生产中运行，可以正确地了解外部依赖关系，通过负载平衡器将流量路由到Kubernetes，并且可以访问日志记录和指标。您还将拥有自动缩放功能。

Kubernetes成熟度模型的这一阶段让您承担所有工作，包括实施构建和部署过程，设置CI / CD，赋予开发人员权力以及引入一些有限的监视和可观察性。

您将在此阶段熟悉所有Kubernetes基础知识。这很重要，因此您可以建立坚实的基础。您几乎已经超过了学习曲线的驼峰。

***

### CI/CD

您将开始建立结构化的构建和部署过程，以展示云和容器本机CI / CD系统的质量。 CI / CD将成为用于构建和部署的经过功能验证的工作流程。

您和您的团队可以半可靠地将代码交付给工作和生产环境，可以通过Git分支和标签管理部署，并可以判断构建和部署是否成功。

范围更广的团队了解部署的结果，包括应更改的内容以及在何处可以到达端点。最后，随着部署过程的成熟，CI / CD将有助于在部署过程的早期发现问题。

***

### 赋予开发人员和运维人员权力

您的开发人员应有权部署到Kubernetes。为了使之成为可能，您的团队将对从源代码管理（scm）到部署的工作流程有基本的了解，并有权访问scm中的合并/标记提交以触发部署。开发人员需要能够执行CI / CD流程的基本调试，并具有对Kubernetes资源定义或Helm图表的kubectl访问。

同时，您将希望操作员以及开发人员可以访问Kubernetes API，以确保他们了解工作负载在何处以及如何查看其状态。 这是确保他们能够调试在Kubernetes中遇到的简单错误或阻止程序的重要部分。

为确保Kubernetes取得成功，请在此阶段花费时间进行培训，并授权所有开发人员和操作团队进行这些活动。

***

### 用户基础

在此阶段，您的团队将能够操作Kubernetes的基础知识。这包括学习和执行Kubernetes操作基础知识，包括：

* 将操作员连接到Kubernetes API
* 了解如何列出和查看资源
* 执行基本动作（对动作如何理解有限的机械动作）

***

### 有限的监控和可视化

您需要运营商和开发人员将监视和可观察性纳入他们的工作负载中。有流行的开源工具，例如Prometheus，还有来自[Datadog](https://www.datadoghq.com/)等供应商的工具。在基础阶段，您将开始使用这些工具，对群集的监视和可观察性有限。您将开始了解工作量和基础结构的哪些方面需要深入了解或保持关注。

***

### 策略&审核工具

在基础阶段结束时，您和您的团队将寻求工具来帮助您成熟的Kubernetes环境。这将包括探索开源工具[open source tooling](https://www.fairwinds.com/open-source-software) ，以帮助解决安全性，策略管理，工作负载配置错误，资源请求和限制等问题。您还可以探索来自软件供应商的工具，这些工具为Kubernetes提供策略驱动的解决方案。[policy-driven solutions for Kubernetes](https://www.fairwinds.com/insights). 

***

### 挑战

在基础阶段，您可能会面临以下挑战：

1. 新的概念和技能可能会使团队进度缓慢或陷入困境
2. 曾经工作的东西不太一样
3. 尚未深入了解，一些问题似乎无法解决或令人沮丧
4. 你有很多问题，没人可以解答

您可能会花费时间询问Kubernetes的问题是否可以解决，从生态系统寻求帮助或寻求Kubernetes专家的帮助。

***

### 输出

* 你已经为Kubernetes工作负载奠定了坚实的基础
* 您有信心并准备在整个组织中部署Kubernetes
* 企业看到了Kubernetes的价值

***

### 学习资源

[Kube 101 Training](https://www.fairwinds.com/kubestart)

[Best Practices for Implementing CI/CD Pipelines in Kubernetes](https://www.fairwinds.com/blog/best-practices-for-implementing-ci-cd-pipelines-in-kubernetes)

[Kubernetes: What Needs Monitoring and Why](https://www.fairwinds.com/kubernetes-monitoring-white-paper)

[Most Helpful Kubernetes Open Source Projects We Use](https://www.fairwinds.com/blog/most-helpful-kubernetes-open-source-projects)

***

## 4. 建立自信阶段

建立对自己的核心能力的信心，以成功地定期部署和发布功能。

在该阶段，你将学习/精通以下内容：

建立核心能力，以成功地定期部署和发布功能。您正在建立更深刻的理解，从而导致更多的自定义，实验和更广泛的组织使用。您将：

* 通过经验建立信心
* 围绕基础设施即代码和配置建立核心标准
* 选择可以紧密协作的Kubernetes附加组件
* 开始使用监控工具来帮助您了解服务挑战

***

### 必要条件

**在生产环境运行工作负载**

现在，你已经运行部分或者全部的工作负载在生产环境

**实施CI/CD**

您已经建立了CI / CD管道，它是经过功能验证的工作流程，用于构建和部署

**初始化监控和可视化**

您已经开始使用监视和可观察性工具来确定Kubernetes基础架构中的差距。

****

### 介绍

随着Kubernetes环境的成熟，您将奠定坚实的基础。现在，当您进入成熟度模型的第四阶段时，就该建立信心了。在第三阶段，您已经建立并运行了Kubernetes基础架构。在第四阶段，您将开始了解Kubernetes的细微差别。

重要的是要记住，建立信心将需要时间，因为您重复执行任务并经历类似的情况。

***

### 每天都有新的理解

想象你是刚刚学习骑自行车的时候，您可能可以直线骑行。建立信心时，您将开始转弯，越过颠簸，甚至在没有握住把手的情况下骑行。在第三阶段中，您学习了如何骑自行车，现在在第四阶段中，当您通过经验了解细微差别时，每天都在学习新的提示和技巧。

您会在某些方面感到很自在，但对其他方面则缺乏信心。例如，随着您对活动性和就绪性探针的熟悉，您将了解微小的配置更改如何改变工作负载的行为。您将更有能力进行更改以产生积极的影响。另一个示例是您可能已经制定了安全策略，但是不能确保所有群集都具有符合要求的配置。

在此阶段，您将了解这些细微差别，然后开始在团队中培训其他人。这将帮助您进一步建立信心。

当您建立信心时，您的好奇心将围绕您迄今为止所经历的事情以及如何进行改进而增加。如果您的Kubernetes集群出现故障，您将进行试验而不是惊慌。

***

### 胜任力

您在选择和实施第三方工具以增强Kubernetes体验方面的核心能力将会增强。您将研究有助于提高安全性的工具([Trivy](https://github.com/aquasecurity/trivy), [Polaris](https://www.fairwinds.com/polaris), [Kubesec](https://github.com/controlplaneio/kubesec))，设置正确的资源利用率([Goldilocks](https://github.com/FairwindsOps/goldilocks))或帮助进行升级 ([Nova](https://github.com/FairwindsOps/nova))等等。

您还可以胜任正在执行的非标准活动，以及已更改和自定义的默认活动。

***

### 排查故障

您还将对故障排除更有信心。也许您的应用程序在每次部署时都会崩溃，而您却不知道为什么。您将开始研究配置并研究资源。以前您知道这个概念，但是您不了解它对应用程序的影响。现在，您发现没有为应用程序提供足够的资源来启动它。

您现在有一个可重复进行的故障排除周期，因此更改可以快速完成并反复进行，直到它们起作用为止。

***

### 挑战

在基础阶段，您可能会面临以下挑战：

1. 担心单点故障和故障排除
2. 团队实践不统一
3. 对所做的决定感到遗憾
4. 功能缺失
5. 令人失望的表现

对您的Kubernetes集群充满信心与经验有关。但是，许多团队缺乏学习工作所需的时间。培训，专业和托管服务，审计和配置验证可在此提供帮助。

***

### 输出

* 通过经验以及Kubernetes专家培训，专业和托管服务，审计和配置验证的帮助，您已经对Kubernetes集群建立了信心。
* 您正在尝试使用Kubernetes，因为您已经实现了有关基础架构的代码（IaC）和配置标准
* 您开始监视和了解您的服务挑战
* 您可以放心地选择加载项，但是在部署和管理所有工具时会遇到挑战

***

### 学习资源

[Kubernetes Audit and Improve](https://www.fairwinds.com/kubernetes-audit-improve)

[5 Kubernetes Security Tools You Should Use](https://www.fairwinds.com/5-kubernetes-security-tools-you-should-use)

[Managing Kubernetes Configuration for Security, Reliability and Efficiency](https://www.fairwinds.com/manage-kubernetes-configuration-security-efficiency-reliability)

***

## 5. 改进操作阶段

提升集群的安全性、可靠性和效率

在该阶段，你将学习/精通以下内容：

在整个企业中成功部署Kubernetes

* 通过花费时间配置集群来提高安全性，效率和可靠性
* 你的团队可以聚焦于业务，而不是维护kubernetes
* 您的挑战现在很复杂，需要您的内部团队以外的Kubernetes专业知识。

***

### 必要条件

**kubernetes自信**

你以前通过经验建立了在kubernets上的自信

**标准实施**

你已经建立了CI/CD，IaC和配置的标准

**开始监控**

您开始监视和了解您的服务挑战。

***

### 介绍

到达kubernetes成熟度模型的该阶段是一个标志性的里程碑。您将定期成功地将功能部署和交付到Kubernetes中。您的开发人员将会满意：

1. 预计它们会影响Kubernetes的术语，例如Deployment，Daemon Set，Service或Namespace
2. 修改Kubernetes资源的某些配置，例如ConfigMap或Helm图表
3. 对CI / CD过程进行故障排除
4. 对Kubernetes中的应用程序/服务进行故障排除，包括访问日志和事件以及检索指标/监控

您不仅拥有基础知识，而且现在可以专注于改善运营。

***

### 安全、效率和可靠

为了使组织成功采用Kubernetes，您将需要花费时间来提高安全性，效率和可靠性。重要的是要了解有关这些主题的配置。

#### 安全

您需要确定谁负责Kubernetes集群安全以及如何对其进行管理。您是否可以快速识别配置错误，从而在容器和Kubernetes实施中留下安全漏洞？

#### 效率

Kubernetes是否有效运行？谁负责监视资源利用率，以确保您不会过度配置资源或配置资源不足。您的应用程序或服务的范围是什么？

#### 可靠

Kubernetes是否会带来任何停机挑战？系统可靠吗？您是否实现了所有的自我修复，自动缩放功能，并且没有引入配置问题？

这些领域中的每一个都需要您和您的团队制定策略并建立方法，以轻松确保在整个集群中实施这些策略。策略驱动的配置验证可以帮助：

* 在CI / CD阶段通过开放策略代理（OPA）集成或作为准入控制器来实施自定义策略
* 通过在应用程序开发期间检测问题来防止错误，从而防止错误首先进入生产环境
* 通过自动扫描容器中的漏洞并审核群集中的漏洞来节省时间
* 通过确定如何提高Kubernetes计算资源的效率来降低成本

***

### 挑战

与每个成熟阶段一样，您会遇到新的痛点。您可能会遇到以下挑战：

1. 复杂性现在不在您的舒适范围内
2. 维护和运营工作量很大，并花费了团队时间和精力
3. 团队担心人员不足，无聊，分心或技能不足以应对更深的Kubernetes挑战

达到这一成熟水平时，克服这些挑战可能会很复杂。它可能需要内部雇用专家，但是预算可能不存在。这是培训，专业和托管服务，审计和配置验证可以提供帮助的另一个领域。

***

### 输出

* 您在Kubernetes成熟度上取得了重要的里程碑
* 通过花费时间配置群集来提高安全性，效率和可靠性
* 您的团队将能够专注于您的业务，而不是维护Kubernetes

第五阶段不是终点线。如果不进行第6阶段和第7阶段，您将永远不会意识到Kubernetes的全部好处和价值。

***

### 学习资源

[Kubernetes Audit and Improve](https://www.fairwinds.com/kubernetes-audit-improve)

[Kubernetes Best Practices for Security](https://www.fairwinds.com/blog/kubernetes-best-practices-for-security)

[Closing the Kubernetes Services Gap](https://www.fairwinds.com/kubernetes-managed-services-whitepaper)

[Kubernetes Best Practices for Reliability](https://www.fairwinds.com/blog/kubernetes-best-practices-reliability)

[Kubernetes Best Practice for Efficient Resource Utilization](https://www.fairwinds.com/blog/kubernetes-best-practice-efficient-resource-utilization)

***

## 6. 测量&控制

通过复杂的监控来推动策略和控制，从而对工作负载有更深入的了解。

在该阶段，你将学习/精通以下内容：

复杂的监视和警报功能可让您对工作负载有更深入的功能了解。您将：

* 围绕允许的行为，安全性，配置和标准，提出更强的见解和更严格的控制
* 将基础结构作为代码和CI / CD驱动的流程加倍
* 探索网络策略，工作负载标识和服务网格以锁定工作负载能力

***

### 必要条件

**部署和发布**

您定期成功部署和发布功能

**集群配置**

您正在花费时间通过评估群集来提高安全性，效率和可靠性

**团队聚焦于业务**

您的Kubernetes基础架构是生产级的（技术部门有限），使您的团队可以专注于您的应用程序或服务

***

### 介绍

Kubernetes成熟度的下一阶段是引入更多的环境测量和控制。 您和您的团队在Kubernetes中运作良好，具有全面的了解，并在整个组织范围内得到采用。 您正在对Kubernetes进行更深入的功能理解，并就如何在集群和整个环境中完成工作提出了意见。 此外，团队已准备好解决先前阶段的技术债务。

先前的阶段已经引入了一些监视和可观察性。在此阶段，您将收集并处理更多数据，见解和工具，以开始了解要测量和跟踪的内容以及如何控制Kubernetes。

***

### 更复杂的监视和警报

现在，使用更复杂的监视和警报是帮助了解常见问题，Kubernetes错误以及如何解决它们的重点。 您将更熟悉如何构建监视仪表板以在需要帮助之前发现问题和常见的错误配置。 当问题发生时，您也将知道它们的大致位置，以便更轻松地进行故障排除。

***

### 测量和跟踪

在此阶段，您将开始改进衡量Kubernetes环境并跟踪成功的方式。测量将围绕五个关键领域：

1. **安全** - 您将测量容器或群集中存在多少个漏洞以及哪些漏洞，以及修补工作负载，群集或附加组件的频率/时间。
2. **审计** - 您将创建一个审计跟踪，以了解谁最近执行了操作以及集群中工作负载正在执行哪些操作。您将能够确定是否发生了未经授权的访问或操作。
3. **漂移** - 您将能够确定哪些工作负载不符合您的标准，正在运行哪些版本的依赖项/集群附加组件以及工作负载是否与Kubernetes的未来版本兼容。
4. **效率** - 您将进行测量以跟踪工作负载的典型或标准资源使用情况以及集群中节点的典型容量/使用情况。您还将知道群集扩展的频率。
5. **速度** - 您将采取措施提高开发速度。这将包括了解部署的发布频率，多少用户访问您的集群以及在集群内执行的最常见操作。

***

### 控制

您将在工作负载和其他Kubernetes资源方面遇到痛苦，主要是在一致性方面。 工作负载可能不一致，也可能手动部署，然后进行修改。 跨容器和群集的配置中可能存在差异，这对于识别，更正和保持一致可能是一个挑战。 工作负载可能会杂乱无章，影响其他工作负载。 工作负载可能访问过多，从而导致安全问题。 可能存在可靠性或可伸缩性问题（无法充分扩展或扩展得太频繁）。 由于使用过多的资源或未清理工作负载，成本可能会攀升至很高。

随着您和您的团队的成熟，所有这些要点都是很自然的。在此阶段中，您要围绕安全性，配置和工作流来制定控制策略。以下是您需要考虑的一些示例。

***

### 安全

Kubernetes工作负载安全性至关重要。您需要控制群集权限，并且应该能够回答：

* 谁有权访问集群 
* 用户可以在集群中执行哪些操作？
* 工作负载在集群中可以采取什么行动？ 
* 集群中具有什么级别的权限工作负载？ 
* 集群中工作负载之间的网络策略是什么？

***

### 配置

坚固的Kubernetes环境将具有用于一致性的配置标准。您应该在以下位置设置控件：

* Kubernetes资源的住处和定义位置？ 
* 什么变化会在何时发生？ 
* 您的资源代码审查流程是什么？ 
* 集群中可以部署什么类型的资源？
* 哪些名称空间可供哪些用户使用？ 
* 哪些名称空间工作负载部署到？ 
* 如何设置可用于工作负载或名称空间的资源量？ 
* 您在工作负载/部署中常见的标准/默认设置是什么？

***

### 工作流

同样，您将建立工作流，以了解如何部署工作负载和服务，升级路径和职责：

* 谁可以将工作负载和服务部署到您的集群？ 
* 如何将工作负载和服务部署到您的集群？ 
* 环境之间的提升路径是什么？ 
* 谁负责您的环境的哪些方面？

通过回答这些问题，您现在将拥有一组策略来开始在集群中实施配置更改。 您还具有Kubernetes经验，可以循环回到这些更高级的主题。 您将重新访问可能只是选择了默认选项的配置，检查发生了什么并进行更改。

***

### 挑战

1. 跨工作负载，集群，团队的配置和流程不一致 
2. 用户和工作负载具有过多的访问权限，并且可能以不安全的方式运行/操作。
3. 常见的可靠性问题很麻烦，并且由于缺乏有效的监视/警报而无法很好地理解。

***

### 输出

第六阶段将使您能够改善和完善Kubernetes环境，以确保其能够满足您的业务需求。 它还可以帮助您重新访问以前的阶段，以便在迁移新应用或设置新环境时避免引入不必要的问题。

 您可能在成熟的这个阶段需要帮助，以确定可以在哪里进行改进。 在这里，对Kubernetes环境进行审核是一个很好的工具。 您也可以使用配置验证工具。 

退出第六阶段时，您将建立协议和程序，以便团队与系统保持一致的交互并了解优先级。 您还将对基础架构（如代码和CI）有深入的了解

***

### 学习资源

[Kubernetes: What Needs Monitoring and Why](https://www.fairwinds.com/kubernetes-monitoring-white-paper)

[Kubernetes Audit and Improve](https://www.fairwinds.com/kubernetes-audit-improve)

[Managing Kubernetes Configuration for Security, Reliability and Efficiency](https://www.fairwinds.com/manage-kubernetes-configuration-security-efficiency-reliability)

[Why Infrastructure as Code and Kubernetes](https://www.fairwinds.com/blog/why-infrastructure-as-code-kubernetes)

***

## 7. 优化&自动化

消除人为错误和辛劳，提高可靠性并最大化效率。

在该阶段，你将学习/精通以下内容：

使用更复杂的工具来消除人为错误和辛劳，提高可靠性并最大程度地提高效率。

***

### 必要条件

**改进和完善**

您正在衡量和跟踪Kubernetes部署，以确保其交付符合业务需求。

**建立协议**

您已经制定了协议，策略和过程，因此团队可以与系统和优先级保持一致的交互。

**IaC和CI/CD**

您对作为代码和CI / CD的基础架构有深刻的理解，有助于一次性更改。

***

### 介绍

随着Kubernetes完全成熟，您现在将专注于优化和自动化环境。这包括优化Kubernetes的成本和效率，尽可能自动化以及定期运行配置验证以检查错误。

***

### 优化

现在，您正在跟踪和测量，您将在仪表板中拥有数据。这将帮助您优化Kubernetes使其更加高效或可靠。您将进行一些细微的改变，从而产生很大的变化。如，您可以通过以下方式优化群集：

* 根据您的工作负载需求使用正确的实例类型 
* 扩展自定义指标与通用CPU使用率 
* 转向多地区以更有效地为全球交通服务 
* 跟踪和管理云支出成本
* 通过增加工作负载弹性来降低升级风险

您将永远不会停止优化集群。随着新数据的出现以及您的应用程序可与更多用户一起运行，您将需要不断查看仪表板并进行调整。这是成熟的最后阶段，很难做到。您需要一次解决一个问题以进行优化。

***

### 自动化

这是成熟的顶峰。到目前为止，您一直在手工做所有事情。在这里，您要自动化在前六个阶段中手动完成的所有操作。例如，您将：

* 查看您的基础架构即代码（IaC）以确保其牢固 
* 使用监视失败来重新启动或管理有问题和失败的资源
* 自动审核并标记配置错误或安全问题 
* 从生产中删除人员访问权限，转而使用服务帐户 
* 通过软件和工具构建，升级和备份系统和基础架构

***

### 配置验证

最后，您将使用CI / CD流程中内置的配置验证工具，以确保不会将安全性，可靠性和效率问题部署到生产中。使用按政策编码，您将能够：

* 通过在CI / CD阶段或作为准入控制器的Open Policy Agent（OPA）集成，自动化部署护栏和最佳安全实践。
* 在应用程序开发过程中自动进行问题检测，以防止错误从一开始就进入生产。
* 通过扫描容器中的漏洞并审核群集中的弱点，可以持续了解Kubernetes的安全状况。

***

### 挑战

您可能会面临以下挑战：

1. 从CI / CD到生产的政策管理 
2. 工作负载和其他Kubernetes资源使用资源效率低下，扩展不正确或性能不佳
3. 成本攀升或不为人知 
4. 可靠性问题仍然需要大量的人为干预和辛劳

***

### 输出

* 您将通过策略驱动的容器和Kubernetes配置验证来简化从开发到操作的交接，从而加强法规遵从性。
* 在CI / CD流程到生产过程中都包含策略执行，可防止错误引起安全性或可靠性挑战。
* 您正在使用OPA创建自定义策略。 
* 您可以深入研究Kubernetes的可靠性，效率和安全性实践。

***

### 学习资源

[Managing Kubernetes Configuration for Security, Reliability and Efficiency](https://www.fairwinds.com/manage-kubernetes-configuration-security-efficiency-reliability)

[Kubernetes Security, Reliability, Efficiency: It’s all about Configuration](https://www.fairwinds.com/blog/kubernetes-security-reliability-efficiency-its-all-about-configuration)

[Fairwinds Insights: CI Pipeline to Protect Your Kubernetes Clusters](https://www.fairwinds.com/blog/fairwinds-insights-continuous-integration-pipeline-to-protect-your-kubernetes-clusters)

[Managing OPA Policies with Fairwinds Insights](https://www.fairwinds.com/blog/managing-opa-policies-with-fairwinds-insights)

[Addressing Kubernetes Security Vulnerabilities with Policy Enforcement](https://www.fairwinds.com/blog/addressing-kubernetes-security-vulnerabilities-with-policy-enforcement)


