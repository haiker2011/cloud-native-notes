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

* Build

> 驱动构建过程的自定义 Kubernetes 资源。在定义构建时，您将定义如何获取源代码以及如何创建将运行源代码的容器镜像。

* Build Template

> 封装可重复构建步骤集合并允许对构建进行参数化的模板。

* Service Account

>允许对私有资源（如 Git 存储库或容器镜像库）进行身份验证。

1. Service Account

Example 1-1. knative-build-demo/secret.yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: dockerhub-account
  annotations:
    build.knative.dev/docker-0: https://index.docker.io/v1/
type: kubernetes.io/basic-auth
data:
  # 'echo -n "username" | base64'
  username: dXNlcm5hbWUK
  # 'echo -n "password" | base64'
  password: cGFzc3dvcmQK
```

Example 1-2. knative-build-demo/serviceaccount.yaml

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-bot
secrets:
- name: dockerhub-account
```

2. Build Resource

Example 1-3. knative-helloworld/app.go

```go
package main

import (
    "fmt"
    "log"
    "net/http"
)

func handlePost(rw http.ResponseWriter, req *http.Request) {
    fmt.Fprintf(rw, "%s", "Hello from Knative!")
}

func main() {
    log.Print("Starting server on port 8080...")
    http.HandleFunc("/", handlePost)
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

Example 1-4. knative-helloworld/Dockerfile

```dockerfile
FROM golang

ADD . /knative-build-demo
WORKDIR /knative-build-demo

RUN go build

ENTRYPOINT ./knative-build-demo
EXPOSE 8080
```

Example 1-5. knative-build-demo/service.yaml

```yaml
apiVersion: serving.knative.dev/v1alpha1
kind: Service
metadata:
  name: knative-build-demo
  namespace: default
spec:
  runLatest:
    configuration:
      build:
        serviceAccountName: build-bot
        source:
          git:
            url: https://github.com/gswk/knative-helloworld.git
            revision: master
         template:
           name: kaniko
           arguments:
           - name: IMAGE
             value: docker.io/gswk/knative-build-demo:latest
      revisionTemplate:
        spec:
          container:
            image: docker.io/gswk/knative-build-demo:latest
```

3. Build Template

* Kaniko

> 在运行的容器中构建容器镜像，而不依赖于运行 Docker daemon 。

* Jib

> 为Java应用程序构建容器镜像。

* Buildpack

> 自动检测应用程序的运行时，并建立一个容器镜像使用 Cloud Foundry Buildpack。

Example 1-6. https://github.com/knative/build-templates/blob/master/kaniko/kaniko.yaml

```yaml
apiVersion: build.knative.dev/v1alpha1
kind: BuildTemplate
metadata:
  name: kaniko
spec:
  parameters:
  - name: IMAGE
    description: The name of the image to push
  - name: DOCKERFILE
    description: Path to the Dockerfile to build.
    default: /workspace/Dockerfile
  steps:
  - name: build-and-push
    image: gcr.io/kaniko-project/executor
    args:
    - --dockerfile=${DOCKERFILE}
    - --destination=${IMAGE}
```

### Serving

* 请求驱动计算，可以收缩到零
* 根据需求自动伸缩和调整工作负载
* 使用蓝绿部署路由和管理流量

<span id="fingure-2-2">*Autoscaler 和 Activator 如何和 Routes 及 Revisions 协同工作。*</span>

<div align="center">
<img src="https://ws2.sinaimg.cn/large/006tKfTcly1g0yrmo1t2cj31z70u0afi.jpg" alt="Autoscaler and Activator with Route and Revision" />
Autoscaler 和 Activator 如何和 Routes 及 Revisions 互动。
</div>

> Autoscaler 如何伸缩
>
> Autoscaler 采用的伸缩算法针对两个独立的时间间隔计算所有数据点的平均值。它维护两个时间窗，分别是 60 秒和 6 秒。Autoscaler 使用这些数据以两种模式运作：Stable Mode (稳定模式) 和 Panic Mode (忙乱模式)。在 Stable 模式下，它使用 60 秒时间窗平均值决定如何伸缩部署以满足期望的并发量。
>
> 如果 6 秒窗口的平均并发量两次到达期望目标，Autoscaler 转换为 Panic Mode 并使用 6 秒时间窗。这让它更加快捷的响应瞬间流量的增长。它也仅仅在 Panic Mode 期间扩容以防止 Pod 数量快速波动。如果超过 60 秒没有扩容发生，Autoscaler 会转换回 Stable Mode。

<span id="fingure-2-2">*Autoscaler 和 Activator 如何和 Routes 及 Revisions 协同工作。*</span>

<div align="center">
<img src="https://ws1.sinaimg.cn/large/006tKfTcly1g0yrpiumcqj31230u0jxo.jpg" alt="Knative Serving Object" />
Knative Serving 对象模型
</div>

#### 配置

示例 2-1. knative-helloworld/configuration.yml

```yaml
apiVersion: serving.knative.dev/v1alpha1
kind: Configuration
metadata:
  name: knative-helloworld
  namespace: default
