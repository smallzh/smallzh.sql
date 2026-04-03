# 更新索引数据

## 准备数据

继续基于Elasticsearch-head的`复合查询[+]`标签，先添加如下内容：

```json
{
    "name":"活着",
    "author": "余华",
    "publisher": "作家出版社",
    "year": "2012-08",
    "pageNum": 191,
    "isbn": "9787506365437",
    "introduction": "《活着(新版)》讲述了农村人福贵悲惨的人生遭遇。福贵本是个阔少爷，可他嗜赌如命，终于赌光了家业，一贫如洗。他的父亲被他活活气死，母亲则在穷困中患了重病，福贵前去求药，却在途中被国民党抓去当壮丁。经过几番波折回到家里，才知道母亲早已去世，妻子家珍含辛茹苦地养大两个儿女。此后更加悲惨的命运一次又一次降临到福贵身上，他的妻子、儿女和孙子相继死去，最后只剩福贵和一头老牛相依为命，但老人依旧活着，仿佛比往日更加洒脱与坚强。"
}
```

以上内容均来自[豆瓣读书](https://book.douban.com/)

第二个输入框中输入：

```txt
book_index/_doc/7
```

执行结果如下：

![](/assets/img/tutorial/elastic/3-1.jpg)

## 更新数据操作

第二个输入框中，修改为`book_index/_search`，文本框内容填写如下：

```json
{
  "query": {
    "match": {
      "name": "活着"
    }
  }
}
```
点击提交后，显示如下：

![](/assets/img/tutorial/elastic/3-2.jpg)

将第二个输入框中，修改为`book_index/_update/7`，文本框内容如下：

```json
{
  "doc": {
    "name": "活着1"
  }
}
```
点击提交后，显示如下：

![](/assets/img/tutorial/elastic/3-3.jpg)

再修改为之前的查询，提交后，如下：

![](/assets/img/tutorial/elastic/3-4.jpg)

可以看到，内容已被修改