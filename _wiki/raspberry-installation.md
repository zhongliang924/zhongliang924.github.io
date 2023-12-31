---
layout: wiki
title: 树莓派系统安装
cate1: 树莓派
---

将 SD 卡使用读卡器插入电脑进行操作系统的安装，进入安装界面 https://www.raspberrypi.com/software/，下载安装程序 `Download for Windows`：

![image-20230710191849564](/images/wiki/image-20230710191849564.png)

下载后得到 `imager_1.7.5.exe` 程序，打开该 exe 程序选择需要安装的操作系统和存储卡，这里操作系统选择`RASPBERRY PI OS(64-BIT)`，点击烧录：

![image-20230710192113394](/images/wiki/image-20230710192113394.png)

准备写入，整个写入时间比较长：

![image-20230710194334455](/images/wiki/image-20230710194334455.png)

插上 SD 卡之后就可以启动树莓派的系统了

首先更新：

```
sudo apt-get update
```

1、开启树莓派的 SSH （重启失效）

```
sudo service ssh start
```

2、开启树莓派的 SSH （重启后不失效）

```
sudo raspi-conifg
```

![image-20230710210537439](/images/wiki/image-20230710210537439.png)

选择 `Interface Options` 选项，回车，在里面选择 SSH：

弹出对话框：`Would you like the SSH server to be enabled`，选择 `Yes`。

选择完毕后关闭配置界面，重启之后 SSH 就能连接上了

连接 SSH 的相关信息：

主机地址：10.0.0.129

用户名： yq2

密码：123

![image-20230710211136992](/images/wiki/image-20230710211136992.png)

顺利通过 SSH 连接到树莓派。