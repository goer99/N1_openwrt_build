# 一、 Github Actions 自动构建斐讯N1 OpenWrt固件脚本  
Automatically Build OpenWrt Firmware for PHICOMM N1 by Github Actions

**制作脚本已部署到Github Action，真正实现一栈式全自动构建，每月28日自动构建，无须自行制作，下载即可用**

**这里使用的是稳定版核心4.19.134，** ***如需要最新版核心，见下面的链接（不一定稳定、不一定可用，建议先看看该库的issue列表）：***

[![N1-OpenWrt-CI](https://github.com/tuanqing/mknop/workflows/N1-OpenWrt-CI/badge.svg?branch=master)](https://github.com/tuanqing/mknop/actions)  




# 二、自己手动编译&构建之方法

1. Linux环境，推荐使用 Ubuntu 18.04 LTS
2. 编译好待构建的 OpenWrt 固件，不会的自行科普 [Lean's OpenWrt source](https://github.com/coolsnowwolf/lede "Lean's OpenWrt source")  
   编译N1固件的配置如下：
   ``` 
   Target System (QEMU ARM Virtual Machine)  --->
   Subtarget (ARMv8 multiplatform)  --->
   Target Profile (Default)  --->
   ```

   **注意：  
   一键安装到 emmc 脚本已迁移至 openwrt package。使用方法如下，悉知！！**

   **用法**：  
   a. `git clone https://github.com/tuanqing/install-program package/install-program`  
   b. 执行 `make menuconfig` ，选中Utilities下的install-program
      ``` 
      Utilities  --->  
         <*> install-program
      ```
   c. 编译完成之后使用本源码制作镜像写入U盘启动，之后执行 `n1-install` 即可安装到emmc  
   d. 将固件上传到 `/tmp/upgrade`( xxx.img )，之后执行 `n1-update` 即可从该固件升级

3. 克隆仓库到本地  
   `git clone https://github.com/tuanqing/mknop` 
4. 将你编译好的固件拷贝到 openwrt 目录( 可复制多个 )
5. 使用 sudo 执行脚本  
   `sudo ./make` 
6. 按照提示操作，如，选择你要制作的固件、选择内核版本、设置 ROOTFS 分区大小等  
   如果你不了解这些设置项，请按回车保持默认，或者直接执行  
   `sudo ./make -d` 
7. 等待构建完成，默认输出文件夹为 out
8. 写盘启动，写盘工具务必使用 [Etcher](https://www.balena.io/etcher/)，使用其他写盘工具即便可以U盘启动，也会出现kernel panic而无法进入系统。

**注意**：  
  a. 待构建的固件格式只支持rootfs.tar[.gz]、 ext4-factory.img[.gz]、root.ext4[.gz] 6种，推荐使用rootfs.tar.gz格式  
  b. 默认不会清理out目录，请手动删除，或使用 `sudo ./make -c` 清理



## 特别说明

* 目录说明
``` 
   ├── armbian                               armbian 相关文件
   │   └── phicomm-n1                        设备文件夹
   │       ├── boot-common.tar.gz            公有启动文件
   │       ├── firmware.tar.gz               armbian 固件
   │       ├── kernel                        内核文件夹，在它下面添加你的自定义内核
   │       │   └── 5.4.60                    kernel 5.4.60-flippy-42+o @flippy
   │       └── root                          rootfs 文件夹，在它下面添加你的自定义文件
   ├── LICENSE                               license
   ├── make                                  构建脚本
   ├── openwrt                               固件文件夹(to build)
   ├── out                                   固件文件夹(build ok)
   ├── tmp                                   临时文件夹
   └── README.md                             readme, current file
```

* 使用参数
   * `-c, --clean` ，清理临时文件和输出目录
   * `-d, --default` ，使用默认配置来构建固件( openwrt下的第一个固件、构建所有内核、ROOTFS分区大小默认设为800m )
   * `--kernel` ，显示kernel文件夹下的所有内核
   * `-k=VERSION` ，设置内核版本，设置为 `all` 将会构架所有内核版本固件，设置为 `latest` 将构建最新内核版本固件
   * `-s, --size=SIZE` ，设置 ROOTFS 分区大小，不要小于 256M
   * `-h, --help` ，显示帮助信息
   * examples：  
      `sudo ./make -c` ，清理文件  
      `sudo ./make -d` ，使用默认配置  
      `sudo ./make -k 5.4.60` ，将内核版本设置为 5.4.60  
      `sudo ./make -k latest` ，使用最新内核  
      `sudo ./make -s 512` ，将 ROOTFS 分区大小设置为 512M  
      `sudo ./make -d -s 512` ，使用默认，并将分区大小设置为 512M  
      `sudo ./make -d -s 512 -k 5.4.60` ，使用默认，并将分区大小设置为 512M，内核版本设置为 5.4.60  
      `sudo ./make -e`，从 openwrt 目录中提取内核，仅支持 .img 格式和 xz 压缩的 .img 格式

* 自定义
   * 使用自定义内核  
     使用 `sudo ./make -e`，从 openwrt 目录中提取内核

   * 添加自定义文件  
      向 armbian/phicomm-n1/root 目录添加你的文件

      **注意**：添加的文件应保持与 ROOTFS 分区目录结构一致
      
      
* N1做旁路由使用，需在防火墙里添加如下规则。若无法访问国内网站，则需要在”网络“-”接口“-”LAN“-”物理设置“-”桥接接口“中取消桥接。
iptables -t nat -I POSTROUTING -o eth0 -j MASQUERADE
