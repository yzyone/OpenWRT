# OpenWrt 快速入门

文章目录
1. OpenWrt 优点
1. OpenWrt 版本发展
1. OpenWrt 源码

缩略语（Acronyms and Abbreviations）

OpenWrt（Wrt：Wirless Router）（官网 www.openwrt.org）是一个嵌入式的 Linux 发行版，目前常用在路由器上。但是作为基于 linux 系统的它，其实可以做更多的事情。

OpenWrt 是一个高度模块化、高度自动化的嵌入式 Linux 系统，拥有强大的网络组件和扩展性。可被用于工控设备、电话、小型机器人、远程监控、智能家居以及 VOIP 设备中。

OpenWrt 不同于其他许多用于路由器的发行版（主流路由器固件有 DD-WRT，Tomato，OpenWrt 三类），它是一个从零开始编写的、功能齐全的、容易修改的路由器操作系统。

## 1、OpenWrt 优点 ##

你不需要对 MIPS 处理器有很深入的了解，也不用懂得如何去设计一个 ARM 或 MIPS处理器专用的 linux 内核，因为这些移植工作在 OpenWrt 里已有人为你做好，你只需懂得如何安装和使用 OpenWrt就行了，不过你也可以去http://www.linux-mips.org 找到相关的资料。

如果你对 Linux 系统有一定的认识，并想学习或接触嵌入式 Linux 的话，OpenWrt 很适合你，你将学会一些无线路由器的基本知识，以及一般嵌入式 Linux 的开发过程。但凡做过或者了解过嵌入式开发的人，都知道无论是 ARM，PowerPC 或 MIPS 的处理器，都必需经过以下的开发过程：

1. 创建 Linux 交叉编译环境
1. 建立 Bootloader
1. 移植 Linux 内核
1. 建立 Rootfs (根文件系统)
1. 安装驱动程序
1. 安装软件

采用上面传统的方法进行嵌入式开发，费时费力。我们可以通过 OpenWrt 快速构建一个应用平台，OpenWrt 从交叉编译器，到 Linux 内核，再到文件系统甚至 Bootloader 都整合在了一起，形成了一个SDK环境。
其多达 3000 多种软件包（数量还在增加），囊括从工具链（Toolchain），到内核（Linux Kernel），到软件包（Packages），再到根文件系统（Rootfs）整个体系，使得用户只需简单的一个make命令即可方便快速地定制一个具有特定功能的嵌入式系统来制作固件，大大减少了嵌入式软件开发的工序。

当熟悉这些嵌入式 Linux 的基本开发流程后，你不再局限于MIPS处理器和无线路由器，你可以尝试在其它处理器，或者非无线路由器的系统移植嵌入式 Linux，定制合适自己的应用软件，并建立一个完整的嵌入式产品。

OpenWrt 的成功之处 还在于 它的文件系统是可写的，开发者无需在每一次修改后重新编译系统，并且可以像 PC 机上的 Linux 系统一样，用命令安装一些安装包，不用手动配置，这些都令它更像一个小型的 Linux 电脑系统。

## 2、OpenWrt 版本发展 ##

OpenWrt 所支持的平台包括：Broadcom的SoC，ARM，PowerPC，MIPS 24K R2，x86 。在软件应用上出现了以 LuCI 跟 Webif 为首的 UI 以及各种更新软件包。

```
序号    版本    发布日期    代号
1    10.03.1    2011年12月    Backfire
2    12.09    2013年4月    Attitude Adjustment
3    14.07    2014年10月    Barrier Breaker
4    15.05    2015年9月    Chaos Calmer
5    15.05.1    2016年3月    Chaos Calmer
6    18.06    2018年7月    -
```

## 3、OpenWrt 源码 ##

```
序号    目录和文件    描述
1    /config    存放着整个系统的配置文件
2    /docs    包含了整个宿主机的文件源码的介绍，里面还有Makefile 为目标系统生成 docs。
使用make -C docs/可以为目标系统生成文档
3    /feeds    下载管理软件包的
默认的feeds下载有packages、management、luci、routing、telephony

如要下载其他的软件包，需打开源码根目录下面的feeds.conf.default文件，去掉相应软件包前面的#号，然后更新源：
./scripts/feeds update -a
安装下载好的包：
./scripts/feeds install -a
4    /include    OpenWrt 的很多Makefile都存放在这里，文件名为*.mk
这里的文件是在Makefile里被include的，类似于库文件，这些文件定义了编译过程
5    /package    存放了 OpenWrt 系统中适用的软件包，包含针对各个软件包的Makefile
6    /scripts    存放了一些脚本，使用了 Bash，Python，Perl 等多种脚本语言
编译过程中，用于第三方软件包管理的feeds文件也是在这个目录当中
在编译过程中，使用到的脚本也统一放在这个目录中
7    /target    OpenWrt 的源码可以编译出各个平台适用的二进制文件，各平台在这个目录里定义了firmware和kernel的编译过程
8    /toolchain    存放的就是编译交叉编译链的软件包
包括：binutils，gcc，libc等等
9    /tools    编译时，主机需要使用一些工具软件，tools 里包含了获取和编译这些工具的命令
软件包里面有Makefile文件，有的还包含了patch
每个Makefile当中都有一句$(eval $(call HostBuild))，这表明编译这个工具是为了在主机上使用的
10    Config.in    在include/toplevel.mk中可以看到，这是和make menuconfig相关联的文件
11    feeds.conf.default    可以配置下载第三方软件包时所使用的地址
12    Makefile    在顶层目录执行make命令的入口文件
13    rules.mk    定义了Makefile中使用的一些通用变量和函数
14    LICENSE    软件许可证
15    README    软件基本说明：描述了编译软件的基本过程和依赖文件
```

编译一次 OpenWrt 源码后，会出现一些新的目录：

```
序号    目录    描述
1    /build_dir/host    在该文件夹中编译主机使用的工具软件
2    /build_dir/target-mipsel_24kec+dsp_uClibc-0.9.33.2    在此编译目标平台的目标文件，包括各个软件包和内核文件
3    /build_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2    在该文件夹中编译交叉工具链
4    /bin    保存编译完成后的二进制文件
包括：完整的 bin 文件，所有的ipk文件
5    /dl    在编译过程中使用的很多软件
刚开始下载源码并没有包含，而是在编译过程中从其他服务器下载的
这里是统一的保存目录
6    /staging_dir    用于保存在build_dir目录中编译完成的软件
所以这里也和build_dir有同样的子目录结构
比如：在target-XXX文件夹中保存了目标平台编译好的头文件，库文件
在我们开发自己的 ipk 文件时，编译过程中，预处理头文件，链接动态库，静态库都是到这个子文件夹中
7    /tmp    在编译过程中，有大量中间临时文件需要保存，都是在这里
8    /logs    编译过程中出错的信息，只有当编译出错了才会出现
```

缩略语（Acronyms and Abbreviations）

```
序号    缩写    原语    描述
1    mipsel    mips little endian    mips 小端字节序
2    ramips    Ralink mips    -
3    squash    -    -
4    Wrt    Wirless Router    无线路由器
```

————————————————

版权声明：本文为CSDN博主「deepwater_zone」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/Hongwei_1990/article/details/93791798