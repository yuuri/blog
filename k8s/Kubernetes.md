### Kubernetes 



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





#### 5.Kubernetes 资源对象

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

    





[参考资料]

[1].https://edu.aliyun.com/roadmap/cloudnative



