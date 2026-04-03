# 创建索引并添加内容

## 通过elasticsearch-head创建索引

点击elasticsearch-head中的第二个Tab标签，点击创建索引按钮

![](/assets/img/tutorial/elastic/2-1.jpg)

创建名为`book_index` 的索引，后续，都用这个索引进行说明

## 准备添加索引内容

点击`复合查询[+]`Tab标签，在查询输入框中输入`http://127.0.0.1:9200/book_index/`，并点击`提交请求`按钮，如下

![](/assets/img/tutorial/elastic/2-2.jpg)

### 准备以下5条内容，用于测试

数据：

```json
{
    "name": "Elasticsearch实战",
    "author": "Radu Gheorghe",
    "publisher": "人民邮电出版社",
    "translator":"黄申",
    "year":"2018-10",
    "pageNum": 337,
    "isbn":"9787115449153",
    "introduction":"本书主要展示如何使用Elasticsearch构建可扩展的搜索应用程序。书中覆盖了Elasticsearch的主要特性，从使用不同的分析器和查询类型进行相关性调优，到使用聚集功能进行实时性分析，还有地理空间搜索和文档过滤等更多吸引人的特性。"
}
{
    "name":"实战Java高并发程序设计",
    "author":"葛一鸣/郭超",
    "publisher":"电子工业出版社",
    "year":"2015-10",
    "pageNum": 339,
    "isbn":"9787121273049",
    "introduction":"《实战Java高并发程序设计》主要介绍基于Java的并行程序设计基础、思路、方法和实战。第一，立足于并发程序基础，详细介绍Java中进行并行程序设计的基本方法。第二，进一步详细介绍JDK中对并行程序的强大支持，帮助读者快速、稳健地进行并行程序开发。第三，详细讨论有关“锁”的优化和提高并行程序性能级别的方法和思路。第四，介绍并行的基本设计模式及Java 8对并行程序的支持和改进。第五，介绍高并发框架Akka的使用方法。最后，详细介绍并行程序的调试方法。"
}
{
    "name": "经济学通识",
    "author":"薛兆丰",
    "publisher": "北京大学出版社",
    "year":"2015-08",
    "pageNum": 626,
    "isbn": "9787301258699",
    "introduction":"《经济学通识》就是薛兆丰教授的一部自选集，他从自己10多年写作的文章中，精选出98篇。就像他对我说的：“想成为真正的市场经济支持者，或真正的自由主义支持者，你绕不开这本书所讨论的每一个议题。"
}
{
    "name": "Python极简讲义：一本书入门数据分析与机器学习",
    "author": "张玉宏",
    "publisher": "电子工业出版社",
    "year": "2020-05",
    "pageNum": 588,
    "isbn": "9787121387043",
    "introduction": "第1章至第5章以极简方式讲解了Python的常用语法和使用技巧，包括数据类型与程序控制结构、自建Python模块与第三方模块、Python函数和面向对象程序设计等。第6章至第8章介绍了数据分析必备技能，如NumPy、Pandas和Matplotlib。第9章和第10章主要介绍了机器学习的基本概念和机器学习框架sklearn的基本用法。"
}
{
    "name": "斗破苍穹",
    "author": "天蚕土豆",
    "publisher": "湖北少年儿童出版社",
    "year": "2010-07",
    "pageNum": 6840,
    "isbn": "9787535381668",
    "introduction": "《斗破苍穹(珍藏全集)(套装共27册)》讲述了天才少年萧炎在创造了家族空前绝后的修炼纪录后突然成了废人，整整三年时间，家族冷遇，旁人轻视，被未婚妻退婚……种种打击接踵而至。就在他即将绝望的时候，一缕幽魂从他手上的戒指里浮现，一扇全新的大门在面前开启！"
}
```

以上内容均来自[豆瓣读书](https://book.douban.com/)

将数据添加到Elastic中，在查询的第二个文本框中，填入以下内容：

```txt
book_index/_doc/1
```
并填入以上Json中的第一个数据，如下

![](/assets/img/tutorial/elastic/2-3.jpg)

依次全部添加以上Json数据，执行查询全部，第二个文本框中输入以下内容：

```txt
book_index/_search
```
在文本框中，填写如下：

```txt
{
  "query": {
    "match_all": {}
  }
}
```
最终查询结果为：

![](/assets/img/tutorial/elastic/2-4.jpg)