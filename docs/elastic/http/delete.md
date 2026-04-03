# 删除索引数据

## 准备数据

继续基于Elasticsearch-head的`复合查询[+]`标签，先添加如下内容：

```json
{
    "name":"漫长的余生",
    "author": "罗新",
    "publisher": "北京日报出版社",
    "year": "2022-07",
    "pageNum": 328,
    "isbn": "9787547743126",
    "introduction":"公元466年，宋明帝刘彧与在寻阳称帝的侄子刘子勋二帝并立，内战几乎波及刘宋全境，继而演变为与北魏之间的战争。生于南朝中层官僚家庭的王钟儿，被迫卷入，家破人亡，两年后被掠为平城宫的普通宫女，时年三十岁。可是，她的命运却偶然地与“子贵母死”制度发生了联系，意外卷入权力斗争的漩涡，先后以宫女和比丘尼的身份成为抚育两代皇帝的关键人物，竟在北魏宫廷生活了五十六年之久。"
}
```

以上内容均来自[豆瓣读书](https://book.douban.com/)

第二个输入框中输入：

```txt
book_index/_doc/8
```

执行结果如下：

![](/assets/img/tutorial/elastic/4-1.jpg)

## 删除数据操作

在第二个输入框中输入`book_index/_search`，文本框中输入

```json
{
  "query": {
    "match": {
      "name": "余生"
    }
  }
}
```
点击提交后，看到如下：

![](/assets/img/tutorial/elastic/4-2.jpg)

接下来，第二个输入框中输入内容：
```txt
book_index/_delete_by_query
```
文本框中，还是以上的
```json
{
  "query": {
    "match": {
      "name": "余生"
    }
  }
}
```
点击提交，看到如下结果：

![](/assets/img/tutorial/elastic/4-3.jpg)

将第二个输入框中的内容改为`book_index/_search`，添加提交，结果如下

![](/assets/img/tutorial/elastic/4-4.jpg)

可以看到，以上内容被删除