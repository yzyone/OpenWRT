# 编译错误 error New address family defined, please update secclass_map.解决 #

In file included from scripts/selinux/genheaders/genheaders.c:19:

```
./security/selinux/include/classmap.h:245:2: error: #error New address family defined, please update secclass_map.
#error New address family defined, please update secclass_map.
-git a/scripts/selinux/mdp/mdp.c b/scripts/selinux/mdp/mdp.c:
xxxxxxxxxxxxxxxxxx
```

```
In file included from scripts/selinux/genheaders/genheaders.c:19:
./security/selinux/include/classmap.h:245:2: error: #error New address family defined, please update secclass_map.
  245 | #error New address family defined, please update secclass_map.
      |  ^~~~~
make[3]: *** [scripts/Makefile.host:102: scripts/selinux/genheaders/genheaders] Error 1
make[2]: *** [scripts/Makefile.build:585: scripts/selinux/genheaders] Error 2
make[1]: *** [scripts/Makefile.build:585: scripts/selinux] Error 2
make: *** [Makefile:572: scripts] Error 2
```

1.找到错误中提示的.c文件，genheader.c文件和mdp.c文件
编辑

```
vim ./kernel/scripts/selinux/genheaders/genheaders.c
vim ./kernel/scripts/selinux/mdp/mdp.c
```

去掉两个文件的头部引用中的

```
#include <sys/socket.h>
```

2.找到错误提示中的classmap.h文件

```
vim ./kernel/security/selinux/include/classmap.h 
```

编辑classmap.h，在头文件中添加

```
#include <linux/socket.h>
```

重新编译即可

————————————————
版权声明：本文为CSDN博主「孤星入命孑然一身」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/zhangpengfei991023/article/details/109672491