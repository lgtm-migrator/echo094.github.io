---
layout: post
title:  "Openwrt配置说明"
date:   2018-09-25 00:00:00 +0000
tags: linux router openwrt
---

修改时间： 2020-06-09

**最近学校愈发丧心病狂，现在ipv6不仅只能获取到/128的单播地址，还需要认证，之前桥接的路子也不管用了，只能用NAT6。**

路由器版本：

Model: Newifi-D2

Architecture: MediaTek MT7621 ver:1 eco:3

Firmware Version: OpenWrt 19.07.3 r11063-85e04e9f46



## 配置HTTPS访问

参考：[openwrt开启ssl 使用let‘s encrypt的证书](https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=310924&page=1)

1. 前往` www.sslforfree.com`申请通配符证书
2. 安装下面两个包：`luci-ssl`，`luci-app-uhttpd`
3. 将`certificate.crt`以及`private.key`上传到路由器
4. 修改`uhttpd`配置文件`/etc/config/uhttpd.conf`，修改证书文件的路径以及监听端口
5. 重启`uhttpd`服务
6. 在防火墙中添加端口转发并重启

注意事项：

* 建议关闭`HTTP到HTTPS的重定向`，否则内网使用IP访问会有警告
* `let‘s encrypt`证书有效期为3个月，需要定期更新



## 配置 ipv6

