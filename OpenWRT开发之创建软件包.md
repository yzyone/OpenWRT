# OpenWRT开发之创建软件包 #

OpenWRT二次开发时总免不了开发自己的软件包。本文介绍如何在OpenWRT中创建一个新的软件包。

首先创建软件包所在的目录，在openwrt根目录中执行:

	mkdir -p package/mypackages/helloworld

这里的mypackages目录和helloworld目录都是新建的，helloworld就是我们本次新建的软件包的包名。我们后续可以将自己创建的包都放在mypackages目录下。

helloworld包的目录结构如下：

```
helloworld
├── Makefile  #openwrt’s package manifest file
└── src
    ├── helloworld.c  #helloworld source code
    └── Makefile  #helloworld’s makefile
```

**package manifest file**

即软件包helloworld目录下的Makefile文件。例子以及注释如下：

```
# 导入通用编译规则
include $(TOPDIR)/rules.mk

# name和version用来定义编译目录名$(PKG_BUILD_DIR)]
PKG_NAME:=helloworld
PKG_VERSION:=1.0
PKG_RELEASE:=1
#PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)  # 也可以直接定义编译目录名，代替默认的目录名

# 导入包定义
include $(INCLUDE_DIR)/package.mk

# 包定义：定义我们的包在menuconfig中的位置
# Makefile中的define语法可以理解为函数，用于定义命令集合
define Package/helloworld
  SECTION:=examples
  CATEGORY:=Examples
  TITLE:=helloworld, learn from example.
endef

# 包描述：关于我们包的更详细的描述
define Package/helloworld/description
  A simple helloworld example, my first openwrt package example.
endef

# 编译准备. 必须使用tab缩进，表示是可执行的命令
define Build/Prepare
    echo "Here is Build/Prepare"
    mkdir -p $(PKG_BUILD_DIR)
    cp ./src/* $(PKG_BUILD_DIR)/
endef

# 安装
define Package/helloworld/install
    $(INSTALL_DIR) $(1)/usr/bin
    $(INSTALL_BIN) $(PKG_BUILD_DIR)/helloworld $(1)/usr/bin
endef

# 这一行总是在最后
$(eval $(call BuildPackage,helloworld))
```

上面的例子中没有定义define Build/Compile，表示使用默认的Compile命令。默认的Compile行为就是在$(PKG_BUILD_DIR)目录下执行make命令。

**helloworld.c及其Makefile**

helloworld.c内容如下：

```
#include<stdio.h>
int main(void)
{
    printf("Hello world!\n");
    printf("This is my first package!\n");
    return 0;
}
```

与helloworld.c同目录的Makefile内容如下：

```
TARGET = helloworld
OBJS = helloworld.o

$(TARGET):$(OBJS)
    $(CC) $(LDFLAGS) -o $@ $^

%.o: %.c
    $(CC) $(CFLAGS) -c $< -o $@

.PHONY: clean
clean:
    rm -f $(TARGET) $(OBJS)
```

说明：这里的(CFLAGS)、$(LDFLAGS)都是由OpenWRT的build系统赋值的，CC就是目标平台对应的交叉编译工具链里的gcc。

**测试**

在OpenWRT根目录下运行make menuconfig，可以看到多出来一个"Examples --->"菜单，按回车进去后可以看到我们新建的"helloworld" 包。
（从这里也可以看出，在执行make menuconfig时，OpenWRT会自动扫描package目录以及其子目录下所有的包。）

选中这个"helloworld"包。然后再OpenWRT根目录下执行：

	make package/helloworld/compile V=s

此命令即为OpenWRT单package编译命令。

通过log，可以看到我们的包编译成功。编译目录为
build_dir/target-XXXX/helloworld-1.0

如果要再次编译，可以执行：

	make package/helloworld/{clean,compile} V=s

本文源码见：

https://github.com/jian-soft/openwrt-package-example



参考文章：

- https://openwrt.org/docs/guide-developer/packages
- https://openwrt.org/zh-cn/doc/devel/packages
- https://openwrt.org/docs/guide-developer/toolchain/use-buildsystem
- https://openwrt.org/docs/guide-developer/helloworld/start
- https://github.com/mwarning/openwrt-examples

> 作者：Jiansoft
> 链接：https://www.jianshu.com/p/72f47efa5894
> 来源：简书
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。