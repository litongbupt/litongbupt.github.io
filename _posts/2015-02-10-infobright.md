---
layout: post
title: "infobright安装"
description: "infobright的安装"
category: sql_and_nosql
tags: [infobright,安装]
---
{% include JB/setup %}
我们可以直接下载rpm或者deb版本进行安装。 
1.Download the install package (e.g. infobright-3.4-x86_64.rpm) to the server where you are installing Infobright 

2.Obtain root user access 

3.To install the RPM package, run: 
rpm -ivh infobright_version_name.rpm [optional: --prefix=path] 
To install the DEB package, run: 
dpkg -i infobright_version_name.deb 

Note: Please do not install ICE in the root or home directories due to possible MySQL permission checking issues during install, start up, and/or load. [需要注意会有mysql的权限问题，所以安装的目录需要chown mysql:mysql授予访问权限] 

4.To change the default install options, after installation run: 
/usr/local/infobright/postconfig.sh 

You can run this script at any time after installation to change the datadir, CacheFolder, socket, and port. The script must be run as root and ICE must not be running. 【需要在infobright停止运行的时候再修改目录相关，该脚本需要在安装目录下运行，所以需要承 cd /usr/local/infobright/】 

5.The installation determines the optimum memory settings based on the physical memory of the system. You may change these settings by editing the file brighthouse.ini within the data directory. 

Important: The memory settings assume that there are no other services on the machine consuming significant memory. If this is not the case, please lower the memory settings for Infobright. 

6.To start or stop ICE, run: 
/etc/init.d/mysqld-ib start 
/etc/init.d/mysqld-ib stop 

7.To connect to ICE, use the script mysql-ib: 
/usr/bin/mysql-ib [optional:db_name] 

8.To uninstall ICE, run either: 
rpm -e infobright 
dpkg -r infobright 
 
备注：    
数据导入导出工具 http://www.infobright.org/Downloads/contribute/
load工具：https://github.com/litongbupt/infobright_load_tool

9.示例：从普通的MySQL数据库（假设MySQL安装路径为/usr/local/webserver/mysql）导出数据到csv文件： 
    
    /usr/local/webserver/mysql/bin/mysql  -S /tmp/mysql3306.sock -D tongji_logs -e "select * from  log_visits_2010_05_10 into outfile '/data0/test.csv' FIELDS TERMINATED  BY ',' ENCLOSED BY '"'  ESCAPED BY '\' LINES TERMINATED BY 'n';" 

10.示例：普通MySQL和Infobright建表对比     
①、普通MySQL的InnoDB存储引擎建表： 

    CREATE TABLE IF NOT EXISTS `log_visits_2010_05_12` ( 
    `id` int(11) NOT NULL AUTO_INCREMENT, 
    `cate_id` int(11) NOT NULL, 
    `site_id` int(11) unsigned NOT NULL, 
    `visitor_localtime` char(8) NOT NULL, 
    `visitor_idcookie` varchar(255) NOT NULL, 
    PRIMARY KEY (`id`), 
    KEY `cate_site_id` (`cate_id`,`site_id`), 
    KEY `visitor_localtime` (`visitor_localtime`) 
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8; 
②、Infobright的BRIGHTHOUSE存储引擎建表： 

    CREATE TABLE IF NOT EXISTS `log_visits` ( 
    `id` int(11) NOT NULL, 
    `cate_id` int(11) NOT NULL, 
    `site_id` int(11) NOT NULL, 
    `visitor_localtime` char(8) NOT NULL, 
    `visitor_idcookie` varchar(255) NOT NULL, 
    ) ENGINE=BRIGHTHOUSE DEFAULT CHARSET=utf8; 

注：BRIGHTHOUSE存储引擎建表时不能有AUTO_INCREMENT自增、unsigned无符号、unique唯一、主键PRIMARY KEY、索引KEY。 

11.示例：从csv文件导入数据到Infobright数据仓库： 

    /usr/local/infobright/bin/mysql  -S /tmp/mysql3307.sock -D dw --skip-column-names -e "LOAD DATA INFILE  '/data0/test.csv' INTO TABLE log_visits_2010_04_13 FIELDS TERMINATED BY  ',' ESCAPED BY '\' LINES TERMINATED BY 'n';" 

12.更改目录后不能启动mysqld的种种解答 

- 配置文件在/etc/my-ib.cnf ;
- 启动脚本/etc/init.d/mysqld-ib可以知道配置文件中设置了mysql的sock，必须得让mysql对mysql.sock存放路径有访问权限
 