根据这个帖子：[广州移动家宽已支持 dhcpv6-pd#52](https://www.v2ex.com/t/483739#r_6117230)，找到了以下两个帖子：[OpenWrt/LEDE 内网转发 IPv6](https://i-meto.com/lede-ipv6/)，[OpenWRT/LEDE 开启免 NAT 全局 IPv6](https://blog.rabit.pw/2017/lede-ipv6/)。其中有三种解决方案。



### Relay

如果上级网关支持，可照着这个资料修改：[OpenWrt 路由器如何让 lan 口主机获得 ipv6 网络访问？](https://www.zhihu.com/question/29667477/answer/93634257)。

* 在LuCI界面删除**接口/全局网络选项**配置下的IPv6 ULA 前缀

* 配置/etc/config/dhcp

```wiki
config dhcp 'lan'
	option interface 'lan'                   
	option start '100'                       
	option limit '150'                       
	option leasetime '12h' 
	option ra 'relay'                                            
	option ndp 'relay'
	option dhcpv6 'relay'
	
config dhcp 'wan'                                
	option interface 'wan'                   
	option ignore '1'

config dhcp 'wan6'
	option interface 'wan'
	option ra 'relay'                        
	option ndp 'relay'  
	option dhcpv6 'relay'  
	option master '1'
```

关于中继代理，可以参考下述文章：[DHCPv6中的中继代理在IPv6路由器中的实现](http://dx.chinadoi.cn/10.3969/j.issn.1007-130X.2009.06.003)



### NAT6

官方文档：[OpenWrt Project: NAT6: IPv6 Masquerading Router](https://openwrt.org/docs/guide-user/network/ipv6/ipv6.nat6)

参考这个文档：[OpenWRT 路由器作为 IPv6 网关的配置](https://github.com/tuna/ipv6.tsinghua.edu.cn/blob/master/openwrt.md)。

支持`masq6`的防火墙版本：[nomagick/firewall3: A fork from openwrt/firewall3. With NAT6 support](https://github.com/nomagick/firewall3)

首先，要让路由器不获取ipv6前缀：

```wiki
# /etc/config/network
config interface 'wan6'
	option reqprefix 'no'
```

然后，要让lan的设备获取到ipv6地址。

1. 安装`kmod-ipt-nat6`另一个包`ip6tables`已经自带了。

2. 设置 IPv6 ULA 前缀，这个也是自带的，注意前缀的第一个数字（16进制）不要是F。

3. 配置DHCPv6

    ```wiki
    # /etc/config/dhcp
    config dhcp 'lan'
        option dhcpv6 'server'
        option ra 'server'
        option ra_management '1'
        option ra_default '1'
    ```

4. 修改防火墙规则，防火墙说明文档：[OpenWrt Project: Netfilter Management](https://openwrt.org/docs/guide-user/firewall/netfilter_iptables/netfilter_management)

   ```shell
   # /etc/config/firewall
   config zone
       option name 'wan'
       # add
       option masq6 '1'
   
   config rule
       option name 'Allow-ICMPv6-Forward'
       # add
       option enabled '0'
   ```

   添加文件并增加执行权限[akatrevorjay/openwrt-masq6](https://github.com/akatrevorjay/openwrt-masq6)：

   ```shell
   # /etc/firewall.nat6
   # Masquerading nat6 firewall script
   #
   # Then you can configure in /etc/config/firewall per zone, ala where you have:
   #   option masq 1
   # Just drop this in beneath it:
   #   option masq6 1
   # For IPv6 privacy (temporary addresses used for outgoing), also add:
   #   option masq6_privacy 1
   #
   # Hope it's useful!
   #
   # https://github.com/akatrevorjay/openwrt-masq6
   # ~ trevorj <github@trevor.joynson.io>
    
   set -eo pipefail
    
   . /lib/functions.sh
   . /lib/functions/network.sh
   . /usr/share/libubox/jshn.sh
    
   log() {
       logger -t nat6 -s "$@"
   }
    
   get_ula_prefix() {
       uci get network.globals.ula_prefix
   }
    
   validate_ula_prefix() {
       local ula_prefix="$1"
       if [ $(echo "$ula_prefix" | grep -c -E "^([0-9a-fA-F]{4}):([0-9a-fA-F]{0,4}):") -ne 1 ] ; then
           log "Fatal error: IPv6 ULA ula_prefix=\"$ula_prefix\" seems invalid. Please verify that a ula_prefix is set and valid."
           return 1
       fi
   }
    
   ip6t() {
       ip6tables "$@"
   }
    
   ip6t_add() {
       if ! ip6t -C "$@" &>/dev/null; then
           ip6t -I "$@"
       fi
   }
    
   nat6_init() {
       iptables-save -t nat \
       | sed -e "/\s[DS]NAT\s/d;/\sMASQUERADE$/d" \
       | ip6tables-restore -T nat
   }
    
   masq6_network() {
       # $config contains the ID of the current section
       local network_name="$1"
    
       local device
       network_get_device device "$network_name" || return 0
    
       local done_net_dev
       for done_net_dev in $DONE_NETWORK_DEVICES; do
           if [ "$done_net_dev" = "$device" ]; then
               log "Already configured device=\"$device\", so leaving as is."
               return 0
           fi
       done
    
       log "Found device=\"$device\" for network_name=\"$network_name\"."
    
       if [ $zone_masq6_privacy -eq 1 ]; then
           log "Enabling IPv6 temporary addresses for device=\"$device\"."
    
           log "Accepting router advertisements on $device even if forwarding is enabled (required for temporary addresses)"
           echo 2 > "/proc/sys/net/ipv6/conf/$device/accept_ra" \
             || log "Error: Failed to change router advertisements accept policy on $device (required for temporary addresses)"
    
           log "Using temporary addresses for outgoing connections on interface $device"
           echo 2 > "/proc/sys/net/ipv6/conf/$device/use_tempaddr" \
             || log "Error: Failed to enable temporary addresses for outgoing connections on interface $device"
       fi
    
       append DONE_NETWORK_DEVICES "$device"
   }
    
   handle_zone() {
       # $config contains the ID of the current section
       local config="$1"
    
       local zone_name
       config_get zone_name "$config" name
    
       # Enable masquerading via NAT6
       local zone_masq6
       config_get_bool zone_masq6 "$config" masq6 0
    
       log "Firewall config=\"$config\" zone=\"$zone_name\" zone_masq6=\"$zone_masq6\"."
    
       if [ $zone_masq6 -eq 0 ]; then
           return 0
       fi
    
       # IPv6 privacy extensions: Use temporary addrs for outgoing connections?
       local zone_masq6_privacy
       config_get_bool zone_masq6_privacy "$config" masq6_privacy 1
    
       log "Found firewall zone_name=\"$zone_name\" with zone_masq6=\"$zone_masq6\" zone_masq6_privacy=\"$zone_masq6_privacy\"."
    
       log "Setting up masquerading nat6 for zone_name=\"$zone_name\" with zone_masq6_privacy=\"$zone_masq6_privacy\""
    
       local ula_prefix=$(get_ula_prefix)
       validate_ula_prefix "$ula_prefix" || return 1
    
       local postrouting_chain="zone_${zone_name}_postrouting"
       log "Ensuring ip6tables chain=\"$postrouting_chain\" contains our MASQUERADE."
       ip6t_add "$postrouting_chain" -t nat \
           -m comment --comment "!fw3" -j MASQUERADE
    
       local input_chain="zone_${zone_name}_input"
       log "Ensuring ip6tables chain=\"$input_chain\" contains our permissive DNAT rule."
       ip6t_add "$input_chain" -t filter -m conntrack --ctstate DNAT \
           -m comment --comment "!fw3: Accept port forwards" -j ACCEPT
    
       local forward_chain="zone_${zone_name}_forward"
       log "Ensuring ip6tables chain=\"$forward_chain\" contains our permissive DNAT rule."
       ip6t_add "$forward_chain" -t filter -m conntrack --ctstate DNAT \
           -m comment --comment "!fw3: Accept port forwards" -j ACCEPT
    
       local DONE_NETWORK_DEVICES=""
       config_list_foreach "$config" network masq6_network
    
       log "Done setting up nat6 for zone=\"$zone_name\" on devices: $DONE_NETWORK_DEVICES"
   }
    
   main() {
       nat6_init
       config_load firewall
       config_foreach handle_zone zone
   }
    
   main "$@"
   ```
   
   在防火墙中添加该文件
   
   ```shell
   # /etc/config/firewall
   config include 'nat6'
   	option path '/etc/firewall.nat6'
   	option reload '1'
   ```



### Passthrough

关于这个方法，网上很多文档都过时了。和NAT相比，配置起来很简单。

1. 安装包`ebtables`和`kmod-ebtables-ipv6`
2. 设置ipv4转发：

```shell
ebtables -t broute -A BROUTING -p ! ipv6 -j DROP -i eth0.2
```

3. 内外网桥接

```shell
brctl addif br-lan eth0.2
```

4. 杀死`odhcpd`并禁止自动启动
5. 在LAN接口的 `IPv6 Settings` 选项卡中勾上 `Always announce default router`
6. 可将上述两条指令添加到启动脚本`/etc/rc.local`中。

但是这样的话路由器就没有ipv6地址了，对此，以下文章提供了一个方案：[桥接校园网IPv6，并让路由器自身获取IPv6地址](https://zhuanlan.zhihu.com/p/50356712)



## 功能模块

### Firewall

防火墙的额外配置

#### 安装ipset

使用`opkg`安装即可，一般用不到，但可以去除一个`Warning`。

```shell
opkg install ipset
```



#### NAT6端口转发

毕竟是一个很小众的使用方式，只能通过命令配置。

参照NAT6的官方文档，在正确配置NAT6后，ipv6的端口转发会自动添加。

但目前`masq6`不管用，因此只能手动添加。

将防火墙中`wan`的`Input`以及`Output`设置为`accept`，否则外网无法访问路由器。

将`/lib/functions/network.sh`添加执行权限。

创建脚本文件：

```sh
#!/bin/sh
# dnat6 integration for firewall3

. /lib/functions/network.sh

network_find_wan6 wan_iface6
network_get_ipaddr6 wan_ip6 "$wan_iface6"
if [ -z "$wan_ip6" ]; then
	logger -t IPv6 "NAT6 wan iface: $wan_iface6 ip not found"
	echo "NAT6 wan iface: $wan_iface6 ip not found"
	exit 0
fi
logger -t IPv6 "NAT6 wan iface: $wan_iface6 ip: $wan_ip6"
echo "NAT6 wan iface: $wan_iface6 ip: $wan_ip6"

network_get_ipaddr6 lan_ip6 "lan"
if [ -z "$lan_ip6" ]; then
	logger -t IPv6 "NAT6 lan iface: lan ip not found"
	echo "NAT6 lan iface: lan ip not found"
	exit 0
fi
logger -t IPv6 "NAT6 lan iface: lan ip: $lan_ip6"
echo "NAT6 lan iface: lan ip: $lan_ip6"

network_get_subnet6 sub_ip6 "lan"
if [ -z "$sub_ip6" ]; then
	logger -t IPv6 "NAT6 iface: lan subnet not found"
	echo "NAT6 iface: lan subnet not found"
	exit 0
fi
logger -t IPv6 "NAT6 iface: lan subnet: $sub_ip6"
echo "NAT6 iface: lan subnet: $sub_ip6"

# exit 0

IP6TABLES=/usr/sbin/ip6tables

nat6_client_rule() {
	local src_dport="$1"
	local dest_ip="$2"
	local dest_port="$3"

	echo "NAT6 DNAT CLIENT $src_dport $dest_ip $dest_port"
	# zone_lan_postrouting
	$IP6TABLES -t nat -A POSTROUTING -o br-lan -p tcp -s "$sub_ip6" -d "$dest_ip" --dport "$dest_port" -j SNAT --to-source "[$lan_ip6]"
	# zone_lan_prerouting
	$IP6TABLES -t nat -A PREROUTING -i br-lan -p tcp -s "$sub_ip6" -d "$wan_ip6" --dport "$src_dport" -j DNAT --to-destination "[$dest_ip]":"$dest_port"
	# zone_wan_prerouting
	$IP6TABLES -t nat -A PREROUTING -i eth0.2 -p tcp --dport "$src_dport" -j DNAT --to-destination "[$dest_ip]":"$dest_port"
}

nat6_gate_rule() {
	local src_dport="$1"
	local dest_ip="$2"
	local dest_port="$3"

	echo "NAT6 DNAT GATE $src_dport $dest_ip $dest_port"
	# zone_lan_prerouting
	$IP6TABLES -t nat -A PREROUTING -i br-lan -p tcp -s "$sub_ip6" -d "$wan_ip6" --dport "$src_dport" -j DNAT --to-destination "[$dest_ip]":"$dest_port"
	# zone_wan_prerouting
	$IP6TABLES -t nat -A PREROUTING -i eth0.2 -p tcp --dport "$src_dport" -j DNAT --to-destination "[$dest_ip]":"$dest_port"
}

# Accept port forwards
$IP6TABLES -t filter -A zone_lan_forward -m conntrack --ctstate DNAT -j ACCEPT
$IP6TABLES -t filter -A zone_wan_forward -m conntrack --ctstate DNAT -j ACCEPT
# Accept port redirections
$IP6TABLES -t filter -A zone_lan_input -m conntrack --ctstate DNAT -j ACCEPT
$IP6TABLES -t filter -A zone_wan_input -m conntrack --ctstate DNAT -j ACCEPT

# Add redirect rules
nat6_client_rule "公网端口" "内网IP" "内网端口"

```

在防火墙配置文件中包含该脚本文件：

```
config include
	option type 'script'
	option path '/etc/config/firewall-dnat6'
	option family 'any'
	option reload '1'
```



### Openvpn

新版的系统中设置方法又简化了。

OpenWrt官网的文档：[点我](https://openwrt.org/docs/guide-user/services/vpn/openvpn/start)



#### 路由器配置过程

**千万不要添加端口转发**

* 安装 `openvpn-openssl `以及界面` luci-i18n-openvpn-zh-cn`

* 配置`/etc/config/openvpn`，现在可直接引用外部文件：

    ```wiki
    config openvpn 'custom_config'
        # 启用配置文件
        option enabled '1'
        # 使用的配置文件
        option config '/etc/openvpn/server.ovpn'
    ```
    
    配置文件例子，注意`redirect-gateway`不需要`bypass-dhcp`属性：
    
    ```wiki
    # 实现子网掩码24的子网地址扩展
    topology subnet
    # 配置为UDP协议 如果是ipv6则使用udp6
    proto udp
    # 适配器编号 如果写tun默认从0开始编号
    dev tun0
    # 服务器监听端口
    port 1194
    # 服务器地址
    server 10.8.1.0 255.255.255.0
    # 加密选项
    ncp-ciphers AES-128-GCM:AES-256-GCM:AES-128-CBC:AES-256-CBC
    cipher AES-128-CBC
    # 允许客户端之间互相访问
    client-to-client
    # 心跳包
    keepalive 15 60
    # 超时后保留key
    persist-key
    # 超时后保留tun适配器
    persist-tun
    # 一个client证书可以同时使用多次
    duplicate-cn
    # 日志文件以及记录等级
    verb 3
    log '/etc/openvpn/openvpn.log'
    # 推送内容
    # 使客户端所有的IP请求都通过该服务器
    push "redirect-gateway def1"
    # 推送内网路由
    push 'route 192.168.1.0 255.255.255.0'
    # 推送DNS服务器
    push 'dhcp-option DNS 192.168.1.1'
    # 推送其它配置项
    push 'persist-tun'
    push 'persist-key'
    # 证书等
    <dh></dh>
    <ca></ca>
    <cert></cert>
    <key></key>
    ```
    
* 在 网络/接口 界面中添加新接口，选择openvpn的设备，不配置协议

    ```wiki
    config interface 'OpenVPN_0'
        option proto 'none'
        # 上面服务器配置中的接口名称
        option ifname 'tun0'
    ```

* 配置/etc/config/firewall

    ```wiki
    # 通信规则
    # 允许数据包发送给OpenVPN服务器的监听端口
    config rule
        option name 'Allow-OpenVPN'
        # OpenVPN中配置的协议
        option proto 'udp'
        option src 'wan'
        # OpenVPN中的监听端口
        option dest_port '1294'
        option target 'ACCEPT'

    # 添加在lan区域中添加tun0所在的接口
    config zone
        option name 'lan'
        # add context
        list network 'OpenVPN_0'
    ```



#### 客户端配置文件

这是电脑客户端的配置

```wiki
client
# 通信协议 ivp6使用UDP6
proto udp
# 适配器类型
dev tun
# 服务器地址以及端口 地址可以使用域名
remote [ip] 1194
# 允许服务器ip变更
float
# 加密选项
ncp-ciphers AES-128-GCM:AES-256-GCM:AES-128-CBC:AES-256-CBC
cipher AES-128-CBC
# 心跳包
keepalive 15 60
# 采用服务器校验方式
remote-cert-tls server
# 服务器为域名时始终重新解析IP
resolv-retry infinite
# 本机不绑定监听数据的端口
nobind
# 证书
<ca></ca>
<cert></cert>
<key></key>
```



### vlmcsd

升级了Office2019，学校的KMS还没更新。虽然自己有365，但是不够用。找了一下，有人移植了KMS服务到OpenWrt上，就拿来试试了。

项目地址：[cokebar/openwrt-vlmcsd](https://github.com/cokebar/openwrt-vlmcsd)

编译好的文件在gh-pages分支。Newifi-D2的架构是mipsel_24kc。

luci的安装包是通用的：[cokebar/luci-app-vlmcsd](https://github.com/cokebar/luci-app-vlmcsd)。

用winscp上传到tmp文件夹安装即可。

关于自动激活：

原理很简单， DHCP服务器指定的DNS服务器上有SRV记录指向vlmcsd 。 

Windows下的测试方法如下：

```bash
nslookup -type=srv _vlmcs._tcp.lan
```



### 其它模块

`upnp`:

```shell
opkg install luci-i18n-upnp-en
```

`wol`:

```shell
opkg install luci-i18n-wol-en
```

