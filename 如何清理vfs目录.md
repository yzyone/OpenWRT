# 如何清理/var/lib/docker/vfs目录 #

我正在尝试弄清楚如何从Docker那里回收一些磁盘空间。我的/var/lib/docker目录当前如下所示：

```
42M     containers
3.6G    devicemapper
14M     image
68K     network
20K     plugins
4.0K    swarm
4.0K    tmp
4.0K    trust
9.0G    vfs
28K     volumes
```

我不知道为什么我的vfs目录这么大，也不知道如何清理它！我在其他服务器上较新安装的docker甚至没有这个目录，所以删除它安全吗？

我在Ubuntu 14.04服务器上运行docker 1.13.1，使用的是devicemapper存储驱动程序。这个docker在过去已经升级过几次了。我只在上面运行了3个容器，并且大多数重要的卷都挂载到了主机上。

我已经运行了"`docker system prune`“，但是它没有回收任何东西。我也清理了所有我不需要的图片。

在`docker system prune -a --volumes`

并检查`docker system df`和`docker container ls -a`没有运行任何东西之后，

我简单地删除了vfs目录：`rm - r vfs/dir/3213412.../`。

如果你查看这些文件夹，你会发现它们基本上都是容器的运行时文件。我不知道为什么这些东西在整个系统都被修剪后还在那里...但我干脆摆脱了他们，赢回了10G。