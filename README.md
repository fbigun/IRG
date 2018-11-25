# 使用grub2引导linux启动

## 概述

本项目名称由 `Install RancherOS From grub2` 简写而成。

## 创建新的启动菜单

从已有的系统启动并进入系统创建启动菜单

```bash
mkdir -p /boot/newinstall/ && cd /boot/newinstall/
wget https://releases.rancher.com/os/latest/initrd
wget https://releases.rancher.com/os/latest/vmlinuz
```

RancherOS 当前可用版本（截至2018年11月25日）

```
rancher/os:v1.4.2 remote latest
rancher/os:v1.4.1 remote available
rancher/os:v1.4.0 remote available
rancher/os:v1.3.0 remote available
rancher/os:v1.2.0 remote available
rancher/os:v1.1.4 remote available
rancher/os:v1.1.3 remote available
rancher/os:v1.1.2 remote available
rancher/os:v1.1.1 remote available
rancher/os:v1.1.0 remote available
rancher/os:v1.0.5 remote available
rancher/os:v1.0.4 remote available
rancher/os:v1.0.3 remote available
rancher/os:v1.0.2 remote available
rancher/os:v1.0.1 remote available
rancher/os:v1.0.0 remote available
rancher/os:v0.9.2 remote available
rancher/os:v0.8.1 remote available
rancher/os:v0.7.1 remote available
rancher/os:v0.6.1 remote available
rancher/os:v0.5.0 remote available
rancher/os:v0.4.5 remote available
```

低版本的RancherOS 引导对系统内存要求更低

> 不高于1G内存的使用`RancherOS`版本低于`v1.3.0`理论上可引导成功，已测试过`v1.2.0`可以启动(`docker engine`<=`17.12.1`)，或者通过降低`docker engine`版本重新打包`RancherOS`最新版本降低对内存使用要求。

> ```
wget https://releases.rancher.com/os/v1.2.0/initrd
wget https://releases.rancher.com/os/v1.2.0/vmlinuz
> ```


## 延长系统启动引导菜单等待时间

更改等待时间60s
```
sed -e 's/GRUB_TIMEOUT=5/GRUB_TIMEOUT=60/' -i /etc/default/grub
update-grub
```

## 添加启动菜单项目，**注意**内核参数`rancher.network.interfaces.eth*.dhcp=true`对应网卡

```
cat >> /boot/grub/grub.cfg << EOF
menuentry 'RancherOS Stable New Install' {
insmod part_msdos
insmod ext2
set root='(hd0,msdos1)'
linux /boot/newinstall/vmlinuz printk.devkmsg=on rancher.debug=true rancher.state.dev=LABEL=RANCHER_STATE rancher.state.wait console=tty0 rancher.state.mdadm_scan console=ttyS1,115200n8 rancher.autologin=ttyS1 rancher.network.interfaces.eth*.dhcp=true rancher.cloud_init.datasources=[url:https://github.com/fbigun/IRG/raw/master/cloud-config-boot.yml]
initrd /boot/newinstall/initrd
}
EOF
```

## 重启并选择新添加的启动项目

启动到RancherOS后进行硬盘安装RancherOS系统，文件`cloud-config-custom.yml`为自定义需要的配置内容，[参考](https://rancher.com/docs/os/v1.x/en/installation/configuration/)

```
sudo ros install -c https://github.com/fbigun/IRG/raw/master/cloud-config-custom.yml -d /dev/vda -i registry.docker-cn.com/rancher/os:latest
```


### Docker 国内镜像

* Tencent云内网
  mirror.ccs.tencentyun.com



> **参考**：
> * [引导 Linux 使用 LILO 或GRUB](https://www.debian.org/releases/stable/amd64/ch05s01.html.zh-cn#boot-initrd)
> * [RancherOS github链接](https://github.com/rancher/os)
> * [RancherOS 官方文档](https://rancher.com/docs/os/v1.x/en/)
