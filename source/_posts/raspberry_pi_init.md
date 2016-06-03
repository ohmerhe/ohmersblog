title: 树莓派入手，无显示器远程操作
date: 2016-05-31 00:11:57
tags: 
- 树莓派
- 安装
- samba
- NAS
- Raspberry Pi

thumbnail: http://7xpox6.com1.z0.glb.clouddn.com/image/lake_forest.jpg?imageView2/1/w/200
banner: http://7xpox6.com1.z0.glb.clouddn.com/image/lake_forest.jpg?imageView2/1/w/1024/h/460 

---


## 下载镜像文件
官网下载相应的镜像文件：<https://www.raspberrypi.org/downloads/>

也可以下载Centos系统：<http://mirror.centos.org/altarch/7/isos/armhfp/>

我这里下载的是官网推荐的"RASPBIAN WHEEZY"

<!-- more -->

## 烧录镜像文件

在mac下可以参考[Mac OSX下给树莓派安装Raspbian系统](http://shumeipai.nxez.com/2014/05/18/raspberry-pi-under-mac-osx-to-install-raspbian-system.html)

在Windows下面的话直接使用`Win32 Disk Imager`进行烧录。

## 初始化系统操作

### 激活root用户

raspberrypi系统默认提供的用户为`pi`，密码为`raspberry`，so，第一步我们就是要激活`root`帐号：

```
// 设置 root 账号的密码
sudo passwd root

// 启用 root 账号登录
sudo passwd --unlock root
```
但是需要注意的是，root帐号启用后，默认不支持远程登录。

### 配置ssh key证书登录

为了减去每次远程登录系统都需要手动输入密码，同样也更加安全，可以配置ssh key登录，ssh key的生成参考[generating-ssh-keys](https://help.github.com/articles/generating-ssh-keys/)

在服务端生成`.ssh/authorized_keys`文件

```
mkdir .ssh
touch .ssh/authorized_keys
```

将本地的公匙复制到`.ssh/authorized_keys`文件中

```
echo XXXX >> .ssh/authorized_keys #XXXX替换成对应的公匙
```

或者直接执行

```
cat ~/.ssh/id_rsa.pub | ssh pi@192.168.199.121 'mkdir -p .ssh && cat - >> ~/.ssh/authorized_keys'
```

### 网络配置
执行`ifconfig`命令,查看网卡物理地址，将该网址在路由器上绑定到固定的ip

```
eth0      Link encap:Ethernet  HWaddr 38:22:db:e7:ed:0c // 物理地址
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```
然后执行`sudo nano /etc/network/interfaces`，将`manual`改为`dhcp`

```
iface eth0 inet dhcp
```

如果你有无线网卡的话，还要进行无线网络的配置，首先一样的需要将无线网络的网卡地址在路由器上绑定到一个固定ip。

```
wlan0     Link encap:Ethernet  HWaddr e2:42:06:3e:w5:4e  // 物理地址
```

然后需要将无线网络的名称和密码设置一下

```
iface wlan0 inet dhcp
wpa-ssid 网络名称
wpa-psk 网络密码
#wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
```

### 扩大系统容量

树莓派默认并没有完全加载SD卡的内存容量，通过运行`sudo raspi-config`调出树莓派的配置界面

选择第一项`Expand Filesystem`,确认重启后可以再执行`df -h`查看系统容量已经增大

### 替换软件源的国内镜像

国内有许多平台提供了树莓派的软件源的国内镜像，具体参考[镜像列表](https://segmentfault.com/a/1190000000503041)。这里选择阿里云的镜像地址：

```
cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo vi /etc/apt/sources.list

//将原来的注释掉，替换成如下软件源地址
deb http://mirrors.aliyun.com/raspbian/raspbian/ jessie main non-free contrib
deb-src http://mirrors.aliyun.com/raspbian/raspbian/ jessie main non-free contrib
```

PS：有些库国内镜像并没有及时同步，如果安装软件出现问题时，可以尝试再把镜像地址换回官方镜像尝试一下

### 安装常用软件

```
sudo apt-get install vim wget -y
```

## 安装samba

最后也来装个家庭内部NAS服务玩玩

```
// 更新源
sudo apt-get update
// 安装samba
sudo apt-get install samba samba-common-bin -y
```

创建samba文件夹

```
mkdir /samba

// 修改samba的访问权限
sudo chmod 777 /samba
```

安装好samba后，需要配置samba分享的信息，编辑`/etc/samba/smb.conf`文件，在文件的末尾添加如下内容：

```
[share]
   comment = share folder
   path = /samba
   browseable = yes
   valid users = root pi
   public = yes
   writable = yes
```

```
# Path to the directory you want scanned for media files.
#
# This option can be specified more than once if you want multiple directories
# scanned.
#
# If you want to restrict a media_dir to a specific content type, you can
# prepend the directory name with a letter representing the type (A, P or V),
# followed by a comma, as so:
#   * "A" for audio    (eg. media_dir=A,/var/lib/minidlna/music)
#   * "P" for pictures (eg. media_dir=P,/var/lib/minidlna/pictures)
#   * "V" for video    (eg. media_dir=V,/var/lib/minidlna/videos)
#
# WARNING: After changing this option, you need to rebuild the database. Either
#          run minidlna with the '-R' option, or delete the 'files.db' file
#          from the db_dir directory (see below).
#          On Debian, you can run, as root, 'service minidlna force-reload' instead.
media_dir=/samba/DLNA
media_dir=A,/samba/DLNA/Music
media_dir=P,/samba/DLNA/Picture
media_dir=V,/samba/DLNA/Video

# Path to the directory that should hold the database and album art cache.
db_dir=/samba/DLNA/db

# Path to the directory that should hold the log file.
log_dir=/samba/DLNA/log
```

```
$ sudo /etc/init.d/minidlna restart
//查看状态
$ sudo /etc/init.d/minidlna status 
[ ok ] minidlna is running.
```

```
sudo vim /etc/rc.local
```

```
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.

# Print the IP address
_IP=$(hostname -I) || true
if [ "$_IP" ]; then
  printf "My IP address is %s
" "$_IP"
fi

#==============================
sudo /etc/init.d/samba restart
sudo /etc/init.d/minidlna restart
#==============================

exit 0
```

## 参考内容

- [树莓派及其他硬件平台国内外Linux镜像站全汇总](https://segmentfault.com/a/1190000000503041)
- [Mac OSX下给树莓派安装Raspbian系统](http://shumeipai.nxez.com/2014/05/18/raspberry-pi-under-mac-osx-to-install-raspbian-system.html)
