---
layout: post
title: 'Docker部署Mysql并实现数据持久化'
date: 2019-11-07
author: 白皓
cover: ''
tags: Docker
---
###  一、下载

前往[Docker Hub](https://hub.docker.com/_/mysql) 搜索mysql镜像,此处使用mysql 5.7。

>   <span style="color:red;font-size:15px">注:</span>不推荐使用5.7以下的版本，因为5.7开始支持noSQL，效率比之前的版本更快。


拉取mysql5.7的镜像文件
```docker
$   docker pull mysql:5.7
```

### 二、配置文件

在/home/文件夹下创建docker持久化数据目录用来保存持久化数据，方便统一管理，防止数据随着容器的删除而丢失。
```docker
/home/docker/mysql/conf     数据库配置文件
/home/docker/mysql/data     数据库文件
/home/docker/mysql/logs     数据库日志
```
将mysql的配置文件复制到/home/docker/mysql/conf目录中
>  <span style="color:red;font-size:15px">注:</span> mysql配置文件目录为<span style="color:red;background-color:GhostWhite">/etc/mysql</span> 中,若无配置文件可去掉<span style="color:red;background-color:GhostWhite">-v /home/docker/mysql/conf:/etc/mysql </span> 后执行下面的启动命令新建一个容器，然后进入<span style="color:red;background-color:GhostWhite">/home/docker/mysql/conf</span> 目录中执行<span style="color:red;background-color:GhostWhite"> docker cp -rf mysql5.7:/etc/mysql/* .</span>容器中的配置值文件复制到宿主机中的<span style="color:red;background-color:GhostWhite">/home/docker/mysql/conf</span> 目录中，然后将此容器删除，重新运行下面启动命令创建一个新容器即可。

### 三、启动

启动mysql容器，设置默认密码<span style="color:red;background-color:GhostWhite">123456</span> 
```docker
$   docker run -it -p 3306:3306 \               
    -v /home/docker/mysql/conf:/etc/mysql \
    -v /home/docker/mysql/logs:/var/log/mysql \
    -v /home/docker/mysql/data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=123456 \
    --name mysql5.7 \
    docker.io/mysql:5.7
```

命令说明：
>   -v /home/docker/mysql/conf:/etc/mysql      
>   将宿主机上的mysql配置文件目录挂载到容器中

>   -v /home/docker/mysql/logs:/var/log/mysql 
>      挂载宿主机的日子文件目录到容器中

>    -v /home/docker/mysql/data:/var/lib/mysql 
         挂载宿主机上的数据库文件目录到容器中

>    -e MYSQL_ROOT_PASSWORD=123456 
            设置数据库初始密码

>    --name mysql5.7 
          给容器命名

### 四、 常用命令
进入容器
```docker
$ docker exec -it mysql5.7 bash
```

查看日志
```docker
$ docker logs -f mysql5.7
```

备份数据
```docker
$ docker exec mysql5.7 sh -c 'exec mysqldump --all -databases -u root -p "123456"' > /some/path/on/your/host/all-databases.sql
```

恢复数据
```docker
$ docker exec -i mysql sh -c 'exec mysql -u root -p "123456"' < /some/path/on/your/host/all-databases.sql
```

### 五、 其他问题

<span style="color:red;background-color:GhostWhite">only_full_group_by</span>  问题
如果安装的版本5.7版本在查询数据时出现如下错误

>   this is incompatible with sql_mode=only_full_group_by

可以使用使用下列方式解决

1. 查询 <span style="color:red;background-color:GhostWhite">sql_mode</span>
```sql
select @@sql_mode
```
结果如下
```sql
ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

2. 重置
删除其中的 <span style="color:red;background-color:GhostWhite">ONLY_FULL_GROUP_BY</span>配置，添加到到 <span style="color:red;background-color:GhostWhite">mysqld.cnf</span>中
```docker
[mysqld]
# 表名不区分大小写
lower_case_table_names=1 
#server-id=1
datadir=/var/lib/mysql
#socket=/var/lib/mysql/mysqlx.sock
#symbolic-links=0
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

3. 重启容器
```docker
$ docker restart mysql
```
