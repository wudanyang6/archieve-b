---
title: "树莓派 ubuntu 配置网络"
date: 2021-04-28T10:18:04+08:00
draft: false
tags: ['ubuntu', '树莓派', '网络']
---

# 树莓派 `ubuntu` 配置网络

树莓派现在可以安装很多操作系统，其中 `ubuntu` 也提供了树莓派版本，因为对 `ubuntu` 更加熟悉，所以就往 `SD` 卡中烧录了 `ubuntu` 的 `20.04 lts 64位` 版本。[这里有可供选择的操作系统。](https://www.raspberrypi.org/software/operating-systems/)

简单提一嘴烧录操作系统到 `SD` 卡上面的方法，树莓派提供了一个[傻瓜式的软件](https://www.raspberrypi.org/software/)给大家使用，直接打开之后选择系统，就可以烧录了，烧录过程大概会有几分钟。

烧录完成之后，如果你按照树莓派官网的教程进行操作，肯定都行不通，因为树莓派的官方操作系统 `Raspberry Pi OS` 和 `Ubuntu For Raspberry` 上面的网络配置方式完全不一样。



## 安装了 ubuntu 的树莓派怎么配置网络？

打开你烧录好的 `SD` 卡

找到 `network-config` 文件

写入如下配置：

```yaml
wifis:
  wlan0:
  dhcp4: true
  optional: true
  access-points:
    <wifi network name>:
      password: "<wifi password>"
```

如果是企业网络：

```yaml
wifis:
  wlan0:
  dhcp4: true
  optional: true
  access-points:
        <wifi network name>:
          auth:
            key-management: eap
            method: peap
            identity: "wudanyang"
            password: ""
```

[配置文件的格式](https://netplan.io/examples/)

修改完之后，把 `SD` 卡从电脑上弹出，然后插到树莓派里。

不过，**重启之后，你可能还是无法连接到网络**。可以看下[这里](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#3-wifi-or-ethernet)

>   Note: During the first boot, your Raspberry Pi will try to connect to this network. It will fail the first time around. Simply reboot `sudo reboot` and it will work.

引用`ubuntu`官网的一段话，第一次会失败，直接重启，**第二次就能连上了**。



## 开启网络之后，如何登录树莓派？

安装好之后默认用户名和密码都是 `ubuntu` 

`ssh ubuntu@<raspberry ip>`



ip 地址的获取这里列举几种方法，不详细介绍：

1.  **自己的路由器**：登录后台查看分配的 ip
2.  **局域网扫描**（局域网机器不多的情况下）：
    1.  下载个局域网扫描工具，如果局域网机器不多，会看到一个制造商为树莓派的 ip
    2.  使用命令行 `arp -a ` 挨个试一下
3.  `ubuntu` 启动之后会使用 `avahi` 服务通过 [mdns](https://baike.baidu.com/item/mdns) 协议在局域网注册一个域名 `ubuntu.local`，所以你可以这样登录 `ssh ubuntu@ubuntu.local`




# 参考

[树莓派支持的操作系统](https://www.raspberrypi.org/software/operating-systems/)

[树莓派操作系统烧录软件](https://www.raspberrypi.org/software/)

[无屏幕和键盘配置树莓派WiFi和SSH](https://shumeipai.nxez.com/2017/09/13/raspberry-pi-network-configuration-before-boot.html)

[百度百科-mdns](https://baike.baidu.com/item/mdns)
