MySql://www.enterprisedb.com/download-postgresql-binaries 下载对应的zip包

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
