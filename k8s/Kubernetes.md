### Kubernetes 

[TOC]



#### 1.云原生概念

#### 2.容器基本概念

#### 3.Kubernetes基本概念

- 安装MiniKube  https://developer.aliyun.com/article/221687
- 安装kubectl [https://kubernetes.io/zh/docs/tasks/tools/install-kubectl-linux/]

#### 4.Pod 

Pod 实现机制:

- 共享网络

- 共享存储

  

容器设计模式 SideCar

- 1.Init Container
- 2.Proxy Container
- 3.Adapter Container



总结:

- Pod 是Kubernetes 实现 容器设计模式的核心机制
- 容器设计模式是Google Borg 的大规模容器集群管理最佳实践
  - 也是Kubernetes 进行复杂应用编排的基础依赖之一
- 所有设计模式的本质都是解耦和重用



#### 5. 应用与编排-核心原理

**Kubernetes 资源对象**

- Spec
- Status
- Metadata
  - Labels
  - Annotation
  - OwnReference

**控制器模式**

- 控制循环

  ​	控制器模式的核心,控制循环包括了**控制器,被控制的系统,以及能够观测系统的传感器.**

  ​	外界通过修改资源spec来控制资源,控制器比较资源spec和status,从而计算出diff,根据diff来决定执行对系统进行怎样的控制操作.控制操作会产生新的输出,被传感器以资源status的形式上报,控制器的各个组件都会是独立自主的运行,根据资源的status进行调整下一步的操作,不断的趋近spec终态.

  

- Sensor

  控制循环中逻辑传感器由以下三个组件构成

  - Reflector

  - Informer

  - Indexer

    ​	Reflector 通过List 和Watch K8s server 获取资源数据.List 在Controller 重启以及Watch 中断的情况下,进行系统的全量更新;而Watch 则在多次List 之间进行增量的资源更新.Reflector 在获取新的资源数据后,会在Delta 队列中塞入一个包括资源对象信息本身以及资源对象事件类型的Delta 记录,Delta队列中可以保证同一个对象在队列中仅有一条记录,避免Reflector 重新List 和Watch 的时候产生重复的记录.

    ​	Informer 组件不断从Delta队列中弹出delta 记录,然后把资源对象交给Indexer, 让indexer 把资源记录在缓存中,缓存默认设置为用资源命名空间作为索引,可以背Controller Manager 或多个Controller 所共享.再把事件交给事件的回调函数

- 控制循环的控制器组件

  ​	主要有两部分组成,分别是:

  - 事件处理函数

  - worker 

    事件处理函数会关注资源的新增,修改,删除的事件,并根据控制器的逻辑决定是否需要进行处理.对需要处理的事件,会把事件关联的资源空间和名字塞入一个工作队列中,由后续的work池中的一个work进行操作,对于工作队列会进行去重操作,避免多个work 处理同一个资源

- 总结:

  - Kubernetes 所采用的控制器模式,是由声明式API驱动.或者说是基于对Kubernetes 资源对象修改驱动的

  - Kubernetes 资源背后是关注该资源的控制器,控制器将驱动异步控制系统走向设置的最终状态

  - 控制器是自主运行的,可以实现自动化和无人值守

  - Kubernetes 的控制器和资源都是可以自定义的,可以方便的扩展控制器模式.特别对于有状态应用,我们往往通过自定义资源和控制器的方式来自动化运维操作.




#### 6.编排与管理-Deployment

- Deployment: 管理部署发布的控制器



查看Deployment 状态

```
kubectl get deployment
```



- Deployment 只负责管理不同版本的ReplicaSet,由ReplicaSet 管理Pod副本数

- 每个ReplicaSet 对应了一个Deployment template 的一个版本
- 一个ReplicaSet 下面的Pod 都是相同版本

**Deployment 控制器**

**ReplicaSet 控制器**



#### 7.编排与管理-Job与DaemonSet

- Job 管理任务的控制器

  新的参数:

  - restartPolicy 重启策略
  - backoffLimit 重试次数

**查看Job 状态**

```
kubectl get jobs
```

- 并行运行job

  spec中新增如下参数:

  - completions:代表pod 执行的次数
  - parallelism:代表并行执行的pod 个数

- CronJob

  - schedule: crontab 格式相同
  - startingDeadlineSeconds: 最长执行时间
  - concurrencyPolicy: 是否允许并行
  - successfulJobHistoryLimit:运行留存历史Job 个数

**Job-管理模式**

- Job Controller 负责根据配置创建pod
- Job Controller 跟踪Job 状态,根据配置及时重试或者创建pod
- Job Controller 会自动添加label 跟踪pod,并根据配置并行或者串行创建pod

**Job 控制器**



**DaemonSet:守护进程控制器**

- 保证集群内每一个(或一些)节点都运行一组相同的 Pod
- 跟踪集群节点状态,保证新加入的节点自动创建对应的Pod
- 跟踪集群节点状态,保证删除对节点删除对应的Pod
- 跟踪Pod 状态,保证每个Pod 处于运行状态

DaemonSet 适用场景:

- 集群存储进程: glusterd,Ceph
- 日志收集进程:fluentd,logstash
- 需要在每个节点运行的监控收集器



查看DaemonSet 

```
kubectl get ds
```

更新DaemonSet

- RollingUpdate:默认更新策略,当更新模版后,老旧的Pod 模版会被删除,然后创建新的Pod,可以配合健康检查做滚动更新
- OnDelete:更新DaemonSet 模版后,只有手动的删除对应的Pod,此节点的Pod 才会更新.



**DaemonSet -管理模式**

- DaemonSet Controller 负责根据配置创建Pod
- DaemonSet Controller 跟踪Job状态,根据配置及时重试或创建Pod
- DaemonSet Controller 会自动添加affinity&label 跟踪对应的Pod,根据配置在每个节点或者适合的节点创建Pod



**DaemonSet-控制器**



#### 8.应用配置管理

- **ConfigMap**

- **Secret**

- **ServiceAccount**

  解决集群身份认证问题

- **Resource**

  容器资源配置管理

- **SecurityContext**

- **InitContainer**



#### 9.应用存储和持久化数据卷-核心知识

- Volume
  - Persistent Volumes
  - Persistent Volume Claim
  - Static Volume Provisioning



#### 10.应用存储和持久化数据卷-存储快照与拓扑调度

- Kubernetes CSI Snapshot Controller
- 存储拓扑调度问题 
  - Local PV 只能在指定的Node 上被Pod 使用 (PV 有位置限制[.spec.nodeAffinity], 当Pod 从原来Node1 迁移到Node2 时就会产生问题) 
  - Pod 与PV 不在同一可用区(Zone)

- 处理流程
  - Volume Snapshot 处理流程
  - Volume Topology-aware Scheduling 处理流程


#### 11.可观测性

- Liveness 

- Readiness

- 应用健康诊断方式

  - 探测方式

    - httpGet .

      发送Http请求,返回结果在200-399之间状态码则表明容器健康

    - Exec. 

      通过命令来检查服务是否正常,命令返回0则表示容器健康

    - tcpSocket

      通过容器的IP 和Port执行TCP检查,如果能建立起TCP连接则表示容器健康

  - 诊断结果

    - Success. Container通过了检查
    - Failed.     Container未通过检查
    - Unknown. 未能执行检查,不做任何操作

  - 重启策略

    - Always 总是重启
    - OnFailure 失败才重启
    - Never 从不重启

- 应用健康故障排除

  - 通过describe 查看状态,通过状态判断排查方向
  - 查看对象event事件,查看更详细的信息
  - 查看pod 日志确定应用自身情况

- 常见应用异常

  - Pod停留在Pending

  Pending 表示调度器没有介入,可以通过`kubectl describe pod`,查看时间排查问题,通常和资源使用相关.

  - Pod 停留在Waiting

  一般表示Pod 的镜像没有正常的拉取,通常可能和私有镜像拉取,镜像地址不存在,镜像公网拉取相关.

  - Pod 不断被拉起且可以看到crashing

  通常表示Pod 已经完成调度并启动,但是启动失败,通常是由配置、权限造成,需要查看Pod 日志.

  - Pod处于running 但是没有正常工作

  通常是由于部分认证字段拼写错误造成,可以通过校验部署来排查, `kubectl apply -validate -f pod.yaml`

  - Service 无法正常工作

  排除网络自身的问题之外,最有可能的问题是label配置有问题,可以通过查看endpoint 的方式进行检查.

- 启用远程应用调试

  - 代理本地应用到集群 - Telepresence
  - 代理远程应用到本地 - Port-Forward
  - 开源调试工具 - kubectl-debug



#### 12.可观测性-监控与日志

- 监控
  - 资源监控
  - 性能监控
  - 安全监控
  - 性能监控

- 日志
  - 主机内核日志
  - runtime日志
  - 核心组件日志
  - 部署应用日志
- 日志采集
  - 宿主机文件
  - 容器内文件
  - 容器标准/错误输出
- fluentd 日志采集方案



#### 13.Kubernetes 网络概念与策略

- 基本网络模型

  - 三个基本条件
    - Pod 与Pod 之间直接通信,无需显式使用NAT
    - Node 与Pod 之间直接通信,无需显式使用NAT
    - Pod 可见的IP地址确为其他Pod 与其通信时所用,无需显式转换
  - 四大目标
    - 容器与容器之间对通信
    - Pod 与Pod 之间的通信
    - Pod 与Service 之间的通信
    - 外部世界与Service 的通信

- Network Namespace (Netns)

  - 实现内核网络虚拟化的基础,创建了隔离的网络空间
  - 独立的附属网络设备 (lo、veth等虚拟设备或者物理网卡)
  - 独立的协议栈,IP地址和路由表
  - iptables规则
  - ipvs等

- Pod 与Netns关系

  - 每个Pod 拥有独立的Netns空间,Pod 内的Container 共享该空间,通过共享等Pod-IP对外提供服务
  - 宿主机上还有Root Netns,可以看作一个特殊的容器空间

- 常见的网络实现方案

  - Flannel,最为普遍的实现,提供多种网络backed实现,覆盖多种场景
  - Calico,采用BGP提供网络直连
  - Canal(Flannel for Network + Calico for firewalling) ,嫁接型创新项目
  - Cilium,基于eBPF + XDP的高性能Overlay网络方案
  - Kube-route,同样采用BGP提供网络直连,集成机遇LVS的负载均衡能力
  - Romana,采用BGP or OSPF提供网络直连能力的方案
  - WeaveNet,采用UDP封装实现L2 Overlay,支持用户态(慢,可加密)/内核态(快,不能加密)两种实现

- Network Policy

  基于策略的网络控制,用于隔离应用并减少攻击面.使用标签选择器模拟传统的分段网络,并通过策略控制他们的之间的流量以及外部流量

  使用之前要注意:

  - api server 开启extensions/v1beat1/networkpolicies

  - 网络插件要支持Network Policy,如Calico、Romana、WeaveNet、trireme等

  - 配置实例

    通过标签选择器(包括namespaceSelector 和PodSelector)来控制Pod流量

    - 控制对象 

    通过spec字段,podSelector筛选

    - 流方向

    Ingress(入Pod 流量) + from、Egress(出Pod 流量) + to

    - 流特征

    对端(通过name/PodSelector)、IP端(idBlock)、协议(Protocol)、端口(Port)

[参考资料]

[1].https://edu.aliyun.com/roadmap/cloudnative

[2].https://time.geekbang.org/column/intro/116

[3].https://portworx.com/ (#1 Kubernetes Storage Platform)

