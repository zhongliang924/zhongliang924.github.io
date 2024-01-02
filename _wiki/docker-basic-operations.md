---
layout: wiki
title: docker基本操作
cate1: docker
---

## 修改docker默认存储位置

docker默认存储位于`/var/lib/docker`下面，可以通过以下命令查看我们的Docker容器具体位置

```
docker info | grep "Docker Root Dir"
```

停掉docker服务

```
systemctl stop docker
```

移动整个`/var/lib/docker`目录到空间富裕的路径

```
mv /var/lib/docker /ssd1/docker
```

更改docker配置文件，首先在`/etc/docker`目录下新建daemon.json文件

```
vim /etc/docker/daemon.json
```

输入

```
"data-root": "/ssd1/docker"
```

重启docker服务

```
systemctl restart docker
```

这样，我们的docker的默认存储位置就由`/var/lib/docker`更改到`/ssd1/docker`目录下了

## Docker镜像容器删除

- 删除名称为\<none>的镜像

```
docker rmi $(docker images -f "dangling=true" -q)
```

- 删除状态为\<Exited>的容器

```
docker rm $(docker ps -q -f status=exited)
```

- 删除名称包含"calico"的镜像

```
docker rmi $(docker images | grep calico | awk '{print $3}')
```

## 修改共享内存

docker中的进程要与宿主机使用共享内存通信，共享内存过小可能导致在执行某些程序时发生内存溢出的现象，默认的共享内存大小为64MB，下面介绍如何在启动的docker容器中修改共享内存的大小。

进入容器，输入df可以看到共享内存的大小：

![image-20230510133912899.png](/images/wiki/image-20230510133912899.png)

首先在主机上使用`docker ps -a`查看容器id：

![image-20230510134044004.png](/images/wiki/image-20230510134044004.png)

接下来关闭docker服务

```
service docker stop
```

进入该容器目录`cd /ssd1/docker/containers/容器id`，修改hostconfig.json文件，找到“ShmSize”字段并进行更改我们需要的共享内存大小

![image-20230510134334049.png](/images/wiki/image-20230510134334049.png)

最后重启docker服务

```
systemctl restart docker
```

## 添加端口映射

与修改共享内存类似，进入容器目录修改hostconfig.json文件，找到“PortBindings"字段，新增2001，2002，2003号端口，

```
"PortBindings":{
	"2001/tcp": [{"HostIp":"","HostPort":"2001"}],
	"2002/tcp": [{"HostIp":"","HostPort":"2002"}],
	"2003/tcp": [{"HostIp":"","HostPort":"2003"}]
}

```

![image-20230524104312750.png](/images/wiki/image-20230524104312750.png)

然后修改config.v2.json文件，找到”ExposedPorts“字段，新增2001，2002，2003号端口的主机映射：

```
"ExposedPorts":{
	"22/tcp": {},
	"2001/tcp": {},
	"2002/tcp": {},
	"2003/tcp": {}
}
```

![image-20230524104818596.png](/images/wiki/image-20230524104818596.png)

重启容器完成端口映射，使用命令可以查看端口映射

```
docker port lzl
```

![image-20230524105100650.png](/images/wiki/image-20230524105100650.png)

## Windows 修改 Docker 镜像存储位置

