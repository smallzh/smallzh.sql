# Elastic 相关使用教程

## 安装并启动Elastic

Elastic版本为 `8.4.3`

### 本地安装

下载地址

[https://www.elastic.co/cn/downloads/elasticsearch](https://www.elastic.co/cn/downloads/elasticsearch)

文档参考地址

[https://www.elastic.co/guide/en/elasticsearch/reference/8.4/query-dsl.html](https://www.elastic.co/guide/en/elasticsearch/reference/8.4/query-dsl.html)

这里只做学习使用，我下载的window版本

下载完成后，先执行 `bin/elasticsearch.bat` 启动elastic，启动过程中，会输出一些内容，这些内容，我这里就忽略了，我这里目前只关注**elastic的使用**

启动完成后，先关闭应用，关掉启动窗口即可

然后修改`config/elasticsearch.yml`配置文件，如下：

```yml
# Enable security features
xpack.security.enabled: false

...

# Enable encryption for HTTP API client connections, such as Kibana, Logstash, and Agents
xpack.security.http.ssl:
  enabled: false
  keystore.path: certs/http.p12

...
http.cors.enabled: true
http.cors.allow-origin: "*"
```

改动地方：
1. 上面两处的 enabled，改成 false，使用http访问，因为我们这里 只做学习使用
2. 添加cors跨域支持，给 `elasticsearch-head`使用

### Docker方式

拉取docker镜像

```shell
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.4.3
```

创建 `elasticsearch`用户及其用户组

```shell
addgroup elasticsearch
adduser elasticsearch --no-create-home --ingroup elasticsearch --disabled-password --disabled-login
```

创建必要目录，并修改目录的用户和用户组

```shell
mkdir -p /home/elastic/data
mkdir -p /home/elastic/config
mkdir -p /home/elastic/plugins
```

将本地下载的config目录上传到`config`目录中，然后修改目录拥有者

```shell
chown -R elasticsearch /home/elastic
chgrp -R elasticsearch /home/elastic
```

启动容器

```shell
docker run -d --name elastic8 --restart=always -v /home/elastic/plugins:/usr/share/elasticsearch/plugins -v /home/elastic/data:/usr/share/elasticsearch/data -v /home/elastic/config:/usr/share/elasticsearch/config -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" -e TAKE_FILE_OWNERSHIP=111 docker.elastic.co/elasticsearch/elasticsearch:8.4.3
```

### 访问

配置修改完成后，重新启动应用，浏览器访问 `http://127.0.0.1:9200/`，出现以下页面

![](/assets/img/tutorial/elastic/1-1.jpg)

## 安装 elasticsearch-head

先安装`NodeJs`环境

然后，GitHub上下载项目

```shell
git clone git://github.com/mobz/elasticsearch-head.git
```

安装依赖并启动项目, 进入到clone下来的elasticsearch-head目录，执行以下命令

```shell
npm install
npm run start
````

然后浏览器访问`http://127.0.0.1:9100/`，连接输入框中填写`http://127.0.0.1:9200/`，如下

![](/assets/img/tutorial/elastic/1-2.jpg)

点击其中的`复合查询[+]` Tab标签，看到如下，后续的Elastic使用，都基于这个功能界面说明

![](/assets/img/tutorial/elastic/1-3.jpg)