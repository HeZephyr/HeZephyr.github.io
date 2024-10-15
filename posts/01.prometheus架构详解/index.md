# Prometheus架构详解

## Prometheus简介

Prometheus 是一个开源的系统监控报警工具套件，它最初由SoundCloud开发，并于2016年成为CNCF（云原生计算基金会）托管的第二个项目（第一个是kubernetes）。Prometheus 以其简单高效的方式收集指标而闻名，能更好地与容器平台、云平台配合，这使得它在现代云原生环境中非常受欢迎。Prometheus 被广泛应用于各种场景中，包括但不限于：

- **应用性能监控**：监控应用程序的健康状态和性能指标。
- **基础设施监控**：监控服务器、存储设备、网络设备等基础设施的状态。
- **业务监控**：监控业务层面的关键性能指标（KPIs）。

Prometheus的主要优势如下：

- **独立性**：Prometheus 服务器本身是一个独立的二进制文件，易于部署和管理。
- **多维度数据模型**：数据以多维键值对的形式存储，便于灵活查询。
- **无中介**：Prometheus 通过HTTP协议直接抓取被监控系统的度量信息，不需要中间代理。
- **丰富的数据存储**：Prometheus 内置的时间序列数据库（TSDB）能够高效存储大量的时间序列数据。
- **强大的查询语言**：PromQL 提供了强大的查询功能，支持复杂的聚合操作。
- **灵活的告警机制**：通过Alertmanager 可以配置复杂的告警规则并处理告警消息。
- **插件化设计**：支持各种Exporter，能够轻松集成到现有系统中。

## 架构

Prometheus的架构设计旨在提供一个简单、高效且可扩展的监控解决方案，核心组件包括Prometheus Server、Exporters、PushGateway、Service Discovery、Alertmanager以及数据可视化工具（如Grafana）等。其整体架构如下图所示：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/018f226650834e569b66ecfffe60623f.png)


1. 客户端

   - **Short-lived jobs**：对于那些生命周期较短的任务，可能在Prometheus来pull之前就消失了。Prometheus提供了PushGateway机制。客户端（或服务端）安装官方的PushGateway，这些任务可以将它们的监控数据组织成Key-Value形式并推送到PushGateway，然后Prometheus Server会定期从PushGateway拉取数据。这里需要注意的是PushGateway不一定要安装在被监控端，也可以安装在服务端，甚至是一台不相关的主机上，换句话来说，它只是一个中间转发的媒介，确保即使是在任务结束之后，Prometheus也能获取到必要的监控数据。
   - **Jobs supported by Prometheus**：Prometheus支持多种服务（如cAdvisor, Kubernetes, Etcd, Gokit等）直接向Prometheus暴露监控数据。这些服务通常被称为“Jobs”，它们通过内置的端点直接与Prometheus交互，无需额外的中间层。
   - **Exporter**：Exporter是Prometheus生态系统中的重要组成部分。它们通常作为中间层运行在被监控系统旁边，负责从不同的后端系统中提取度量数据，并将其转换为Prometheus可以理解的格式。这样，即使原有的监控目标不直接支持 Prometheus，也可以通过 Prometheus 提供的 Client Library 编写该监控目标的监控采集程序。例如，可以通过 Node Exporter 来监控 Linux 或 Windows 主机的硬件资源使用情况。常用的 Exporter 包括 Mysql Exporter、JMX Exporter、Consul Exporter 等。

2. Prometheus Server

   Prometheus Server是Prometheus组件中的核心部分，负责实现对监控数据的获取，存储以及查询。 

   * **数据获取 (Data Retrieval)**：Prometheus Server 可以通过静态配置的方式来管理其监控的目标，同时也支持结合 Service Discovery 动态地发现和管理监控目标。这种方式允许 Prometheus Server 在网络中自动识别服务实例，并从这些实例中拉取监控数据。
   * **数据存储 (Data Storage)**：Prometheus Server 内置了时序数据库功能，能够将采集到的监控数据按时间序列的形式存储在本地磁盘上。此外，Prometheus 社区也提供了与外部时序数据库集成的方案，以满足特定场景下的存储需求。
   * **数据查询 (Data Querying)**：Prometheus Server对外提供了自定义的PromQL语言，实现对数据的查询以及分析。

