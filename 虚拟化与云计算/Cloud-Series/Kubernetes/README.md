![](https://i.postimg.cc/cC1YKzNS/image.png)

# Kubernetes 部署与实战

Kubernetes 的官方定义为：Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications.It groups containers that make up an application into logical units for easy management and discovery.

![K8s 概念图](https://i.postimg.cc/BQSNR1yd/image.png)

Kubernetes [koo-ber-nay'-tice] 是 Google 基于 Borg 开源的容器编排调度引擎，其支持多种底层容器虚拟化技术，具有完备的功能用于支撑分布式系统以及微服务架构，同时具备超强的横向扩容能力；它提供了自动化容器的部署和复制，随时扩展或收缩容器规模，将容器组织成组，并且提供容器间的负载均衡，提供容器弹性等特性。作为 CNCF（Cloud Native Computing Foundation）最重要的组件之一，可谓云操作系统；它的目标不仅仅是一个编排系统，而是提供一个规范，可以让你来描述集群的架构，定义服务的最终状态。

# 从 Docker 到 K8s

这里的 Docker 指的是 Docker engine（也叫做 Docker daemon，或最新的名字：Moby），它是一种容器运行时（container runtime）的实现，而且是最主流的实现，几乎就是容器业界的事实标准。Docker 是用来创建和管理容器的，它和容器的关系就好比 Hypervisor（比如：KVM）和虚拟机之间的关系。当然，Docker 公司对 Docker engine 本身的定位和期望不仅仅在于在单机上管理容器，所以近年来一直在向 Docker engine 中加入各种各样的高级功能，比如：组建多节点的 Docker 集群、容器编排、服务发现，等等。

而 Kubernetes，是搭建容器集群和进行容器编排的主流开源项目（由 Google 发起并维护），适合搭建 PaaS 平台。容器是 Kubernetes 管理的核心目标对象，它和容器的关系就好比 OpenStack 和虚拟机之间的关系，而它和 Docker 的关系就好比 OpenStack 和 Hypervisor 之间的关系。一般来说，Kubernetes 是和 Docker 配合使用的，Kubernetes 调用每个节点上的 Docker 去创建和管理容器，所以，你可以认为 Kubernetes 是大脑，而 Docker 是四肢。因此，在这样的背景下，Docker 逐渐式微，而 Kubernetes 崛起，就毫不奇怪了。

K8s 与 Docker 最典型而直接的差异，即 Docker 中是以容器为最小单元，而 K8s 中是以 Pod 为最小执行单元，下面我们就简要讨论下两种设计在实际环境中的差异。

## 为什么需要 Pod？

- 为什么 Kubernetes 使用 Pod 作为最小的可部署单元而不是单个容器？容器是现有实体，表示特定的事物，例如 Docker 容器。要管理容器，Kubernetes 需要其他信息，例如重启策略或实时探测。重新启动策略定义了终止容器时的处理方式。实时探针定义了一种操作，以从应用程序的角度检测容器中的进程是否仍处于活动状态，例如，Web 服务器是否响应 HTTP 请求。Kubernetes 架构师决定使用一个新实体 Pod 逻辑上包含（包装）一个或多个容器，而应将其作为一个实体进行管理，而不是使用其他属性来使现有事物过载。

- 为什么 Kubernetes 在一个 Pod 中允许多个容器？Pod 中的容器在“逻辑主机”上运行：它们使用相同的网络名称空间（相同的 IP 地址和端口空间），IPC 名称空间，并且可以选择使用共享卷。因此，这些容器可以有效地进行通信，从而确保数据的局部性。此外，Pod 允许将多个紧密耦合的应用程序容器作为一个单元进行管理。

因此，如果应用程序需要在同一主机上运行多个容器，为什么不使用这些容器中的所有内容制作一个容器？首先，将许多东西放到一个容器中可能会违反“每个容器一个进程”的原则。其次，为应用程序使用多个容器更易于使用，更透明，并允许将软件依赖关系解耦。同样，团队之间可以重复使用更多的细粒度容器。

## 使用案例

多容器 Pod 的主要目的是为主程序支持位于同一地点，受共同管理的帮助程序。在 Pod 中有一些使用辅助过程的一般模式：

- Sidecar 容器“帮助”主容器。例如，日志或数据更改监视程序，监视适配器等。例如，日志观察者可以由不同的团队构建一次，并可以在不同的应用程序中重复使用。边车容器的另一个示例是为主容器生成数据的文件或数据加载器。

- 代理，网桥，适配器将主容器与外部环境连接起来。例如，Apache HTTP 服务器或 Nginx 可以提供静态文件，并充当主容器中 Web 应用程序的反向代理，以记录和限制 HTTP 请求。另一个示例是一个帮助容器，该容器将请求从主容器重新路由到外部世界，因此主容器连接到本地主机以访问例如外部数据库，而没有发现任何服务。

虽然您可以在单个 Pod 中托管多层应用程序（例如 WordPress），但建议的方法是为每个层使用单独的 Pod。这样做的原因很简单：您可以独立地扩展层并在群集节点之间分布它们。

# 功能特性

Kubernates 是建立在扩展性的具备二次开发的功能层次丰富的体系化系统，首先其最核心的功能是管理容器集群，能管理容器化的集群（包括存储，计算），当然这个是建立在对容器运行时(CRI)，网络接口(CNI),存储服务接口（CSI/FV）的基础上；其次是面向应用(包括无状态/有状态,批处理/服务型应用)的部署和路由能力，特别是基于微服务架构的应用管理，具备了其服务定义和服务发现，以及基于 configmap 的统一配置能力；在基础资源（主要是抽象底层 IaaS 的资源）和应用层的抽象模型之上是治理层，包含弹性扩容，命名空间/租户，等。当然，基于其原子内核的基础能力，在 Kubernetes 的核心之上搭建统一的日志中心和全方位监控等服务是水到渠成的，CNCF 更是有其认定推荐。

## 功能

- 服务发现和负载平衡：Kubernetes 可以使用 DNS 名称或使用自己的 IP 地址暴露容器。如果容器的流量很高，Kubernetes 能够负载均衡并分配网络流量，以便部署稳定。

- 存储编排：Kubernetes 允许您自动安装您选择的存储系统，例如本地存储，公共云提供商等。

- 自动部署和回滚：您可以使用 Kubernetes 描述已部署容器的所需状态，并且可以以受控速率将实际状态更改为所需状态。例如，您可以自动化 Kubernetes 为您的部署创建新容器，删除现有容器并将所有资源用于新容器。

- 自动装箱：Kubernetes 允许您指定每个容器需要多少 CPU 和内存（RAM）。当容器指定了资源请求时，Kubernetes 可以更好地决定管理容器的资源。

- 自我修复：Kubernetes 重新启动失败的容器，替换容器，杀死不响应用户定义的运行状况检查的容器，并且在它们准备好服务之前不会将它们通告给客户端。

- 密钥和配置管理：Kubernetes 允许您存储和管理敏感信息，例如密码，OAuth 令牌和 ssh 密钥。您可以部署和更新机密和应用程序配置，而无需重建容器映像，也不会在堆栈配置中暴露机密。

## 优势

- 它的速度很快：在不停机的情况下持续部署新功能时，Kubernetes 是一个完美的选择。Kubernetes 的目标是以恒定的正常运行时间更新应用程序。它的速度通过您每小时可以运送的许多功能来衡量，同时保持可用的服务。以正确的方式使用 Kubernetes 可帮助 DevOps 即服务团队自动扩展应用程序并以零停机时间进行更新。

- 遵循不可变基础架构的原则：以传统方式，如果多个更新出现任何问题，您就没有任何记录显示您部署了多少更新以及发生了哪个错误。在不可变基础结构中，如果您希望更新任何应用程序，则需要使用新标记构建容器映像并进行部署，从而使用旧映像版本终止旧容器。通过这种方式，您将获得一份记录，并了解您所做的事情以及是否有任何错误; 您可以轻松回滚到上一个图像。

- 提供声明性配置：用户可以知道系统应该处于什么状态以避免错误。作为传统工具的源代码控制，单元测试等不能与命令式配置一起使用，但可以与声明性配置一起使用。

- 大规模部署和更新软件：由于 Kubernetes 具有不可变的声明性，因此扩展很容易。Kubernetes 提供了一些用于扩展目的的有用功能：

  - 水平基础架构缩放：在单个服务器级别执行操作以应用水平缩放。可以毫不费力地添加或分离 atest 服务器。
  - 自动扩展：根据 CPU 资源或其他应用程序指标的使用情况，您可以更改正在运行的容器数
  - 手动缩放：您可以通过命令或界面手动缩放正在运行的容器的数量
  - 复制控制器：复制控制器确保群集在运行条件下具有指定数量的等效窗格。如果存在太多 Pod，则复制控制器可以删除额外的 Pod，反之亦然。

- 处理应用程序的可用性：Kubernetes 检查节点和容器的运行状况，并在由于错误导致的盒中崩溃时提供自我修复和自动替换。此外，它在多个 Pod 之间分配负载，以便在意外流量期间快速平衡资源。

- 存储卷：在 Kubernetes 中，数据在容器之间共享，但如果 Pod 被杀死，则会自动删除卷。此外，数据是远程存储的，因此如果将 Pod 移动到另一个节点，数据将保留，直到用户删除为止。

## 不足

- 初始过程需要时间：创建新进程时，您必须等待应用程序开始，然后才能供用户使用。如果要迁移到 Kubernetes，则需要对代码库进行修改，以使启动过程更有效，这样用户就不会有糟糕的体验。

- 迁移到无状态需要付出很多努力：如果您的应用程序是群集或无状态的，则不会配置额外的 Pod，并且必须在应用程序中重新配置。

- 安装过程繁琐：如果您不使用 Azure，Google 或 Amazon 等任何云提供商，则很难在群集上设置 Kubernetes；不过在有 Rancher 这样的工具辅助下，安装效率也有了大幅度的提高。

# 链接

- https://draveness.me/ 系列 K8S 相关文章
- https://jimmysong.io/kubernetes-handbook/concepts/Pod-overview.html
