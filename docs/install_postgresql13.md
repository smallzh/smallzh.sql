从 https://www.enterprisedb.com/download-postgresql-binaries 下载对应的zip包

解压到安装目录，比如：`E:\pgsql`

进入到bin目录中，执行以下命令

```shell
# 初始化数据库实例
initdb -D "D:\postgresql\data" -E UTF8 -U postgres --locale="Chinese (Simplified)_China.936" --lc-messages="Chinese_China.936" -A scram-sha-256 -W
```


其中各参数的意义：

-D: 数据库数据存放位置
-E: UTF8 编码 
-U: 账号名
--local: 排序规则，这里用Chinese (Simplified)_China.936 
--A: 账户加密方式，使用 scram-sha-256

注册成Window服务
```shell
pg_ctl register -N "PostgreSQL13" -D "D:\postgresql\data" -l "D:\postgresql\log\logfile.log" -o "-p 5432"
```

## Docker 安装

### 拉取镜像

```shell
docker pull postgres:13
```

### 运行容器

```shell
# 基本运行
docker run -d \
  --name postgresql13 \
  -e POSTGRES_PASSWORD=your_password \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_DB=postgres \
  -p 5432:5432 \
  postgres:13
```

### 数据持久化

```shell
# 带数据卷持久化
docker run -d \
  --name postgresql13 \
  -e POSTGRES_PASSWORD=your_password \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_DB=postgres \
  -p 5432:5432 \
  -v postgresql13_data:/var/lib/postgresql/data \
  postgres:13
```

### 常用配置

```shell
# 自定义配置、数据持久化、开机自启
docker run -d \
  --name postgresql13 \
  --restart=always \
  -e POSTGRES_PASSWORD=your_password \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_DB=postgres \
  -p 5432:5432 \
  -v postgresql13_data:/var/lib/postgresql/data \
  -v /path/to/postgresql.conf:/etc/postgresql/postgresql.conf \
  postgres:13 -c config_file=/etc/postgresql/postgresql.conf
```

### 连接命令

```shell
# 进入容器
docker exec -it postgresql13 psql -U postgres

# 外部连接
psql -h localhost -p 5432 -U postgres
```
