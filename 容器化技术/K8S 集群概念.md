K8s是容器编排平台，用来自动化容器部署、扩缩容、故障自愈。核心概念：

1. **Pod**：最小调度单元，一个Pod可包含1个或多个容器，共享网络/存储，Pod生命周期短暂，会被重建。
2. **Node**：节点，分Master控制节点、Worker工作节点；Worker真正跑Pod。
3. **Deployment**：无状态应用控制器，管理Pod副本，支持滚动更新、回滚、副本扩缩。业务服务最常用。
4. **StatefulSet**：有状态应用控制器（MySQL、Redis），稳定网络标识、持久存储，有序启停。
5. **Service**：Pod访问抽象，PodIP会变，Service提供固定访问入口，内置负载均衡。类型ClusterIP/NodePort/LoadBalancer。
6. **ConfigMap**：配置文件，存放非敏感配置，解耦代码与配置。
7. **Secret**：存放敏感信息，密码、密钥，相比ConfigMap做基础加密。
8. **Namespace**：命名空间，资源隔离，多环境/多项目资源分组。
9. **Ingress**：集群入口，七层HTTP路由，把外部流量转发到内部Service。
10. **Label & Selector**：标签，给资源打标记，通过标签筛选资源（Deployment管理Pod靠标签）。
11. **Volume**：存储卷，容器销毁数据不丢失，挂载到Pod。

一句话总结：Pod跑容器，Deployment管理Pod，Service给固定访问地址，Ingress接入外网，ConfigMap/Secret管理配置。

  

## 技巧

1. 优先讲 **Pod、Deployment、Service、Ingress、ConfigMap、Secret**，剩下简要带一句，后端面试这几个是高频。
2. 贴合业务：我们供应链后端服务部署用Deployment，配置放ConfigMap，数据库用StatefulSet。
3. 区分：Pod≠容器；Deployment用于无状态，数据库这类有状态用StatefulSet。

  

## ❌忌讳

1. 说Pod就是容器（Pod是容器封装单元）
2. 混淆Service和Ingress：Service四层，Ingress七层http路由。
3. 以为Secret强加密，Secret只是base64编码，不是高强度加密。

## 高频追问

### 追问1 Deployment滚动更新原理？

**回答**

滚动更新会新建版本Pod，等新Pod就绪，再逐步销毁旧Pod，保证服务不中断；可配置maxSurge最大超量副本、maxUnavailable最大不可用副本。失败支持回滚。


### 追问2 Service ClusterIP原理？

**回答**
ClusterIP是集群内部访问，基于iptables/ipvs，做负载均衡，自动把请求转发到后端符合标签的Pod。

### 追问3 无状态 vs 有状态怎么区分？

**回答**

- 无状态：实例完全对等，随便扩缩、重建，比如Java后端服务，Deployment。
- 有状态：实例有唯一标识、数据持久化，实例不能随便互换，MySQL、Redis，StatefulSet。