title: 树莓派安装 Node
date: 2016-06-26 16:23:04
tags: 
- 树莓派
- Node.js

thumbnail: http://7xpox6.com1.z0.glb.clouddn.com/image/stock-photo-138112499.jpg?imageView2/1/w/200
banner: http://7xpox6.com1.z0.glb.clouddn.com/image/stock-photo-138112499.jpg?imageView2/1/w/1024/h/460 

---


## 安装Node

### 快速安装

国外有个哥们为树莓派的Node.js制作了一个安装包，可以非常方便的安装，但是有一个缺点是版本不是最新。

<!-- more -->

```
wget http://node-arm.herokuapp.com/node_latest_armhf.deb 
sudo dpkg -i node_latest_armhf.deb
```

### 官方渠道安装

官方有两个版本可以选择，LTS版和Current版，选择官方推荐LTS版。然后该选择那个平台的包呢。

运行查看本机的CPU信息

```
cat /proc/cpuinfo
```

可以得到类似下面的输出

```
processor	: 0
model name	: ARMv7 Processor rev 5 (v7l)
BogoMIPS	: 38.40
Features	: half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva idivt vfpd32 lpae evtstrm
CPU implementer	: 0x41
CPU architecture: 7
CPU variant	: 0x0
CPU part	: 0xc07
CPU revision	: 5

processor	: 1
model name	: ARMv7 Processor rev 5 (v7l)
BogoMIPS	: 38.40
Features	: half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva idivt vfpd32 lpae evtstrm
CPU implementer	: 0x41
CPU architecture: 7
CPU variant	: 0x0
CPU part	: 0xc07
CPU revision	: 5

...

```

根据输出可以确定我们该选择`armv7`的包。下载相应的包(当前LTS版最新为4.4.5)

```
wget https://nodejs.org/dist/v4.4.5/node-v4.4.5-linux-armv7l.tar.gz
```

下载完成后直接将包解压到对应的目录，并且重命名

```
sudo tar -xzf node-v4.4.5-linux-armv7l.tar.gz -C /usr/local/
mv /usr/local/node-v4.4.5-linux-armv7l /usr/local/node
```

PS： 如果你下载的是老版本的话，可能需要自行编译

```
./configure
make
sudo make install
```

添加到系统路径中

```
sudo vim ~/.bashrc
```

在最后一行加上

```
PATH=$PATH:/usr/local/node/bin
```

保存以后运行`source ~/.bashrc`更新命令行

### 检测安装

```
pi@raspberrypi:~ $ node -v
v4.4.5
pi@raspberrypi:~ $ npm -v
2.15.5
```

## 安装cnpm

```
npm install cnpm -g --registry=https://registry.npm.taobao.org
```

## 安装LoopBack

```
cnpm install -g strongloop
```

安装失败，原因不明，试试`npm`，安装成功。（对于cnpm和npm的差别不是很了解，不过自己平时在安装的时候可以先用`cnpm`安装，不行的话再尝试`npm`）

```
npm install -g strongloop
```

安装以后，就可以根据官方提供的[文档](http://loopback.io/getting-started/)创建Hello World


## 参考资料

- [http://node-arm.herokuapp.com/](http://node-arm.herokuapp.com/)
- [Install Node on the Raspberry Pi in 5 minutes](http://joshondesign.com/2013/10/23/noderpi)
- [https://nodejs.org/en/](https://nodejs.org/en/)
- [淘宝 NPM 镜像](https://npm.taobao.org/)
