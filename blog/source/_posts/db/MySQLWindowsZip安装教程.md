---
title: MySQLWindowsZip安装教程
tags:
  - MySQL
categories:
  - MySQL
date: 2023-01-11 21:19:43
---

#### Windows下MySQL压缩包版安装教程

##### 附下载地址

##### [mysql](https://dev.mysql.com/downloads/mysql/) 

![mysql_download](..\images\mysql_download.jpg)

##### 创建my.ini配置文件

###### 设置端口、最大连接数、字符编码集、存储引擎、数据存放目录

```html
[mysql]
# 设置mysql客户端默认字符集 #8.0.2 是utf8mb4
default-character-set=utf8mb4
[mysqld]
#设置3306端口
port=3306
# 设置mysql的安装目录
basedir=D:\mysql-8.0.29-winx64
# 设置mysql数据库的数据的存放目录
datadir=D:\mysql-8.0.29-winx64\data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为UTF8 #8.0.2 是utf8mb4
character-set-server=utf8mb4
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
```

##### 执行命令

###### 初始化

```shell
mysqld --initialize-insecure
```

`说明：-insecure表示初始化过程中不设置密码，不设置这个命令后面第一次登陆的时候找自动生成的密码会很麻烦，不体验过不知道啊`:satisfied:

###### 安装并启动服务

```shell
mysqld install mysqlservice 服务名自己定义
```

```shell
net start mysqlservice
```

`卸载windows服务命令` 

```shell
sc delete 服务名称
```

###### 设置密码

```shell
mysql -u root -p 直接回车即可，没有密码
```

```shell
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
```

###### 刷新

```shell
flush privileges;
```

###### 重新登陆搞定。

##### 创建用户并赋予权限

```shell
create user test@'%' identified by 'test';
```

`'%'表示登陆时不限制IP`

```shell
grant select,update,delete,insert on test.* to test@'%';
```



:dog:



