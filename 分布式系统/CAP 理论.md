**CAP 理论**（又称 [CAP 定理](https://www.ibm.com/cn-zh/think/topics/cap-theorem)）指出，==分布式系统最多只能同时满足**一致性（Consistency）**、**可用性（Availability）**和**分区容错性（Partition Tolerance）**中的两项==。由于网络故障在分布式系统中是不可避免的（分区容错性 P 必须保证），因此系统在设计时必须在 **CP** 或 **AP** 之间做出权衡。 

## 核心三要素

- **一致性（Consistency, C）**：所有节点在同一时间具有完全相同的数据；客户端在任意节点读取操作，保证返回最新写入的数据。

- **可用性（Availability, A）**：每个请求都能收到一个非错误的响应，但不保证返回的数据是最新的。

- **分区容错性（Partition Tolerance, P）**：当网络发生通信故障（部分节点之间不连通）导致分区时，系统仍能继续运行。 

![[asssert/Pasted image 20260906011216.png]]