title: 用 U 盘为树莓派备份镜像
date: 2016-06-25 21:02:22
tags: 
- 树莓派

thumbnail: http://7xpox6.com1.z0.glb.clouddn.com/image/stock-photo-156596865.jpg?imageView2/1/w/200
banner: http://7xpox6.com1.z0.glb.clouddn.com/image/stock-photo-156596865.jpg?imageView2/1/w/1024/h/460 

---


树莓派使用了一段时间以后，已经为树莓派装了好多东西，也做了好多配置。有必要为系统做一次备份，就不用每次都重头开始了。

<!-- more -->

准备工作：
- 空白U盘（最好是8G以上，也可以是sd卡+读卡器）
- 可以上网的树莓派

## 下载脚本

进入树莓派系统，下载脚本文件：

```
wget https://raw.githubusercontent.com/billw2/rpi-clone/master/rpi-clone
```

为脚本设置执行权限：

```
sudo chmod +x rpi-clone
```

## U盘准备

在插入u盘前，先运行`sudo fdisk -l`查看树莓派的磁盘。SD卡插槽上正在运行系统的SD卡应该显示为/dev/mmcblk0。

```
Device         Boot  Start      End  Sectors  Size Id Type
/dev/mmcblk0p1        8192   137215   129024   63M  c W95 FAT32 (LBA)
/dev/mmcblk0p2      137216 31116287 30979072 14.8G 83 Linux
```

插入U盘，再运行上面的命令，可以看到增加了下面的内容。

```
Device     Boot Start      End  Sectors  Size Id Type
/dev/sda1  *       64 15771647 15771584  7.5G  b W95 FAT32
```

/dev/sda1（或者是/dev/sdb1）就是我们插入的空白U盘。

## 备份镜像

运行命令

```
sudo ./rpi-clone sda -f
```

这里，第一个参数是SD卡的名字，这里是`sda`。`-f`告诉脚本完整格式化SD卡。

脚本会提示你是否初始化目标SD卡。输入`yes`然后按一下回车。

![](http://7xpox6.com1.z0.glb.clouddn.com/image/raspberry_backup_1.png)

接着，会提示你是否想给你的备份镜像指定一个标签，你可以设置一个标签，或者直接回车跳过。

![](http://7xpox6.com1.z0.glb.clouddn.com/image/raspberry_backup_2.png)

最后，会有一次最终确认，输入`yes`回车，等待一段时间。

![](http://7xpox6.com1.z0.glb.clouddn.com/image/raspberry_backup_3.png)

备份完成后，会提示你是否卸载(unmount)这个新系统卡，输入`yes`回车完成备份。

到这里整个备份已经完成

## 镜像还原

树莓派本身不支持从U盘启动，所以经过上面的步骤备份好的系统要还原使用，还需要进一步处理。

一种方式是让你的树莓派支持从U盘启动，具体可以参考[树莓派支持多系统启动](http://www.geekfan.net/5244/)。

另外就是先将系统还原到sd卡中使用。

## 参考资料

- [为树莓派做系统备份镜像](http://shumeipai.nxez.com/2014/06/01/do-system-backup-image-of-raspberry-pi-for-linux-or-mac.html)
- [制作树苺派SD卡备份镜像——树苺派系统备份与还原指南](http://blog.lxx1.com/1450)
- [如何使用BerryBoot来使树莓派支持多系统启动](http://www.geekfan.net/5244/)
