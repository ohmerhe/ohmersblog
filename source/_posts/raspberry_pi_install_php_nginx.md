title: 树莓派安装Nginx和PHP(记)
date: 2016-06-10 20:56:08
tags: 
- 树莓派
- Nginx
- Php
- Raspberry Pi

thumbnail: http://7xpox6.com1.z0.glb.clouddn.com/image/stock-photo-138104577.jpg?imageView2/1/w/200
banner: http://7xpox6.com1.z0.glb.clouddn.com/image/stock-photo-138104577.jpg?imageView2/1/w/1024/h/460 

---


## 安装Nginx

先将系统更新到最新状态

```
sudo apt-get update && sudo apt-get upgrade
```

<!-- more -->

安装Nginx

```
sudo apt-get install nginx -y
```

启动Nginx

```
sudo /etc/init.d/nginx start
```

如果启动成功的话，我们会得到类似的返回
 
```
pi@raspberrypi:~ $ sudo /etc/init.d/nginx start
[ ok ] Starting nginx (via systemctl): nginx.service.
```

测试远程访问，在浏览器里输入树莓派的ip地址（如果你为你的树莓派配置了域名，也可以通过域名访问），可以看到如下的显示效果：

![nginx_install_success](http://7xpox6.com1.z0.glb.clouddn.com/image/nginx_install_success.png)

## 安装PHP

配合Nginx使用时，PHP的安装包和Apache2配合使用稍微有些不同，PHP以FastCGI接口方式运行，因此我们需要安装`PHP FPM`包。

```
sudo apt-get install php5-fpm -y
```

安装完成后会自动启动PHP服务，或者我们可以通过下面的命令启动。

```
sudo service php5-fpm start
```

## 配置Nginx和PHP

在Nginx和PHP都安装完成以后，我们需要进行一些简单的配置。默认Nginx的配置信息是放在`/etc/nginx/sites-available/default`中，我们可以将这个默认文件先备份一下。

```
sudo mv /etc/nginx/sites-available/default /etc/nginx/sites-available/default.bak
```

然后，然后重新创建一个`/etc/nginx/sites-available/default`文件，将下面的配置信息添加到该文件中

```
sudo nano /etc/nginx/sites-available/default
```

```
server {
    listen 80;
    server_name www.myserver.com;
    root /var/www;
    index index.html index.htm index.php;
    access_log /var/log/nginx/myserver.log;

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;

        # With php5-cgi alone:
        #fastcgi_pass 127.0.0.1:9000;
        # With php5-fpm:
        fastcgi_pass unix:/var/run/php5-fpm.sock;
    }
}
```

### 创建index.php

```
sudo vim /var/www/index.php
```

将下面的信息添加到文件中

```
<?php phpinfo(); ?>
```

### 重启服务

在上面的配置都完成后，重启Nginx服务

```
sudo service nginx restart
```

服务启动后，可以在浏览器访问该服务，看到如下的效果。

![](http://7xpox6.com1.z0.glb.clouddn.com/image/nginx_php_pass.png)

PS： 如果在Nginx启动过程中出现问题，可以通过`nginx -t`查看是什么配置出了问题

### PHP配置

我们需要编辑`/etc/php5/fpm/php.ini`（为了安全性）。

```
sudo vim /etc/php5/fpm/php.ini
```
然后找到`cgi.fix_pathinfo=1`这一行，并将其改成`cgi.fix_pathinfo=0`。

重启PHP服务和Nginx服务

```
sudo service php5-fpm restart
```

## [参考资料]

- [Tutorial – Install Nginx and PHP on Raspbian](https://www.stewright.me/2014/06/tutorial-install-nginx-and-php-on-raspbian/)
- [Raspberry Pi – Install nginx, MySQL & PHP](https://kevindekoninck.com/raspberry-pi-install-nginx-mysql-php/)
- [How to setup a web server with Nginx/PHP on Raspberry Pi](http://workshop.botter.ventures/2013/09/05/how-to-setup-a-web-server-with-nginxphp-on-raspberry-pi/)
- [Can't start Nginx - Job for nginx.service failed](https://www.digitalocean.com/community/questions/can-t-start-nginx-job-for-nginx-service-failed)
