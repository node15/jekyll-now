---
layout: post
title: OpenWrt： 设置一个无线wifi，另加一个客用wifi
blog: true
categories: [云计算]
desc: "OpenWrt basic configuration"
permalink: :title/
---

## OpenWrt功用

1. 用做wifi

读OpenWrt文档知道有多种无线接入分类,dump ap,routed ap,routed clien,bridged ap...,见[wiki.howtos](https://wiki.openwrt.org/doc/howto/start),我想要的是一个能访问192.168.1.1网段的ap,大约相当于里面的dump ap,另外还需要一个供移动设备联网的ap,大约相当于里面的routed ap.

2. NFS file sharng server

书,文章,放在中心服务器,原来想用owncloud,后来发现网络接口是10/100M bps级别的,至少要1G bps,放弃owncloud on Raspberry Pi,换成OpenWrt NFS Server.

## OpenWRT Failsafe Mode

3年前刷的OpenWrt,中间把设置搞乱了.从恢复系统开始.Failsafe里面保存基本的软件,需要新装前检查System -> Software. Failsafe相当于回到刚刷机状态.

1. 进入Failsafe. 只留一根网线连接OpenWrt和电脑.开机等10秒见电源灯闪(缓慢闪约1秒1次),按Reset键(USB口边),电源等迅即快速闪动(1秒10次).

2. PC端,连接网线.

$ sudo telnet 192.168.1.1

$ mount_root ## this remounts your partitions from read-only to read/write mode

$ firstboot  ## This will reset your router after reboot

$ reboot -f ## And force reboot

3. Browser.192.168.1.1,先设置密码, 预先登录root,无密码.

4. 从PC ssh 192.168.1.1,从浏览器有LuCi图形界面可用,以下设置全部从命令行执行.

## PPPOE, Wifi, Routed AP

### /etc/config/network

```
config interface lan
        option ifname   eth0
        option type     bridge
        option proto    static
        option ipaddr   192.168.1.1
        option netmask  255.255.255.0

config interface 'wan'
        option ifname 'eth1'
        option proto 'pppoe'
        option username 'your-accout-from-isp'
        option password 'your-password-from-isp'

config 'interface' 'wlan'
        option 'ifname' 'wlan0'
        option 'type' 'bridge'
        option 'proto' 'static'
        option 'ipaddr' '192.168.2.1' //differ from lan ip
        option 'netmask' '255.255.255.0'
```

### /etc/config/dhcp

```
config dhcp lan
        option interface        lan
        option start    100
        option limit    150
        option leasetime        12h

config dhcp wlan
        option interface wlan
        option start 100
        option limit 50
        option leasetime 1h
```

### /etc/config/wireless
```
config wifi-iface
        option device 'radio0'
        option network 'lan'
        option mode 'ap'
        option ssid 'your-ssid'
        option encryption 'psk2'
        option key 'your-wifi-password'

config 'wifi-iface'
        option device 'radio0'
        option 'network' 'wlan'
        option 'mode' 'ap'
         option ssid 'your-guest-wifi-ssid'
        option encryption 'psk2'                                                                           
        option key 'your-guest-wifi-password'

config wifi-device  radio1
        option type     mac80211
        option channel  36
        option macaddr  c8:be:19:5f:d4:eb
        option hwmode   11na
        option htmode   HT20
        list ht_capab   SHORT-GI-40
        list ht_capab   TX-STBC
        list ht_capab   RX-STBC1
        list ht_capab   DSSS_CCK-40
        # REMOVE THIS LINE TO ENABLE WIFI:
        option disabled 1

config wifi-iface
        option device   radio1
        option network  lan
        option mode     ap
        option ssid     Openwrt
        option encryption none
        
```
### /etc/config/firewall

```
config zone
        option name wlan
        option network wlan
        option input ACCEPT
        option output ACCEPT
        option forward REJECT
                                        
config forwarding
        option src wlan
        option dest wan
```

### Reboot
为避免遗漏,从电源重启系统.否则命令有次序,容易出错.如:

```
ifup wifi
wifi
/etc/init.d/firewall restart
/etc/init.d/dnsmasq restart
```
