# 内容查询

继续使用elasticsearch-head的`复合查询[+]`Tab标签

## 查询全部

查询的第二个输入框中输入`book_index/_search`，下面文本框中输入：

```json
{
  "query": {
    "match_all": {}
  }
}
```
或
```json
{}
```
点击提交，看到如下

![](/assets/img/tutorial/elastic/5-1.jpg)

## 查询某个字段

第二个输入框中依然是`book_index/_search`，下面文本框中输入
```json
{
  "query": {
    "match": {
      "name": "实战"
    }
  }
}
```
点击提交后，显示如下

![](/assets/img/tutorial/elastic/5-2.jpg)

## 查询多个字段，并进行过滤

第二个输入框保持不变，文本框中输入

```json
{
  "query": {
    "bool": {
      "must": [
        { "match": { "name": "实战" } },
        { "match": { "author": "郭" } }
      ],
      "filter": [
        { "range": { "pageNum": { "gte": 200 } } }
      ]
    }
  }
}
```
添加提交，得到如下结果

![](/assets/img/tutorial/elastic/5-3.jpg)