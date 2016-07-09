title: 树莓派安装 MySQL
date: 2016-06-04 00:12:53
tags: 
- 树莓派
- MySQL

thumbnail: http://7xpox6.com1.z0.glb.clouddn.com/image/lovely_fox.jpg?imageView2/1/w/200
banner: http://7xpox6.com1.z0.glb.clouddn.com/image/lovely_fox.jpg?imageView2/1/w/1024/h/460 

---


## 安装

先将系统更新到最新状态

```
sudo apt-get update && sudo apt-get upgrade
```

<!-- more -->

然后安装

```
sudo apt-get install mysql-server
```

接着会提示你为`root`用户设置密码，并且确认密码。输入完成后，稍等mysql就安装完成。可以测试一下。

```
mysql -u root -p
```

然后输入刚刚设置的密码。

## 开启远程登录

### 配置权限

```
sudo nano /etc/mysql/my.cnf
```

找到下一行，并且将`bind-address`的值改成`0.0.0.0`

```
bind-address = 0.0.0.0
```

为root用户开启远程登录权限，并且限制在局域网内

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.1.%' IDENTIFIED BY 'password' WITH GRANT OPTION;
```

重启MySQL服务

```
sudo /etc/init.d/mysql restart
```

### 打开`3306`端口

最后用`iptables`打开`3306`端口

```
iptables -A INPUT -i eth0 -p tcp -m tcp --dport 3306 -j ACCEPT
```

或者限定固定的`ip`才能访问

```
sudo iptables -A INPUT -i eth0 -s 192.168.1.0/24 -p tcp --destination-port 3306 -j ACCEPT
```

### 测试访问

```
echo X | telnet -e X 192.168.1.110 3306
```

或者

```
nc -z -w1 192.168.1.110 3306
```

成功的话，会看到类似的输出

```
Connection to 192.168.199.121 port 3306 [tcp/mysql] succeeded!
```

## 参考资料
- [Tutorial – Install MySQL server on Raspberry Pi](https://www.stewright.me/2014/06/tutorial-install-mysql-server-on-raspberry-pi/)
- [Raspberry Pi MYSQL & PHPMyAdmin Tutorial](https://pimylifeup.com/raspberry-pi-mysql-phpmyadmin/)
- [How Do I Enable Remote Access To MySQL Database Server?](http://www.cyberciti.biz/tips/how-do-i-enable-remote-access-to-mysql-database-server.html)
- [Remote Access to Mysql on PI](https://www.raspberrypi.org/forums/viewtopic.php?f=36&t=20214)