3. Server discovery

   Prometheus为了适应动态变化的环境，提供了服务发现机制来自动管理监控目标列表。在现代云原生环境中，资源使用方式通常是按需的，这意味着没有固定的监控目标，所有的监控对象（包括基础设施、应用和服务）都在动态变化。对于 Prometheus 这一类基于 Pull 模式的监控系统，显然无法继续使用`static_configs`的方式静态地定义监控目标。因此，Prometheus 引入了一个中间的代理人（即服务注册中心），这个代理人掌握着当前所有监控目标的访问信息，Prometheus 只需要向这个代理人询问有哪些监控目标即可。这种模式被称为服务发现。

   Prometheus 支持多种服务发现机制，包括但不限于以下几种：

   - **Consul**：Consul 是一种服务网格工具，可以用来发现和监控服务。Prometheus 可以集成 Consul 来动态发现服务实例，在微服务架构的应用程序中，Consul 常被用作服务发现注册软件，Prometheus 与其集成从而动态发现需要监控的应用服务实例。
   - **DNS SD**：DNS 服务发现是一种基于 DNS 的服务发现机制，适用于那些使用 DNS 解析服务位置的环境。
   - **Kubernetes SD**：在 Kubernetes 这类容器管理平台中，Kubernetes 掌握并管理着所有的容器以及服务信息，Prometheus 只需要与 Kubernetes 打交道就可以找到所有需要监控的容器以及服务对象。
   - **EC2 SD**：在 AWS 的 EC2 环境中，Prometheus 可以使用 EC2 服务发现来自动发现和监控运行中的实例。

4. AlertManager

   AlertManager 是 Prometheus 架构中的另一个关键组件，负责处理来自 Prometheus Server 的警报。支持基于PromQL创建告警规则，当 Prometheus Server 检测到告警规则被触发时，会产生警报，并将这些警报发送给 AlertManager。AlertManager 负责汇总、去重、抑制和路由这些警报，确保警报信息能够及时、准确地通知到相关人员或系统。

   - **汇总与去重**：AlertManager 会汇总来自多个 Prometheus Server 的警报，并去除重复的警报，以减少无效的通知。
   - **抑制 (Silencing)**：管理员可以设置抑制规则，临时屏蔽特定的警报，避免在维护窗口期间产生不必要的警报。
   - **路由 (Routing)**：AlertManager 支持基于警报标签的路由策略，可以根据预定义的规则将警报发送给不同的接收者或接收组，确保警报能够发送给最合适的人或系统进行处理。
   - **通知 (Notification)**：AlertManager 支持多种通知方式，包括电子邮件、PagerDuty、OpsGenie、WeChat Work 等，也可以通过Webhook自定义告警处理方式，以便快速响应警报。

   Prometheus 自带的 AlertManager 模块可以与诸如 PagerDuty 这样的商业化服务集成，实现警报和邮件的发送功能。然而，在国内使用 AlertManager 结合 PagerDuty 可能会遇到一些不便。因此，通常的做法是将Prometheus的数据集成到 Grafana 中进行展示，并在 Grafana 中配置警报，以实现更简便的本地化警报管理。

5. 监控数据可视化

   监控数据可视化是监控系统不可或缺的一部分，它帮助用户直观地理解系统的健康状况和性能表现。Prometheus Web UI是Prometheus内置的一个可视化管理界面，通过Prometheus UI用户能够轻松的了解Prometheus当前的配置，监控任务运行状态等。但在生态系统中常用的数据可视化工具是 Grafana，它是一个开源的度量仪表板和可视化工具，支持多种数据源，包括 Prometheus，使得用户可以轻松地创建图表和仪表板来展示监控数据。Grafana 不仅限于数据展示，还可以配置警报。用户可以在 Grafana 中设置基于 Prometheus 数据的警报规则，当监控数据达到预设的阈值时，可以触发警报。Grafana 可以通过多种渠道发送警报通知，包括邮件、短信、Webhook 等，确保问题能够被及时发现并处理。

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/01.prometheus%E6%9E%B6%E6%9E%84%E8%AF%A6%E8%A7%A3/  

