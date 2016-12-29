title: 在树莓派上用 Docker 安装 Gogs
date: 2016-12-29 23:07:43
tags: 
- 树莓派
- Docker
- Git
  
thumbnail: http://7xpox6.com1.z0.glb.clouddn.com/image/stock-photo-170036539.jpg?imageView2/1/w/200
banner: http://7xpox6.com1.z0.glb.clouddn.com/image/stock-photo-170036539.jpg?imageView2/1/w/1024/h/460 

---


这里假设在你的树莓派上已经装好了 Docker，之前装 Docker 的时候稍微折腾了一下，但是遗憾的是，当时没有记录下来，后面有机会再分享这一部分。

通过搜索可以找到 `Gogs` 官方为树莓派提供的 Docker 镜像，直接拉取下来。

```
# Pull image from Docker Hub.
$ sudo docker pull gogs/gogs-rpi:0.9.46

# Create local directory for volume.
$  mkdir -p /var/gogs

# Use `docker run` for the first time.
$ sudo docker run --name=gogs -p 10022:22 -p 10080:3000 -v /var/gogs:/data gogs/gogs-rpi:0.9.46

# Use `docker start` if you have stopped it.
$ sudo docker start gogs
```

这样你就可以通过访问 `localhost:10080` 访问 `Gogs` 页面了（你也可以通过 `ip` 访问）,第一次进去是需要配置一下的。数据库的话，如果你是自己个人用的话，可以直接选 `SQLite`，简单一点。如果是提供给大家的公共服务，可以选择 `MYSQL` 或者 `POSTGRES`。由于使用 Docker 的原因，这里有几个配置需要额外注意。

- `Domain`，访问 `Gogs` 的域名，如果你要提供给外部机器访问的话，这里要填你的机器的 `ip` 或者域名；
- `HTTP Port`，这里要填 `Gogs` 在容器里的监听的端口号，一般是 `3000`；
- `SSH Port`，与上面的相反，这里要填机器暴露给访问者的端口，而不是容器内部 SSH 服务监听的端口，如我们用 `10022:22` 通过 `10022` 端口暴露，则使用 `10022` 这个值；
- `ROOT_URL`，是提供给外部访问网站的公开 URL，如 `http://192.168.99.100:10080/`。

其他的都可以使用默认值。如果在配置完了以后发现有配的不对的地方，也可以通过修改 `/var/gogs/gogs/conf/app.ini` 这个文件，再重启服务。

这些信息配置好以后，就可以进入网站了。 `Gogs` 和 `Gitlab` 有一点差别，默认并不提供一个管理员账号，而是将第一个注册的账号默认设置为管理员账号，这个账号是不需要邮箱验证的。

更多关于 Gogs 在 Docker 的配置信息可以参考 [Docker for Gogs](https://github.com/gogits/gogs/tree/master/docker)，关于更多 Gogs 的配置信息可以参考 [Configuration Cheat Sheet](https://gogs.io/docs/advanced/configuration_cheat_sheet.html)

## 参考资料

- [Docker for Gogs](https://github.com/gogits/gogs/tree/master/docker)
- [Configuration Cheat Sheet](https://gogs.io/docs/advanced/configuration_cheat_sheet.html)
