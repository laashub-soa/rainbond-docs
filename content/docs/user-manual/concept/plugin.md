---
title: "插件"
description: 介绍Rainbond插件的概念和设计思考
weight: 13
---

> TODO 完善

### 插件的定义

应用插件是标准化的为应用提供功能扩展，与应用共同运行的程序，例如：性能分析插件可以实时看到该服务的性能如何，吞吐率、响应时间以及在线人数等；网路治理插件则可以实现智能路由、A/B测试以及灰度发布等。

插件的运行环境与所绑定的服务一致，包括环境变量、持久化存储、网络空间等。

### 插件的分类

| 分类名称   | 作用                                                         | 控制             |
| ---------- | ------------------------------------------------------------ | ---------------- |
| 性能分析类 | 服务安装性能分析类插件即可显示性能分析页面                   |                  |
| 入口网络类 | ServiceMesh插件类型之一，负载处理服务下游流量                | 动态分配监听端口 |
| 出口网络类 | ServiceMesh插件类型之一，负载处理服务上游流量                | 支持服务发现     |
| 初始化类型 | 服务在插件先启动并正常退出后再启动，用户服务数据初始化或其他初始化操作。 |                  |
| 一般类型   | 比如数据备份、日志收集类                                     |                  |

#### 性能分析类

性能分析类插件通过旁路的方式分析服务的性能指标，或通过服务自身暴露性能指标由插件捕获。插件分析的性能指标参考 [服务性能分析](/docs/user-manual/app-service-manage/service-monitor/#服务性能分析)

Rainbond在每个计算节点提供了statsd服务接收性能分析插件的性能分析结果，并进行存储和展示。数据提供给其他功能比如自动伸缩参考。

#### 入口网络类

入口网络插件主要用于ServiceMesh网络或防火墙安全控制，比如，当我们部署了一个web应用后，我们不希望有不合法的请求（比如SQL注入）到达web中，这时我们可以为web应用安装一个安全插件，用来控制所有访问web的请求，就像是一个入口控制器，所以我们把这类插件叫做入口网络插件。

* 工作原理

当为某个应用安装了入口网络插件后，该插件被置于应用的前面，它必须监听一个由Rainbond分配的新端口用来拦截应用的所有请求，如下图中的8080端口，这时我们可以在插件内部对收到的请求进行必要的处理，然后把处理后的请求转发给应用监听的端口，如下图中的80端口。入口网络插件与应用的关系如下图所示：

![](http://grstatic.oss-cn-shanghai.aliyuncs.com/images/other/net-ingress-plugin.png)

插件需要从Rainbond应用运行时中动态发现配置，配置中包含了服务的实际端口和插件应该监听的端口。

#### 出口网络类

出口网络类插件是最常用插件之一，Rainbond会默认为具有依赖其他服务的服务自动注入此类插件。同时Rainbond也提供了基于envoy实现的默认网络治理插件。出口网络类插件主要提供当前服务访问上游服务时的治理需求。

* 工作原理

Rainbond应用运行时提供了XDS规范的服务和配置发现服务，支持支持envoy类型或其他支持此规范的插件类型。也可以通过获取原生插件标准配置信息自行生成插件自己的配置。

#### 初始化类

初始化类插件工作原理是基于 init-container，即初始化容器，它一般完成数据初始化工作，其工作性质必须是在有限的时间内正常退出。只有初始化插件正常退出后，业务容器才会启动。Rainbond基于此类插件完成多个服务的批量启动时的顺序控制, 参考 rainbond服务组件 [rbd-init-probe](<https://github.com/goodrain/rainbond/tree/master/cmd/init-probe>)
