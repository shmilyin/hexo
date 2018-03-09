title: HHVM配置及简单性能测试
date: 2018-03-08 14:53:42
categories:
- notes
tags:
- php
---


# HHVM配置及简单性能测试

##### 机器配置如下：
>OS: Ubuntu 16.04 xenial
>Kernel: x86_64 Linux 4.13.0-36-generic
>CPU: Intel Xeon CPU E3-1230 v3 @ 3.3GHz
>RAM: 720MiB / 3001MiB
##### PHP Version：
>PHP 7.0.22-0ubuntu0.16.04.1 (cli) ( NTS )
>Copyright (c) 1997-2017 The PHP Group
>Zend Engine v3.0.0, Copyright (c) 1998-2017 Zend Technologies
    >    with Zend OPcache v7.0.22-0ubuntu0.16.04.1, Copyright (c) 1999-2017, by Zend Technologies

##### hvm --version
>HipHop VM 3.11.1 (rel)
>Compiler: 3.11.1+dfsg-1ubuntu1
>Repo schema: 2f678922fc70b326c82e56bedc2fc106c2faca61

## 1、安装LNMP/LAMP服务器
>性能测试对比使用，根据具体需要安装

## 2、安装HHVM 
>[文档地址](https://www.digitalocean.com/community/tutorials/how-to-install-hhvm-with-nginx-on-ubuntu-14-04)

### Installation:

For Ubuntu 14.04 there is an officially supported HHVM repository. To add this repository you have to import its GnuPG public keys with the command:
`sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0x5a16e7281be7a449`

After that you can safely install HHVM's repository with the command:
`sudo add-apt-repository "deb http://dl.hhvm.com/ubuntu $(lsb_release -sc) main"`

Once you have the repository added you have to make apt, Ubuntu's software manager, aware that there are new packages which can be installed with it. This can be done by updating apt's cache with the command:
`sudo apt-get update`

Finally, you can install HHVM with the command:
`sudo apt-get install hhvm`

The above command installs HHVM and starts it for the first time. To make sure HHVM starts and stops automatically with the Droplet, add HHVM to the default runlevels with the command:
`sudo update-rc.d hhvm defaults`

## 3、配置HHVM
```shell
; php options                                                                                
; hhvm specific                                                                              

;php-fpm使用的端口号是9000
;端口号可sock 选用其中1个
hhvm.server.port = 9003 
;hhvm.server.file_socket=/var/run/hhvm/hhvm.sock
hhvm.server.type = fastcgi                                                                   
hhvm.server.default_document = index.php                                                     
hhvm.server.source_root=/home/wwwroot/default                                                
                                                                                             
hhvm.log.use_log_file = false                                                                
hhvm.log.use_syslog = true                                                                   
hhvm.repo.central.path = /var/cache/hhvm/hhvm.hhbc 
```
配置完成后启动
`sudo service hhvm start`
可以使用 `ps -ef | grep hhvm` 查看是否启动成功，`netstat -na | grep 9003` 查看端口是否正常监听

## 4、配置nginx php配置
>我这里使用的是lnmp,php配置文件是`/usr/local/nginx/conf/enable-php.conf`,其他同理
```json
        #location ~ \.php$
        location ~ \.(hh|php)$ #添加.hh文件解析
        {
            try_files $uri =404;
            fastcgi_keep_conn on;
            #fastcgi_pass  unix:/tmp/php-cgi.sock; #php-fpm sock地址
            #fastcgi_pass 127.0.0.1:9000; #php-fpm端口号
            fastcgi_pass 127.0.0.1:9003; #hhvm端口号
            fastcgi_index index.php;
            include fastcgi.conf;
        }
```
ubuntu: `sudo service nginx restart` 
centos: `/usr/sbin/nginx -s reload`
`lnmp nginx restart` #如果使用的是lnmp重启nginx

## 5、查看phpinfo是否配置成功
在网站目录下新建`info.php`文件
```php
<?php
phpinfo();
?>
```

`sudo chown www-data: /usr/share/nginx/html/info.php`

访问phpinfo `http://your_server_ip/info.php` 显示如下信息则配置成功
![hhvm info](https://assets.digitalocean.com/articles/HHVM_ubuntu1404/HHVMinfo.png)

## 6、附上简单的性能测试对比

###HHVM 下
```
➜  default webbench-c 100 -t 60 http://wordpress.demo/ 
Webbench - Simple Web Benchmark 1.5
Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.

Request:
GET / HTTP/1.0
User-Agent: WebBench 1.5
Host: wordpress.demo


Runing info: 100 clients, running 60 sec.

Speed=3136 pages/min, 896948 bytes/sec.
Requests: 3136 susceed, 0 failed.
➜  default webbench -c 1000 -t 60 http://wordpress.demo/
Webbench - Simple Web Benchmark 1.5
Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.

Request:
GET / HTTP/1.0
User-Agent: WebBench 1.5
Host: wordpress.demo


Runing info: 1000 clients, running 60 sec.

Speed=183728 pages/min, 967769 bytes/sec.
Requests: 183728 susceed, 0 failed.
➜  default webbench -c 10000 -t 60 http://wordpress.demo/
Webbench - Simple Web Benchmark 1.5
Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.

Request:
GET / HTTP/1.0
User-Agent: WebBench 1.5
Host: wordpress.demo


Runing info: 10000 clients, running 60 sec.

Speed=428134 pages/min, 2219073 bytes/sec.
Requests: 428116 susceed, 18 failed.
```

###php-fpm模式下

```
➜  default webbench -c 100 -t 60 http://wordpress.demo/
Webbench - Simple Web Benchmark 1.5
Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.

Request:
GET / HTTP/1.0
User-Agent: WebBench 1.5
Host: wordpress.demo


Runing info: 100 clients, running 60 sec.

Speed=625 pages/min, 178648 bytes/sec.
Requests: 625 susceed, 0 failed.
➜  default webbench -c 1000 -t 60 http://wordpress.demo/
Webbench - Simple Web Benchmark 1.5
Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.

Request:
GET / HTTP/1.0
User-Agent: WebBench 1.5
Host: wordpress.demo


Runing info: 1000 clients, running 60 sec.

Speed=778 pages/min, 171166 bytes/sec.
Requests: 778 susceed, 0 failed.

➜  default webbench -c 10000 -t 60 http://wordpress.demo/
Webbench - Simple Web Benchmark 1.5
Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.

Request:
GET / HTTP/1.0
User-Agent: WebBench 1.5
Host: wordpress.demo


Runing info: 10000 clients, running 60 sec.

Speed=288 pages/min, 46903 bytes/sec.
Requests: 288 susceed, 0 failed.
```

默认配置下性能差别5-6倍