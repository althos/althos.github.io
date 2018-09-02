---
layout:     post
title:      mysql的安装
subtitle:   cntos7 MySQL5.7安装版配置
date:       2018-09-01
author:     Cyber
header-img: img/post-bg-map.jpg
catalog: true
tags:
    - 后端
    - linux
    - mysql
    - 服务器
---





> 一个简单的centos7 mysql版本安装......



# 获取源

> local]# wget http://dev.mysql.com/get/mysql57-community-release-el7-8.noarch.rpm
>
> local]#yum localinstall mysql57-community-release-el7-8.noarch.rpm
>
> local]#yum repolist enabled | grep "mysql.*-community.*"



#  修改安装的版本

>
>
> 可以修改vim /etc/yum.repos.d/mysql-community.repo源，改变默认安装的mysql版本。比如要安装5.6版本，将5.7源的enabled=1改成enabled=0。然后再将5.6源的enabled=0改成enabled=1即可。改完之后的效果如下所示：
>
> name=MySQL Connectors Community
> baseurl=http://repo.mysql.com/yum/mysql-connectors-community/el/7/$basearch/
> enabled=1
> gpgcheck=1
> gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
>
> [mysql-tools-community]
> name=MySQL Tools Community
> baseurl=http://repo.mysql.com/yum/mysql-tools-community/el/7/$basearch/
> enabled=1
> gpgcheck=1
> gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
>
> Enable to use MySQL 5.5
>
> [mysql55-community]
> name=MySQL 5.5 Community Server
> baseurl=http://repo.mysql.com/yum/mysql-5.5-community/el/7/$basearch/
> enabled=0
> gpgcheck=1
> gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
>
> Enable to use MySQL 5.6
>
> [mysql56-community]
> name=MySQL 5.6 Community Server
> baseurl=http://repo.mysql.com/yum/mysql-5.6-community/el/7/$basearch/
> enabled=0
> gpgcheck=1
> gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
>
> [mysql57-community]
> name=MySQL 5.7 Community Server
> baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/7/$basearch/
> enabled=1
> gpgcheck=1
> gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
>
> [mysql-tools-preview]
> name=MySQL Tools Preview
> baseurl=http://repo.mysql.com/yum/mysql-tools-preview/el/7/$basearch/
> enabled=0
> gpgcheck=1
> gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
>
>

# 安装

>  local]# yum install mysql-community-server





# 启动Mysql

>  local]# systemctl start mysqld

### 查看状态

> local]# systemctl status mysqld
>
> mysqld.service - MySQL Server
>   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
>   Active: active (running) since Sun 2018-09-02 12:27:10 CST; 1min 21s ago
>     Docs: man:mysqld(8)
>           http://dev.mysql.com/doc/refman/en/using-systemd.html
>  Process: 20011 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
>  Process: 19938 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
> Main PID: 20016 (mysqld)
>   CGroup: /system.slice/mysqld.service
>           └─20016 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid
>
>Sep 02 12:27:05 izwz9fedjd8qp8b9pgico7z systemd[1]: Starting MySQL Server...
>Sep 02 12:27:10 izwz9fedjd8qp8b9pgico7z systemd[1]: Started MySQL Server.
>
>



# 设置开机启动

>  local]#  systemctl enable mysqld
>  local]#  systemctl daemon-reload



# 修改root本地登录密码

mysql安装完成之后，在/var/log/mysqld.log文件中给root生成了一个默认密码。通过下面的方式找到root默认密码，然后登录mysql进行修改：

>local]#  grep 'temporary password' /var/log/mysqld.log
>
>2018-09-02T04:27:06.385413Z 1 [Note] A temporary password is generated for root@localhost: ,`#1xe*HUS5&w`
>
>

找到密码之后登陆修改

> local]# mysql -uroot -p
>
> Server version: 5.7.23
>
> Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.
>
> Oracle is a registered trademark of Oracle Corporation and/or its
> affiliates. Other names may be trademarks of their respective
> owners.
>
> Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
>
> mysql> 

修改密码:

> ALTER USER 'root'@'%' IDENTIFIED BY '123';
>
> flush privileges;



# 修改端口

> vi   /etc/my.cnf
>
>```
>[mysqld]
>#
># Remove leading # and set to the amount of RAM for the most important data
># cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
># innodb_buffer_pool_size = 128M
>#
># Remove leading # to turn on a very important data integrity option: logging
># changes to the binary log between backups.
># log_bin
>#
># Remove leading # to set options mainly useful for reporting servers.
># The server defaults are faster for transactions and fast SELECTs.
># Adjust sizes as needed, experiment to find the optimal values.
># join_buffer_size = 128M
># sort_buffer_size = 2M
># read_rnd_buffer_size = 2M
>port=33899                 #添加port
>character_set_server=utf8  #添加编码
>datadir=/var/lib/mysql
>socket=/var/lib/mysql/mysql.sock
>
># Disabling symbolic-links is recommended to prevent assorted security risks
>symbolic-links=0
>
>log-error=/var/log/mysqld.log
>pid-file=/var/run/mysqld/mysqld.pid
>~                                       
>```
>
>重启mysql
>
>```
>service mysqld restart
>```
>
>查看
>
>```
>netstat -nap|grep 33899
>tcp6       0      0 :::33899                :::*                    LISTEN      20325/mysqld 
>```
>
>