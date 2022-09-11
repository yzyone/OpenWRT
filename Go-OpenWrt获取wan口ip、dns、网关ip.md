# Go-OpenWrt获取wan口ip、dns、网关ip

## 1. 前言 ##

   一般来说，Openwrt可以配置多个wan口和多个lan口，这里获取的wan口的ip、dns等信息是基于知道wan口名称的前提下使用ifstatus命令来获取解析结果的。

## 2. 解决方案思路 ##

   通过uci命令获取相关网络接口名称，然后利用ifstatus查看对应接口的网络信息后获取，当然也可以直接通过uci接口获取：

```
root@OpenWrt:~# uci show network
network.loopback=interface
network.loopback.ifname='lo'
network.loopback.proto='static'
network.loopback.ipaddr='127.0.0.1'
network.loopback.netmask='255.0.0.0'
network.globals=globals
network.globals.ula_prefix='fdcd:b3fe:8353::/48'
network.lan=interface
network.lan.type='bridge'
network.lan.proto='static'
network.lan.netmask='255.255.255.0'
network.lan.ip6assign='60'
network.lan.ipaddr='40.40.40.100'
network.lan.ifname='eth1 eth2 eth3 eth4'
network.wan=interface
network.wan.ifname='eth7'
network.wan.proto='dhcp'
network.wan_eth=interface
network.wan_eth.ifname='eth0'
network.wan_eth.proto='static'
network.wan_eth.netmask='255.255.255.0'
network.wan_eth.gateway='192.168.67.254'
network.wan_eth.dns='8.8.8.8'
network.wan_eth.ipaddr='192.168.67.87'
network.SFP1=interface
network.SFP1.ifname='eth5'
network.SFP1.proto='static'
network.SFP1.netmask='255.255.255.0'
network.SFP1.dns='8.8.8.8'
network.SFP1.ipaddr='192.168.67.88'
network.SFP1.gateway='192.168.67.1'
```

```
root@OpenWrt:~# uci show network.SFP1.ipaddr
network.SFP1.ipaddr='192.168.67.88'
```

```
root@OpenWrt:~# ifstatus SFP1
{
        "up": true,
        "pending": false,
        "available": true,
        "autostart": true,
        "dynamic": false,
        "uptime": 1629,
        "l3_device": "eth5",
        "proto": "static",
        "device": "eth5",
        "updated": [
                "addresses",
                "routes"
        ],
        "metric": 0,
        "dns_metric": 0,
        "delegation": true,
        "ipv4-address": [
                {
                        "address": "192.168.67.88",
                        "mask": 24
                }
        ],
        "ipv6-address": [

        ],
        "ipv6-prefix": [
    
        ],
        "ipv6-prefix-assignment": [
    
        ],
        "route": [
                {
                        "target": "0.0.0.0",
                        "mask": 0,
                        "nexthop": "192.168.67.1",
                        "source": "0.0.0.0/0"
                }
        ],
        "dns-server": [
                "8.8.8.8"
        ],
        "dns-search": [
    
        ],
        "neighbors": [
    
        ],
        "inactive": {
                "ipv4-address": [
    
                ],
                "ipv6-address": [
    
                ],
                "route": [
    
                ],
                "dns-server": [
    
                ],
                "dns-search": [
    
                ],
                "neighbors": [
    
                ]
        },
        "data": {
    
        }

}
```

所以看使用场景决定使用哪种方式，对于OpenWrt来说uci是非常方便且实用的，所以遇到一些问题时首先考虑uci方式是否可行。

## 3. 代码 ##

   uci方式就不说了，利用相关语言的uci库进行处理即可，实在不行调用shell执行uci命令处理也可以，算是比较简单，这里就不多说了，这里主要给一下实用ifstatus命令获取信息的代码，也是通过ifstatus命令执行后将结果进行处理，由于结果为json形式，所以处理起来是比较方便的（给一个json转go结构体的网站，很方便：https://mholt.github.io/json-to-go/）：

```
type ifStatus struct {
    Up          bool     `json:"up"`
    Pending     bool     `json:"pending"`
    Available   bool     `json:"available"`
    Autostart   bool     `json:"autostart"`
    Dynamic     bool     `json:"dynamic"`
    Uptime      int      `json:"uptime"`
    L3Device    string   `json:"l3_device"`
    Proto       string   `json:"proto"`
    Device      string   `json:"device"`
    Updated     []string `json:"updated"`
    Metric      int      `json:"metric"`
    DNSMetric   int      `json:"dns_metric"`
    Delegation  bool     `json:"delegation"`
    Ipv4Address []struct {
        Address string `json:"address"`
        Mask    int    `json:"mask"`
    } `json:"ipv4-address"`
    Ipv6Address          []interface{} `json:"ipv6-address"`
    Ipv6Prefix           []interface{} `json:"ipv6-prefix"`
    Ipv6PrefixAssignment []interface{} `json:"ipv6-prefix-assignment"`
    Route                []struct {
        Target  string `json:"target"`
        Mask    int    `json:"mask"`
        Nexthop string `json:"nexthop"`
        Source  string `json:"source"`
    } `json:"route"`
    DNSServer []string      `json:"dns-server"`
    DNSSearch []interface{} `json:"dns-search"`
    Neighbors []interface{} `json:"neighbors"`
    Inactive  struct {
        Ipv4Address []interface{} `json:"ipv4-address"`
        Ipv6Address []interface{} `json:"ipv6-address"`
        Route       []interface{} `json:"route"`
        DNSServer   []interface{} `json:"dns-server"`
        DNSSearch   []interface{} `json:"dns-search"`
        Neighbors   []interface{} `json:"neighbors"`
    } `json:"inactive"`
    Data struct {
        Leasetime int `json:"leasetime"`
    } `json:"data"`
}

type WanInfo struct {
    Ipv4Address string
    Gateway     string
    DNS1        string
    DNS2        string
}

func GetWanIpDNSGateway(wanName string) (*WanInfo, error) {
    cmd := exec.Command("ifstatus", wanName)
    output, err := cmd.CombinedOutput()
    if err != nil {
        logger.Error(err)
        return nil, err
    }
    logger.Debug(string(output))
    jsonRes := ifStatus{}
    err = json.Unmarshal(output, &jsonRes)
    if err != nil {
        logger.Error(err)
        return nil, err
    }
    wanInfo := &WanInfo{}
    wanInfo.Ipv4Address = jsonRes.Ipv4Address[0].Address
    wanInfo.Gateway = jsonRes.Route[0].Nexthop
    for i := 0; i < len(jsonRes.DNSServer); i++ {
        switch i {
        case 0:
            wanInfo.DNS1 = jsonRes.DNSServer[0]
        case 1:
            wanInfo.DNS2 = jsonRes.DNSServer[1]
        }
    }
    return wanInfo, nil
}
```
```
//一般就是uci中查找出来的network中interfaced的wan口名称
wanInfo, err := utils.GetWanIpDNSGateway(interfaceWan)
    if err != nil {
        logger.Error(err)
        return nil, err
    }
    res.Ipaddr = wanInfo.Ipv4Address
    res.Gateway = wanInfo.Gateway
    if wanInfo.DNS1 != "" {
        res.Dns1 = wanInfo.DNS1
    }
    if wanInfo.DNS2 != "" {
        res.Dns2 = wanInfo.DNS2
    }
```

————————————————

版权声明：本文为CSDN博主「xiaoyaoyou.xyz」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/weixin_39510813/article/details/120206770
