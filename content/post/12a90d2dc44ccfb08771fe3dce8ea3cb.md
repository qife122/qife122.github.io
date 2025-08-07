---
date: 2025-08-07T19:52:30+08:00
title: 在Ubuntu 16.04上安装Cacti 1.1.10的完整指南
tags: [Cacti, Ubuntu, 网络监控, 系统管理]
authors: qife
description: 本文详细介绍了在Ubuntu 16.04系统上安装和配置Cacti 1.1.10网络监控工具的完整步骤，包括系统准备、软件依赖安装、数据库配置、Apache设置以及最终的系统部署和登录过程。
---

# 在Ubuntu 16.04上安装Cacti 1.1.10

**作者：Kent Ickler**  
**注意：** 本文中提到的技术和工具可能已过时，不适用于当前情况。但本文仍可作为学习资源，并可能被整合或更新到现代工具和技术中。

## 什么是Cacti？

Cacti是一个网络系统，用于输入系统生成的量化数据并以精美的图表呈现这些数据。

### 网络管理员视角
在网络管理领域，Cacti提供时间关键和时间历史数据，帮助您做出重要决策。典型的数据输入包括：交换机端口利用率、环境信息（温度、湿度等）、系统关键指标（存储空间、CPU时间等）。

### 安全管理员视角
结合SIEM和其他系统数据源，Cacti可用于生成安全基线和规范化模式。它也是快速检查网络健康状况的工具。

## 安装Ubuntu 16.04

我们从ISO `ubuntu-16.04.2-server-amd64.iso` 进行安装。完成典型设置，但在操作系统安装包选择时确保安装了LAMP包。

您将被提示创建MySQL root账户密码。创建密码（不要留空），妥善保存（稍后在处理mysql和mysqladmin时会需要），然后继续。

安装完成后登录。注意我们需要进行的所有更新！

### 更新基础系统
```bash
sudo -s
apt-get update
apt-get upgrade
reboot -h now
```

### 配置网络
更新完成后，设置您的网络堆栈。然后重启：
```bash
sudo -s
nano /etc/network/interfaces
```

**关于nano的说明：** CTRL+O保存更改，CTRL+X关闭  
更新您的网络设置并再次重启。
```bash
reboot -h now
```

### 关于sudo和root的说明
从现在开始的大部分工作都是在root下完成的，因为这些工作主要在/opt/目录下进行和安装组件。
```bash
sudo -s
```

## 安装预装依赖

重启后再次登录。我们需要为Cacti安装一些预装依赖：
```bash
apt-get install php-xml php-ldap php-mbstring php-gd php-snmp php-gmp rrdtool snmp librrds-perl
```

### 下载Cacti文件
```bash
wget http://www.cacti.net/downloads/cacti-1.1.10.tar.gz
tar xvzf cacti-1.1.10.tar.gz
mv cacti-1.1.10 /opt/cacti
```

### 设置日志位置
```bash
mkdir /opt/logs
touch /opt/logs/cacti.log
touch /opt/logs/httpd_access.log
touch /opt/logs/httpd_error.log
chown -R www-data /opt/logs/*
```

## 配置SQL数据库

```bash
# 创建cacti数据库
mysqladmin --user=root --password create cacti
### 输入您的mysql root密码

# 填充Cacti数据库
mysql --user root -p cacti < /opt/cacti/cacti.sql
### 输入您的mysql root密码
### 这个过程需要几分钟，耐心等待提示符返回

# 在SQL中创建时区表
mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u root -p mysql
### 输入您的mysql root密码

# 为cacti用户提供对cacti数据库和时区数据库的访问权限
mysql --user=root --password mysql
### 输入您的mysql root密码。
### 这将使您进入mysql数据库的控制台。
### 注意：这里引用的'somepassword'是cacti用户的密码，必须与下一节中cacti配置中使用的密码相同

mysql> GRANT ALL ON cacti.* TO cacti@localhost IDENTIFIED BY 'somepassword';
mysql> GRANT SELECT ON mysql.time_zone_name TO cacti@localhost IDENTIFIED BY 'somepassword';

exit
```

## 配置Cacti文件

注意：这里引用的'somepassword'是上面指定的cacti数据库用户密码。
```bash
nano /opt/cacti/include/config.php
### 找到这些变量并进行以下更改

$database_type = 'mysql';
$database_default = 'cacti';
$database_hostname = 'localhost';
$database_username = 'cactiuser';
$database_password = somepassword;
$database_port = '3306';
$database_ssl = false;
$url_path = '';
```

### 设置文件权限
注意：设置完成后，出于安全原因，"Needed for setup"部分应恢复为您的Linux用户。
```bash
# 安装所需
chown -R www-data:www-data /opt/cacti/resource/snmp_queries
chown -R www-data:www-data /opt/cacti/resource/script_server
chown -R www-data:www-data /opt/cacti/resource/script_queries
chown -R www-data:www-data /opt/cacti/scripts

# 始终需要
chown -R www-data:www-data /opt/cacti/rra/ /opt/cacti/log/
chown -R www-data:www-data /opt/cacti/cache/mibcache
chown -R www-data:www-data /opt/cacti/cache/realtime
chown -R www-data:www-data /opt/cacti/cache/spikekill
```

## 配置Apache

```bash
touch /etc/apache2/sites-available/cacti.conf
nano /etc/apache2/sites-available/cacti.conf
```

### 输入以下内容并保存cacti.conf
```apache
<VirtualHost *:80>
    <Location />
        require all granted
    </Location>

    ServerAdmin webmaster@localhost
    DocumentRoot /opt/cacti
    ErrorLog /opt/logs/httpd_error.log
    CustomLog /opt/logs/httpd_access.log combined
</VirtualHost>
```

### 从Apache中移除默认/现有站点
```bash
rm /etc/apache2/sites-enabled/*
```

### 在Apache中启用Cacti站点
```bash
a2ensite cacti.conf
```

## 配置MySQL

```bash
nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

### 在配置文件底部添加以下行：
```ini
Max_heap_table_size = 380M
Tmp_table_size = 64M
Join_buffer_size = 64M
Innodb_doublewrite = OFF
Innodb_buffer_pool_size = 1899M
Innodb_flush_log_at_timeout = 3
Innodb_read_io_threads = 32
Innodb_write_io_threads = 16
```

## 配置Poller Crontab

```bash
nano /etc/crontab
```

### 在底部添加行
```cron
*/5 * * * * www-data php /opt/cacti/poller.php > /dev/null 2>&1
```

## 重启服务

```bash
service apache2 restart
service mysql restart
```

## 启动Web-GUI安装

完成所有预装依赖后，Web-GUI安装应该会很简单。每页的左下角有NEXT按钮。
```bash
http://[your-cacti-ip]
```

注意：确保将cacti日志路径更新为/opt/logs/cacti.conf

确保检查所有可用的安装模板。

## 完成！登录！

首次登录的默认凭据是：  
**用户：** admin  
**密码：** admin  
首次登录时，系统会提示您更改密码。