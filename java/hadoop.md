## hadoop大数据平台架构与实践

掌握大数据存储与处理技术的原理
掌握hadoop的使用和开发能力

Google大数据技术
1、MapReduce
2、Bigtable
3、GFS

革命性的变化：
1、降低成本，PC机替代大型机
2、硬件故障视为常态，用软件保证高可靠
3、简化了并行计算模式，无需控制节点同步和数据交换

核心：
1、HDFS：分布式文件系统 存储海量数据
2、MapReduce：并行处理框架，实现任务分解和调度

Hadoop用来做什么？
A: 搭建大型数据仓库，PB级数据的存储、处理、分析、统计等业务。
如搜索引擎、日志分析、商业智能、数据挖掘

Hadoop的优势：
1、高扩展
2、低成本
3、成熟的生态圈

Hadoop的生态：

HDFS+MapReduce+Hive+Hbase+zookeeper

wget http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-1.2.1/hadoop-1.2.1.tar.gz