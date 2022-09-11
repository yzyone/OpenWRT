# /etc/rc.common的解释【转】

（转自：[/etc/rc.common的解释_June_Hou的博客-CSDN博客_/bin/sh /etc/rc.common](https://blog.csdn.net/u012385733/article/details/84565446)）

1、在shell脚本的头部出现 "#!/bin/sh /etc/rc.common"，如果该脚本为x.sh，如果执行命令为 ./x.sh parameters，即为运行 /etc/rc.common x.sh parameters，这说明 /etc/rc.common用于解析x.sh命令行参数

由于openwrt使用自己的初始script系统，所有的initscript必须使用/etc/rc.common作为wrapper安装在/etc/init.d/<name>下。

如/etc/init.d/httpd：

```bash
#!/bin/sh /etc/rc.common 
# Copyright (C)2006 OpenWrt.org 
START=50 
start() {     
[ -d /www ] && httpd -p 80 -h /www-r OpenWrt 
} 
stop() {     
killall httpd 
}
```

从上可以看出，script本身并不解析命令行参数，而是由**/etc/rc.common**来完成。

start和stop是最基本的功能，几乎所有init script都要提供。start是在**运行/etc/init.d/httpd start时**来调用，可以是系统启动时，或用户手动运行此脚本时。

通过/etc/init.d/<name> enable/disable可以**启用或禁止**模块的初始化脚本，他是通过创建或删除/etc/rc.d中的符号连接来完成，而这些符号连接是/etc/init.d/rcS在启动阶段处理。

脚本运行的顺序是在初始脚本中通过变量START来定义，**改变后需要再次运行/etc/init.d/<name>enable**。

- /etc/rc.common说明

```bash
#!/bin/sh
# Copyright (C) 2006-2012 OpenWrt.org
 
. $IPKG_INSTROOT/lib/functions.sh
. $IPKG_INSTROOT/lib/functions/service.sh
 
initscript=$1
action=${2:-help}
shift 2
 
start() {
        return 0
}
 
stop() {
        return 0
}
 
reload() {
        return 1
}
 
restart() {
        trap '' TERM
        stop "$@"
        start "$@"
}
 
boot() {
        start "$@"
}
 
shutdown() {
        stop
}
 
disable() {    #禁止服务
        name="$(basename "${initscript}")"
        rm -f "$IPKG_INSTROOT"/etc/rc.d/S??$name
        rm -f "$IPKG_INSTROOT"/etc/rc.d/K??$name
}
 
enable() {    #开启服务
        name="$(basename "${initscript}")"
        disable
        [ -n "$START" -o -n "$STOP" ] || {
                echo "/etc/init.d/$name does not have a START or STOP value"
                return 1
        }
        [ "$START" ] && ln -s "../init.d/$name" "$IPKG_INSTROOT/etc/rc.d/S${START}${name##S[0-9][0-9]}"
        [ "$STOP"  ] && ln -s "../init.d/$name" "$IPKG_INSTROOT/etc/rc.d/K${STOP}${name##K[0-9][0-9]}"
}
 
enabled() {
        name="$(basename "${initscript}")"
        [ -x "$IPKG_INSTROOT/etc/rc.d/S${START}${name##S[0-9][0-9]}" ]
}
 
depends() {
        return 0
}
 
help() {
        cat <<EOF
Syntax: $initscript [command]
Available commands:
        start   Start the service
        stop    Stop the service
        restart Restart the service
        reload  Reload configuration files (or restart if that fails)
        enable  Enable service autostart
        disable Disable service autostart
$EXTRA_HELP
EOF
}
 
# for procd
start_service() {
        return 0
}
 
stop_service() {
        return 0
}
 
service_triggers() {
        return 0
}
 
service_running() {
        return 0
}
 
${INIT_TRACE:+set -x}
 
. "$initscript"    #引用脚本
 
[ -n "$USE_PROCD" ] && {
        EXTRA_COMMANDS="${EXTRA_COMMANDS} running trace"
 
        . $IPKG_INSTROOT/lib/functions/procd.sh
        basescript=$(readlink "$initscript")
        rc_procd() {
                procd_open_service "$(basename ${basescript:-$initscript})" "$initscript"
                "$@"
                procd_close_service
        }
 
        start() {
                rc_procd start_service "$@"
                if eval "type service_started" 2>/dev/null >/dev/null; then
                        service_started
                fi
        }
 
        trace() {
                TRACE_SYSCALLS=1
                start "$@"
        }
 
        stop() {
                stop_service "$@"
                procd_kill "$(basename ${basescript:-$initscript})" "$1"
        }
 
        reload() {
                if eval "type reload_service" 2>/dev/null >/dev/null; then
                        reload_service "$@"
                else
                        start
                fi
        }
 
        running() {
                service_running "$@"
        }
}
 
ALL_COMMANDS="start stop reload restart boot shutdown enable disable enabled depends ${EXTRA_COMMANDS}"    #所有命令，包括标准的，及定制的
list_contains ALL_COMMANDS "$action" || action=help
[ "$action" = "reload" ] && action='eval reload "$@" || restart "$@" && :'
$action "$@"
```

- 重载初始化脚本函数

可以通过如下方式覆盖这些标准的初始化脚本函数：

{boot()}，boot时支持的命令，缺省为start。

        Commands to be run at boot time.Defaults to {start()}

{restart()} 重启动服务，缺省为stop然后再start。

        Restart your service. Defaults to{stop(); start()}

{reload()} 重新载入配置文件，缺省是restart。

        Reload the configuration files for yourservice. Defaults to {restart()}

- 定制脚本命令

也可定制命令，创建功能函数，在EXTRA_COMMANDS变量中引用，Helptext添加在EXTRA_HELP中。

如下：

```bash
status() {
 
    # print the status info
 
}
 
EXTRA_COMMANDS="status"
 
EXTRA_HELP="        status Print the status of the service"
```

在/etc/rc.common中可以看出，会包含此脚本，从而包含了其中的所有定义，从而可正确地去使用。

_____________________________________________________________________________________________

例如：

parse.sh

```bash
#!/bin/sh
 
. $1
start_test
```

test.sh

```bash
#!/bin/sh  ./parse.sh
 
 
start_test()
{
        echo "hello，world!"
}
```

执行： ./test.sh




