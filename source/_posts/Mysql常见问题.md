title: Mysql常见问题
author: Jie
date: 2023-04-19 05:19:47
tags:
---
### 一些我使用Mysql时常遇到的小问题
启动服务
```shell
service mysql start >/dev/null 2>&1
```

#### 连不上虚拟机的mysql
```shell
mysql -u root -p

select Host,User,plugin from mysql.user;
update user set host = '%' where host = 'localhost';

flush privileges;
```
<!-- more -->

#### MySQL外网连接不上
```shell
vim  /etc/mysql/mysql.conf.d/mysqld.cnf
```

![](/images/pasted-27.png)
```sh
/etc/init.d/mysql restart
```
```mysql
create user '666'@'%' identified by '000000';
```
```mysql
grant all privileges on *.* to '666'@'%' identified by '000000' with grant option;
```
```mysql
flush privileges;
```

#### 1251 - Client does not support authentication protocol requested by server
```mysql
alter user 'root'@'localhost' identified with mysql_native_password by '000000';
```
```mysql
flush privileges;
```

### 一些碎碎念
最近在忙着考研，摆烂了很久，没有更新。
感觉像Hexo这样的静态博客还是不太适合我，本来就很少有写技术博客的习惯，加上Hexo是本地服务用的就更少了，还是微博跟即刻更新日常比较多，我在考虑要不要整个服务器玩玩了，部署一些服务，顺便整个博客来发发颠。下一篇写tailscale的教程，有一说一Tailscale够用但是已经不能满足我的需求了，主要是用的家里的台式，不太敢乱来，哎，不知道下一篇又会被我推到什么时候了。