参考博客：[https://blog.csdn.net/u013948858/article/details/111464534](https://blog.csdn.net/u013948858/article/details/111464534)

将`docker-desktop-data`导出到文件中(备份image及相关文件)，使用如下命令

```
wsl --export docker-desktop-data "E:\\docker-desktop-data.tar"
```

从wsl`取消注册`docker-desktop-data，请注意`C:\Users\李仲亮\AppData\Local\Docker\wsl\data\ext4.vhdx`文件将被自动删除。

```
wsl --unregister docker-desktop-data
```

将导出的`docker-desktop-data`再导入回wsl，并设置我们想要的路径，**即新的镜像及各种docker使用的文件的挂载目录**，我这里设置到`E:\\docker\\wsl`

```
wsl --import docker-desktop-data "E:\\docker\\wsl" "E:\\docker-desktop-data.tar" --version 2
```

命令执行完毕，就能再目录下看到文件了，这时次`启动Docker Desktop`，可以正常工作了。

通过检查文件大小可验证有效性，更改完删除`docker-desktop-data.tar`。

## Docker容器挂载

目前我们固态硬盘容量不足了，需要大家把数据迁移到其它盘。需要使用**数据挂载**的方式将数据迁移到其它盘。

### 创建容器前挂载

通过 `docker run` 命令，一个示例如下：

```
docker run -it --gpus all --name ${user} -v /hdd0:/data -p 4000:22 nvidia/cuda:11.6.0-devel-ubuntu18.04 /bin/bash
```

使用 `-v` 标记在容器中设置一个挂载点 `/data` （就是容器中的一个目录），并将主机上的 `/hdd0` 目录（机械硬盘位置）中的内容关联到 `/data` 下，这样容器对 `/data` 目录的操作，还是在主机上对 `/hdd0` 的操作，都是完全实时同步的，因为这两个目录实际都是指向主机目录。

### 创建容器后挂载

一般我们不会新建一个容器，如果需要在已有的容器上修改，需要修改两个文件

进入到容器目录 `cd /ssd1/docker/containers`，查看到宿主机部署了很多容器

![image-20231022113720204](/images/posts/image-20231022113720204.png)

那么如何找到自己的容器？

使用 `docker ps -a`，第一列给出了容器 ID，找到自己的容器名对应的容器 ID 即可，比如我的是 cf44d4b38a28

![image-20231022113842355](/images/posts/image-20231022113842355.png)

进入到**自己的**容器目录（这里注意一下不要进错了），目录下有两个文件需要修改，一个是 `hostconfig.json`，另一个是 `config.v2.json`

![image-20231022114052011](/images/posts/image-20231022114052011.png)

> ！！！注意：修改前确保关闭 docker 容器
>
> ```
> systemctl stop docker
> ```

### hostconfig.json

首先修改 `hostconfig.json`，这个文件修改比较简单，直接用 `vim` 就可以修改，仅需要在第一个字段 `Binds` 下添加需要挂载的目录即可，比如把机械硬盘 `/hdd0` 挂载到容器内的 `/data` 目录，把第一块固态硬盘 `/ssd0` 挂载到容器内的 `/data1` 目录

![image-20231022114429010](/images/posts/image-20231022114429010.png)

### config.v2.json

这个文件内容比较多，并且需要修改的东西比较多，如果对 `vim` 操作不是很熟悉，建议拿出来修改，注意备份就行，首先

```
cp config.v2.json /ssd0/config.v2.json
chmod 777 config.v2.json
```

使用 vscode 打开 `config.v2.json`

vscode 设置软换行的方式参考：https://blog.csdn.net/xiang0731/article/details/123084126

![image-20231022113055848](/images/posts/image-20231022113055848.png)

找到 `MountPoints` 字段，仿造其格式进行修改，在我的容器中，出现 `/data` 的地方统统换成 `/data1`，表示容器内的目录，出现 `/hdd0` 的地方统统换成 `/ssd0`，表示需要挂载的目录，其它基本没有改动。

> 我这里为了便于观看插入了几个空格，最后需要把空格还原回去

![image-20231022115220043](/images/posts/image-20231022115220043.png)

保存退出。

对原来的 `config.v2.json` 备份并把新修改的添加进去：

```
mv config.v2.json config.v2.json.bak
cp /ssd0/config.v2.json ./config.v2.json
```

重新启动 docker

```
systemctl restart docker
```

第一块固态硬盘就挂载进我们的容器了，可以把数据集放在这个地方（所有人都能看到）

![](/images/posts/image-20231022115815860.png)

应该能看到这两个挂载信息。

![image-20231022115915934](/images/posts/image-20231022115915934.png)

## 