spec:
  revisionTemplate:
    spec:
      container:
        image: docker.io/gswk/knative-helloworld:latest
        env:
          - name: MESSAGE
            value: "Knative!"
```

#### 路由

示例 2-4. knative-helloworld/route.yml

```yaml
apiVersion: serving.knative.dev/v1alpha1
kind: Route
metadata:
  name: knative-helloworld
  namespace: default
spec:
  traffic:
  - configurationName: knative-helloworld
percent: 100
```

#### 服务

示例 2-8. knative-helloworld/service.yml

```yaml
apiVersion: serving.knative.dev/v1alpha1
kind: Service
metadata:
  name: knative-helloworld
  namespace: default
spec:
  runLatest:
    configuration:
      revisionTemplate:
        spec:
          container:
            image: docker.io/gswk/knative-helloworld:latest
```

### Eventing

* 管理和交付事件
* 将服务绑定到事件
* 对发布/订阅细节进行抽象
* 帮助开发人员摆脱相关负担

#### Source（源）

例3-1: knative-eventhing-demo/app.go

```go
package main

import (
    "fmt"
    "io/ioutil"
    "log"
    "net/http"
)

func handlePost(rw http.ResponseWriter, req *http.Request) {
    defer req.Body.Close()
    body, _ := ioutil.ReadAll(req.Body)
    fmt.Fprintf(rw, "%s", body)
    log.Printf("%s", body)
}
func main() {
    log.Print("Starting server on port 8080...")
    http.HandleFunc("/", handlePost)
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

例3-2： knative-eventing-demo/service.yaml

```yaml
apiVersion: serving.knative.dev/v1alpha1
kind: Service
metadata:
  name: knative-eventing-demo
spec:
  runLatest:
    configuration:
      revisionTemplate:
        spec:
          container:
            image: docker.io/gswk/knative-eventing-demo:latest
```

例3-3: knative-eventing-demo/serviceaccount.yaml

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: events-sa
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: null
  name: event-watcher
rules:
- apiGroups:
  - ""
  resources:
  - events
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: null
  name: k8s-ra-event-watcher
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: event-watcher
subjects:
- kind: ServiceAccount
  name: events-sa
  namespace: default
```

例3-4: knative-eventing-demo/source.yaml

```yaml
apiVersion: sources.eventing.knative.dev/v1alpha1
kind: KubernetesEventSource
metadata:
  name: k8sevents
spec:
  namespace: default
  serviceAccountName: events-sa
  sink:
    apiVersion: eventing.knative.dev/v1alpha1
    kind: Channel
    name: knative-eventing-demo-channel
```

#### Channel（通道）

例3-5: knative-eventing-demo/channel.yaml

```yaml
apiVersion: eventing.knative.dev/v1alpha1
kind: Channel
metadata:
  name: knative-eventing-demo-channel
spec:
  provisioner:
    apiVersion: eventing.knative.dev/v1alpha1
    kind: ClusterChannelProvisioner
    name: in-memory-channel
```

#### Subscriptions（订阅）

<span id="fingure-2-2">*事件传递示意图*</span>

<div align="center">
<img src="https://wx1.sinaimg.cn/large/72bc86eagy1g0yx7pxes1j20q30ao74u.jpg" alt="事件传递示意图" />
事件传递示意图
</div>


例3-6: knative-eventing-demo/subscription.yaml

```yaml
apiVersion: eventing.knative.dev/v1alpha1
kind: Subscription
metadata:
  name: knative-eventing-demo-subscription
spec:
  channel:
    apiVersion: eventing.knative.dev/v1alpha1
    kind: Channel
    name: knative-eventing-demo-channel
  subscriber:
    ref:
      apiVersion: serving.knative.dev/v1alpha1
      kind: Service
      name: knative-eventing-demo
```

## Knative 分析

## Knative 未来发展

### 性能问题

* 0到1的伸缩
* 网络链路过长

### 替换 Queue Proxy 为 Istio 的 sidecar 

### 替换 Autoscaler 的实现为 k8s 的 HPA

### 更多的事件源和消息系统

## 和机器学习的联姻

## 总结


## 参考文献

1. [Knative: 重新定义Serverless](https://skyao.io/talk/201811-knative-redefine-serverless/)
2. [servicemesher/getting-started-with-knative](https://github.com/servicemesher/getting-started-with-knative)