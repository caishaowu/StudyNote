# Elasticsearch介绍

Elasticsearch官网：https://www.elastic.co/cn/products/elasticsearch

**Elasticsearch是一个分布式的 RESTful 风格的搜索和数据分析引擎，底层基于Lucene。**

![1555415772895](C:\Users\csw\AppData\Roaming\Typora\typora-user-images\1555415772895.png)

如上所述，Elasticsearch具备以下特点：

- 分布式，无需人工搭建集群（solr就需要人为配置，使用Zookeeper作为注册中心）
- Restful风格，一切API都遵循Rest原则，容易上手
- 近实时搜索，数据更新在Elasticsearch中几乎是完全同步的。

## 倒排索引

倒排索引（Inverted index），是一种索引方法，被用来存储在全文搜索下**某个单词在一个文档或者一组文档中的存储位置的映射**。它是文档检索系统中最常用的**数据结构**。

看一个例子：

> - T0 = "`it is what it is`"
> - T1 = "`what is it`"
> - T2 = "`it is a banana`"

我们能够得到下面的反向文件索引

> ```json
>  "a":      {2}
>  "banana": {2}
>  "is":     {0, 1, 2}
>  "it":     {0, 1, 2}
>  "what":   {0, 1}
> ```

数据被拆分成了词与文档id列表的集合，这样只通过查询某个词，我们就可以知道哪些文档中有这些词，不用遍历所有文档，效率很高。上面只是最简单的结构，倒排索引还可以包括很多其他信息，比如单词被命中次数，在每个文档中出现的次数等。

> "a":      {(2, 2)}
> "banana": {(2, 3)}
> "is":     {(0, 1), (0, 4), (1, 1), (2, 1)}
> "it":     {(0, 0), (0, 3), (1, 2), (2, 0)} 
> "what":   {(0, 2), (1, 0)}

`"banana": {(2, 3)}`的意思是在第三个文档（T2）的第4个单词（地址为3）就是`banana`。

## 基本概念

ES中存储数据的**基本单位是索引**。

| 概念                          | 说明                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| index                         | 一种类别的表                                                 |
| type（**Deprecated in 6.0**） | 一个index可能有多个type，每个type可以看成是mysql里的一张表   |
| mapping                       | 相当于表结构定义，字段的数据类型、属性、是否索引、是否存储等特性 |
| document                      | 相当于某个表的一行数据                                       |
| field                         | document中的属性，相当于column                               |

- 索引集（Indices，index的复数）：逻辑上的完整索引
- 分片（shard）：数据拆分后的各个部分
- 副本（replica）：每个分片的复制

要注意的是：Elasticsearch本身就是分布式的，因此即便你只有一个节点，Elasticsearch默认也会对你的数据进行分片和副本操作，当你向集群添加新数据时，数据也会在新加入的节点中进行平衡。

