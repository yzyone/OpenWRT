openwrt从gitee pull代码并编译go项目

安装ssh

```
opkg update
# openssh-keygen 可以用来生产密钥
opkg install openssh-keygen
# 连接其他服务器
opkg install openssh-client
# 生成密钥，可以指定文件名，注意算法，gitee有不支持SHA1算法。git pull时会报错
ssh-keygen -t rsa -C "xxx@xxxxx@xxx.com"
```

编辑配置 vim ~/.ssh/config

```
Host gitee.com
  User git
  HostName gitee.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/gitee_rsa
gitee 后台配置公钥步骤此处略。
```

安装golang

```
opkg update
opkg golang
```

go build 时报错

```
# runtime/cgo
cgo: C compiler "gcc" not found: exec: "gcc": executable file not found in $PATH
```

指定CGO_ENABLED 为0

```
CGO_ENABLED=0 go build -o main main.go
```

指定后台运行go编译后文件,指定日志文件

```
./main > /var/golang/stock.log   2>&1 &
```