# 编译更新OpenWrt PassWall和SSR-plus插件

 发表于 2020.05.05 19:59  浏览量 74141  [评论数 53](https://www.mianao.info/2020/05/05/%E7%BC%96%E8%AF%91%E6%9B%B4%E6%96%B0OpenWrt-PassWall%E5%92%8CSSR-plus%E6%8F%92%E4%BB%B6#comments)

~~前面写了自编译 OpenWRT 固件，本来玩的好好的，结果最主要的两个插件据说开发者删库了，只能重新找办法升级了。~~  

都回来了，方法还是好用的  

如果只要更新 Passwall 和 SSR-plus，还有 Clash，那就直接到这里下载 ipk 吧（我编译的插件都不支持 SSR 或 SS）：  
**[https://share.mianao.info/Router/X86-64/](https://mianao.info/go/aHR0cHM6Ly9zaGFyZS5taWFuYW8uaW5mby9Sb3V0ZXIvWDg2LTY0Lw)**  

可以在 **系统**->**文件传输**，直接上传安装 ipk 软件包，如果 openwrt 固件没有这个，那就自行上传了 ssh 命令安装吧。  
上面的文件夹里我也分享了自用的系统固件，一般来说都是比较稳定的，硬件就是去年的那个蜗牛星际的主板。  

**最近看到的 ipk 下载站，还有 openwrt 固件，更新比我快，多：**  
**[https://op.dllkids.xyz/](https://mianao.info/go/aHR0cHM6Ly9vcC5kbGxraWRzLnh5ei8)**  
**下面这个甚至可以网站上自定义固件：**  
**[https://op.supes.top/](https://mianao.info/go/aHR0cHM6Ly9vcC5zdXBlcy50b3Av)**  

以下就可以不用看了。

### 本地自编译

#### 编译 Lienol 源

如果用的源码：[https://github.com/Lienol/openwrt](https://mianao.info/go/aHR0cHM6Ly9naXRodWIuY29tL0xpZW5vbC9vcGVud3J0)  
这个源里有很多常用软件包，大家可以去 fork 下：  
[https://github.com/kenzok8/openwrt-packages](https://mianao.info/go/aHR0cHM6Ly9naXRodWIuY29tL2tlbnpvazgvb3BlbndydC1wYWNrYWdlcw)

添加下面代码到 openwrt 或 lede 源码根目录下的 `feeds.conf.default` 文件：

```linux
src-git kenzo https://github.com/kenzok8/openwrt-packages
src-git small https://github.com/kenzok8/small
```


或者不打开文件编辑直接输入以下命令可以添加到`feeds.conf.default` 文件：

```linux
sed -i '$a src-git kenzo https://github.com/kenzok8/openwrt-packages' feeds.conf.default
sed -i '$a src-git small https://github.com/kenzok8/small' feeds.conf.default
```

**或者**,直接 clone 到 openwrt/package 目录下:

```linux
git clone https://github.com/kenzok8/openwrt-packages.git
git clone https://github.com/kenzok8/small.git
```

然后执行：

```linux
./scripts/feeds update -a
./scripts/feeds install -a
```


接着编译 Passwall 和 SSR-plus 就都有了。

#### Lean's 源

如果用的源码：[https://github.com/coolsnowwolf/lede](https://mianao.info/go/aHR0cHM6Ly9naXRodWIuY29tL2Nvb2xzbm93d29sZi9sZWRl)  
同样的，也可以直接下载这个源的软件包，small 是依赖包：

```linux
cd lede/package
git clone https://github.com/kenzok8/openwrt-packages.git
git clone https://github.com/kenzok8/small.git
```


然后执行：

```linux
./scripts/feeds update -a
./scripts/feeds install -a
```


接着编译 Passwall 和 SSR-plus 就都有了。

**注：**  
如果 feeds update 出现一堆类似下面的警告：

```shell
WARNING: Makefile 'package/lean/shadowsocksR-libev-full/Makefile' has a dependency on 'libpcre', which does not exist
```

解决办法就是删掉 feeds 整个文件夹，在 lede 或 openwrt 目录下执行 `rm -rf ./feeds`，然后再 update。

个人感觉 Lean's 的源码编译不是很好用，时而成功时而不行，原因根本不知道为什么，而 Lienol 的源基本网络没问题就编译没问题。

### GitHub 在线编译

**参考：**[https://p3terx.com/archives/build-openwrt-with-github-actions.html](https://mianao.info/go/aHR0cHM6Ly9wM3RlcnguY29tL2FyY2hpdmVzL2J1aWxkLW9wZW53cnQtd2l0aC1naXRodWItYWN0aW9ucy5odG1s)  
上面这篇文章写得很详细了，我的操作过程说明就直接删除了。

**说明：**

1. 默认情况下触发编译工作流程有两种方式，发布 release 和修改 `.config` 文件，所以无论是点发布还是修改 `.config` 都会自动开始编译。当发现仓库源码有更新时，在 releases 页面发布一个版本就会触发编译的工作流程，使用最新源码进行编译最新固件了。
2. 本方法实际上就是将前面的步骤在本地电脑进行，到最后一步编译命令 `make V=s` 时交给了 GitHub 自动操作，适合网络问题多的情况，后续更新编译也方便。可以看参考文章自定义更多内容。
3. 建议申请新 GitHub 账号，我的账号 actions 功能就被禁用了。
- **推荐 Vultr：**我已使用超过 5 年觉得还算稳定可靠的便宜 VPS，虽然可能会有 IP 被墙，但欧美亚的机房可以随便切换，美国机房最低每月 $3.5：512M 内存 500G 流量。

- **奖励链接：** [**新用户充值有奖励 100 美元 https://www.vultr.com/?ref=9125823-8H**](https://www.vultr.com/?ref=9125823-8H "欢迎使用推荐链接，新用户充值有奖励 100 美元，谢谢!")
