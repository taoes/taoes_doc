## 1、基础概念

1. **ES**：基于 Lucene 分布式全文检索引擎，底层倒排索引。适合日志、商品搜索；**不适合强事务**，不能替代 MySQL。
2. **正向索引**：文档→词条。**倒排索引**：词条→文档 ID 列表，词典 + 倒排表，全文检索快。
3. Index（索引≈库）、Document（JSON 文档，一条记录）、Field（字段）。ES7 废弃 Type，只剩虚拟`_doc`。
4. **主分片 primary**：存真实数据，创建索引时确定，**数量不可改**。副本 replica：主分片拷贝，故障转移、分担查询；副本不能和主分片同节点。
5. **NRT 近实时**：写入不是立刻可搜，依赖 refresh，默认 1s。
6. **refresh / translog / flush**
	- refresh：内存 buffer 生成可检索 segment，**不落地磁盘**
	- translog 事务日志：宕机防丢数据
	- flush：segment 刷磁盘，清空 translog
    
写入流程：写 buffer+translog → refresh → flush

7. **segment 段**：不可变。删除只是标记删除。后台**段合并 merge**，合并小段、清理删除标记；消耗 CPU、IO。

## 二、读写原理

1. **写入流程**
    
    客户端→协调节点 → hash (_id)% 主分片数路由到主分片节点 → 主分片写内存 + translog → 同步副本，副本 ack 返回结果 → refresh 可检索，flush 落盘。
2. **检索流程**
    
    请求到协调节点 → 分发查询到相关分片 → 分片返回文档 ID + 打分 → 协调节点合并、排序分页，返回结果。
3. **协调节点**：接收请求、路由、合并结果，**不存数据**，任意节点都能充当。

## 三、分词器

1. Analyzer 三部分：字符过滤器预处理 → Tokenizer 切词 → TokenFilter（小写、停用词、同义词）。
2. IK 中文分词：`ik_max_word`细粒度；`ik_smart`粗粒度。默认 standard 分词中文单字切割，效果差。

## 四、集群

1. 节点角色
- Master：维护集群元数据，**不处理数据读写**
- Data：存放分片，执行读写聚合
- Ingest：写入前数据预处理

2. **脑裂**：网络故障集群分裂，各自选主，两边独立写入造成数据不一致。ES7 Zen2 自动防脑裂。
3. 集群三色
    Green：主 + 副本分片全部正常    
    Yellow：主分片正常，副本分片未分配（可读，有风险）
    Red：部分主分片不可用，索引读写异常

## 五、查询 & 打分

1. 打分：ES5 后默认**BM25**，限制词频过高权重爆炸，优于 TF-IDF。
2. `term` vs `match`
- term：**不分词，精确匹配**。text 字段分词存储，term 查 text 经常查不到。
- match：对搜索词分词，适合全文检索。

1. text vs keyword
    text：分词，全文检索，**不能排序聚合**
    keyword：不分词，精确匹配，可排序、聚合。常用`name`(text)，`name.keyword`(keyword)
    
2. 深分页 from+size：分片都要取出大量数据，内存爆炸。
    方案：浅分页 from+size；
    深分页用**search_after**；
    scroll 适合大批量导出，不适合前端翻页。
3. bool 查询 4 个子句
    
    - must：必须满足，参与打分
    - filter：必须满足，**不打分，可缓存，优先放过滤条件**
    - should：可选，满足加分
    - must_not：排除，不打分

## 六、聚合

1. 聚合两类：Bucket 桶聚合（分组，terms/range）；Metric 指标聚合（sum/avg/max/cardinality）
2. cardinality 基数：基于 HyperLogLog++，**估算值，非精确去重**。

## 七、优化

1. 写入优化：bulk 批量写入；导入数据调高 refresh_interval、临时副本设 0；合理分片；业务低谷执行段合并。
2. 查询优化：filter 代替 must；避免深分页；keyword 做聚合排序；减少返回字段，按需开启_source。
3. 分片不是越多越好。单分片推荐 20~50G。分片过多，元数据和查询开销变大。

## 八、高频坑题

1. _source：存储原始 JSON。关闭节省磁盘，但无法 update、reindex，谨慎关闭。
2. 更新文档：文档不可变，更新是标记旧文档删除，新增文档，大量更新加重段合并压力。
3. reindex：索引数据迁移。**主分片数不能直接改，要改分片只能 reindex 重建索引**。
4. Yellow 常见原因：副本分片无法分配，比如单节点集群不能放副本。

## 九、Java 客户端

TransportClient（ES7 废弃）；High Level REST Client（ES7 主流）；新 ES8 官方 Elasticsearch Java Client。

## 十、面试追问储备

- ES 不能替代 MySQL：不支持事务，一致性弱。
- 海量日志：按天拆分滚动索引。
- 线上故障案例：段合并 CPU 飙升、分片无法分配、大聚合查询超时。
