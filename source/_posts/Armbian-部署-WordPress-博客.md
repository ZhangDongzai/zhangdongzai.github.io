---
title: Armbian 部署 WordPress 博客
date: 2026-07-20 16:47:20
tags: [Armbian, WordPress, Linux, PHP, Nginx, MySQL]
---

## 准备

对比系统环境来酌情参考

```
    _             _    _              ___  ___ 
   /_\  _ _ _ __ | |__(_)__ _ _ _    / _ \/ __|
  / _ \| '_| '  \| '_ \ / _` | ' \  | (_) \__ \
 /_/ \_\_| |_|_|_|_.__/_\__,_|_||_|  \___/|___/
                                               
 v26.08.0 for Aml.S905l3a running Armbian Linux 6.18.38-ophub

 Packages:     Ubuntu stable (resolute)
```

文章数据库使用MySQL（还可以使用Plesk、cPanel或phpMyAdmin，这里只演示MySQL，其他参考[https://developer.wordpress.org/advanced-administration/before-install/creating-database/](https://developer.wordpress.org/advanced-administration/before-install/creating-database/)），全部指令在Root用户下执行

```shell
sudo -i
```

## 安装 & 配置依赖

安装Nginx、MySQL和PHP，MySQL可能配置root密码

```shell
apt update && apt upgrade -y
apt install nginx mysql-server php php-fpm php-mysql php-mbstring php-xmlrpc php-soap php-gd php-xml php-cli -y
```

### 配置PHP

貌似新版本的`php-fpm`服务名称会跟着PHP版本数字而变，需要查看具体名称

```shell
systemctl list-units | grep php
```

输出结果有一行是这样的(php后数字不一定是一样的)

```
php8.5-fpm.service
```

根据输出结果执行命令

```shell
systemctl start php8.5-fpm
systemctl enable php8.5-fpm
```

### 配置Nginx

#### 第1步：配置开机启动并立即启动

```shell
systemctl start nginx
systemctl enable nginx
```

直接访问armbian的IP地址，如果显示以下内容则Nginx启动成功

![](http://192.168.1.2/blog/wp-content/uploads/2026/07/Nginx-Success.png)

#### 第2步：修改Nginx配置文件，使Nginx与PHP互通

```shell
nano /etc/nginx/sites-available/default
```

在index中添加`index.php`文件，整行如下

```
index index.php index.html index.htm index.nginx-debian.html;
```

取消`location ~ \.php$` 的注释，使用unix通道，整段如下

```
	# pass PHP scripts to FastCGI server
	location ~ \.php$ {
		include snippets/fastcgi-php.conf;
	
		# With php-fpm (or other unix sockets):
		fastcgi_pass unix:/run/php/php-fpm.sock;
	#	# With php-cgi (or other tcp sockets):
	#	fastcgi_pass 127.0.0.1:9000;
	}
```

保存退出即可

### 配置MySQL

笔者这里安装MySQL时没有提示设置root密码，需要手动设置，如果读者设置过了可以跳过第1步

#### 第一步：设置root密码

进入MySQL命令行

```shell
mysql
```

输入以下指令，替换自己希望设置的密码（可以不用和linux的root密码一样），主机名称(hostname)一般是`localhost`

```MySQL
ALTER USER 'root'@'主机名称' IDENTIFIED WITH caching_sha2_password BY 'root密码';
```

刷新并退出

```MySQL
FLUSH PRIVILEGES;  
EXIT
```

#### 第二步：为WordPress创建数据库和用户

重新进入MySQL命令行，需要输入root密码

```shell
mysql -u root -p
```

以下步骤中设置的名称、密码等需要记住，WordPress设置时需要

创建数据库，自行更改数据库名称

```MySQL
CREATE DATABASE 数据库名称;
```

创建用户，自行更改用户名称和密码

```MySQL
CREATE USER "用户名称"@"主机名称" IDENTIFIED BY "用户密码";
```

给该用户访问WordPress数据库的权限

```MySQL
GRANT ALL PRIVILEGES ON 数据库名称.* TO "用户名称"@"主机名称";
```

刷新并退出

```MySQL
FLUSH PRIVILEGES;  
EXIT
```
#### 第3步：配置开机启动并立即启动

```shell
systemctl start mysql
systemctl enable mysql
```

## 安装 & 配置 WordPress

#### 第1步：下载WordPress

切换到`/var/www/html`目录下，下载WordPress，解压

```shell
wget https://cn.wordpress.org/latest-zh_CN.zip
unzip *.zip
rm *.zip
```

如果想直接输IP地址就进入WordPress，把文件夹放在`/var/www/html`目录下；或者重命名成其他名字，如`blog`，通过`IP/blog`地址访问

将文件夹下所有文件权限设置成777，貌似能解决一些权限问题

```shell
chmod -R 777 blog
```

#### 第2步：配置WordPress

在其他电脑的浏览器上进入安装页面，地址：IP/(前缀)/wp-admin/install.php，根据提示输入前文设置的数据库信息即可

## 安装完成

注册账号后，进入你专属的WordPress空间尽情折腾吧！

## 参考文章
[WordPress 官方安装指南](https://developer.wordpress.org/advanced-administration/before-install/howto-install/)  
[linux服务器怎么搭建网站（步骤）](https://zhuanlan.zhihu.com/p/23008505639)  
