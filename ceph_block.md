# Ceph的Block分析
一个块是一个连续的字节序列（例如一个512字节的连续数据是一个块）。基于块的存储接口通常是旋转介质，例如磁盘、光盘、软盘等。块设备接口的普及使得可以用虚拟的块设备成为和大容量数据存储系统交互的接口，如Ceph这样的系统。

![Ceph Block Architecture](http://ceph.com/docs/master/_images/ditaa-dc9f80d771b55f2daa5cbbfdb2dd0d3e6dfc17c0.png)


> 注意：内核模块可以使用Linux页缓存。 对于基于librbd的应用，Ceph支持RBD缓存。


Ceph的块设备可以在为Linux的内核，KVM虚拟机，以及像OpenStack和CloudStack这种使用libvirt的云计算平台提供高性能和无限可扩展的的存储。Ceph支持在集群中同时运行Ceph的RADOS网关，Ceph的文件系统和Ceph的块存储。


## 块设备命令

rbd的命令能够创建、列出、内省、和删除块设备镜像。你一可以使用其来克隆镜像，创建快照，回滚快照，查看快照等。跟详细的信息可以**参阅管理RADOS块设备镜像**

### 创建一个镜像
使用镜像之前，必须在Ceph的存储集群中创建镜像。执行下面的命令：
```
rbd create {镜像名称} --size {镜像大小} --pool {存储池名称}
```

例如，在名为**swinmmingpool**中创建一个1GB的名字为**foo**的镜像：
```
rbd create foo --size 1024
rbd create bar --size 1024 --pool swimmingpool
```
> 你必须首先创建一个存储池（pool）

### 列出镜像

```
rbd ls
rbd ls {poolname}
```

### 获取镜像信息

```
rbd --image {image-name} info
rbd --image {image-name} -p {pool-name} info
```

### 改变镜像大小

```
rbd resize --image foo --size 2048
```

### 删除镜像
```
rbd rm {image-name}
rbd rm {image-name} -p {pool-name}
```

## 快照
快照是一个镜像在某一个特定时间点的只读拷贝。其中Ceph块设备的高级功能是能够创建快照保留镜像的历史状态。Ceph支持快照分层，允许快速克隆镜像。 Ceph还支持使用RDB和如KVM，libvirt等创建快照。

> 当做快照时需要停止I/O。如果镜像包含文件系统，文件系统必须在做快照前保持一致性

![Ceph snapshot](http://ceph.com/docs/master/_images/ditaa-75fdb48a3db2bad6ef749fdae1282f4ae2dd1f7c.png)

### 基本的快照

#### 创建快照

```
rbd --pool rbd snap create --snap snapname foo
rbd snap create rbd/foo@snapname
```

