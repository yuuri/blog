# 1.kubernetes 创建ServiceAccount 后没有生成对应的secret

>
>
>从 1.24 开始就不会自动生成 secret 了，[chanagelog 在这里.](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.24.md#urgent-upgrade-notes)
>
>内容如下 LegacyServiceAccountTokenNoAutoGeneration 功能门是测试版，默认启用。启用后，不再为每个 ServiceAccount 自动生成包含服务帐户令牌的 Secret API 对象。使用 [TokenRequest API](https://kubernetes.io/zh-cn/docs/reference/kubernetes-api/authentication-resources/token-request-v1/) 获取服务帐户令牌，或者如果需要未过期的令牌，请按照[本指南](https://kubernetes.io/docs/concepts/configuration/secret/#service-account-token-secrets)为令牌控制器创建一个 Secret API 对象以填充服务帐户令牌
>
>pr: https://github.com/kubernetes/kubernetes/pull/108309



参考:

[1].https://www.soulchild.cn/post/2945/





# 2.默认的nfs-storage-class 不可用

报错中可以看到kubernetes 移除了SelfLink 相关的支持,由于nfs-client-provisioner 停止维护，导致了相关的报错

需要将对应的nfs StorageClass由 nfs-client-provisioner 替换为 nfs-subdir-external-provisioner



storageclass 相关镜像地址可以使用

`m.daocloud.io/registry.k8s.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2`

参考:

[1].https://github.com/kubernetes/enhancements/blob/master/keps/sig-api-machinery/1164-remove-selflink/README.md

[2].https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner





# 3. kubesphere未能正确识别调度节点

原因：

https://github.com/kubernetes/enhancements/blob/master/keps/sig-cluster-lifecycle/kubeadm/2067-rename-master-label-taint/README.md#renaming-the-node-rolekubernetesiomaster-node-label