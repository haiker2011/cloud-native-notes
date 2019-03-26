# Knative 简介

Knative 的目标是在基于 Kubernetes 之上为整个开发生命周期提供帮助。它的具体实现方式是：首先使你作为开发人员能够以你想要的语言和以你想要的方式来编写代码，其次帮助你构建和打包应用程序，最后帮助你运行和伸缩应用程序。

Knative 是谷歌牵头发起的 serverless 项目，希望通过提供一套简单易用的 Serverless 开源方案，把 Serverless 标准化。

## Knative 定义

> 基于 kubernetes 平台，用于**构建**、**部署**和**管理**现代 serverless 工作负载。

## Knative 三大组件

### Build

* 在 kubernetes 上编排 source-to-url 的工作流程。
* 提供标准化可移植的方法。
* 定义和运行集群上的容器镜像构建。

### Serving

* 请求驱动计算，可以收缩到零
* 根据需求自动伸缩和调整工作负载
* 使用蓝绿部署路由和管理流量

<span id="fingure-2-2">*图 2-2 显示 Autoscaler 和 Activator 如何和 Routes 及 Revisions 协同工作。*</span>

<div align="center">
<img src="https://ws2.sinaimg.cn/large/006tKfTcly1g0yrmo1t2cj31z70u0afi.jpg" alt="Autoscaler and Activator with Route and Revision" />
Autoscaler 和 Activator 如何和 Routes 及 Revisions 互动。
</div>


> Autoscaler 如何伸缩

> Autoscaler 采用的伸缩算法针对两个独立的时间间隔计算所有数据点的平均值。它维护两个时间窗，分别是 60 秒和 6 秒。Autoscaler 使用这些数据以两种模式运作：Stable Mode (稳定模式) 和 Panic Mode (忙乱模式)。在 Stable 模式下，它使用 60 秒时间窗平均值决定如何伸缩部署以满足期望的并发量。

> 如果 6 秒窗口的平均并发量两次到达期望目标，Autoscaler 转换为 Panic Mode 并使用 6 秒时间窗。这让它更加快捷的响应瞬间流量的增长。它也仅仅在 Panic Mode 期间扩容以防止 Pod 数量快速波动。如果超过 60 秒没有扩容发生，Autoscaler 会转换回 Stable Mode。


### Eventing

* 管理和交付事件
* 将服务绑定到事件
* 对发布/订阅细节进行抽象
* 帮助开发人员摆脱相关负担

## Knative 分析

## Knative 未来发展

### 性能问题

### 

## 和机器学习的联姻

## 总结


## 参考文献

1. [Knative: 重新定义Serverless](https://skyao.io/talk/201811-knative-redefine-serverless/)
2. [servicemesher/getting-started-with-knative](https://github.com/servicemesher/getting-started-with-knative)