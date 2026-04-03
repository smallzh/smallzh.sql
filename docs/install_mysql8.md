# MySql8相关

## Docker镜像方式安装

这里使用MySql版本为 `8.0.4` 的 Docker 镜像

拉取镜像

```shell
docker pull mysql:8.0.4
```

创建配置文件和存放数据目录

```shell
mkdir -p /home/mysql/conf
mkdir -p /home/mysql/data
mkdir -p /home/mysql/mysql-files
```

启动容器

```shell
docker run --name mysql8 --restart=always -v /home/mysql/conf/my.cnf:/etc/mysql/my.cnf -v /home/mysql/data:/var/lib/mysql -v /home/mysql/mysql-files:/var/lib/mysql-files -v /etc/localtime:/etc/localtime:ro -p 3306:3306 -e MYSQL_ROOT_PASSWORD=12345687@Smallzh -d mysql:8.0.4
```

其中，

`-v /etc/localtime:/etc/localtime:ro` 表示，本机时间与数据库时间同步

授权root用户远程访问

```shell
docker exec -it mysql8 /bin/bash
mysql -uroot -p
# 授权root远程访问
use mysql;
select Host, User, authentication_string, plugin from user;
# 如果没有 rooter@%用户的话，创建用户
create user 'rooter'@'%' identified with mysql_native_password by '12345687@Sz';
grant all privileges on *.* to 'rooter'@'%' with grant option;
flush privileges;
```

## Window Zip方式安装
从 官网：https://downloads.mysql.com/archives/community/ 下载 zip安装包

解压到安装目录，比如：`E:\mysql8`

在安装目录下创建 my.ini配置文件，内容如下：
注意：修改mysql的安装目录和数据目录

```txt
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8
 
[mysqld]
# 绑定端口
bind-address = 0.0.0.0
# 设置3306端口
port = 3306
# 设置mysql的安装目录
basedir=E:\mysql-8.0.41
# 设置mysql数据库的数据的存放目录
datadir=D:\mysql8\data
# 允许最大连接数
max_connections=10000
# 允许最大连接人数
max_user_connections=1000
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8mb4
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
 
# 连接时间
wait_timeout=2147483
interactive_timeout=31536000
```

进入安装目录执行以为命令

```shell
# 进入目录
cd e:\mysql8\bin

# 初始化mysql数据库
mysqld --initialize-insecure --user=mysql --console

# 注册mysql服务
mysqld --install

# 启动mysql 服务
net start mysql
```

登录mysql数据库，设置root密码。从控制台的输出能看出，数据库初始化时，root密码为空

```shell
# 在mysql安装目录的bin目录下执行
mysql -u root -p

# 修改root的用户密码
alter user 'root'@'localhost' identified by 'root';
flush privileges;
```
