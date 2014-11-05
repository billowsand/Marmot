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

#### 快照回滚

```
rbd --pool rbd snap rollback --snap snapname foo
rbd snap rollback rbd/foo@snapname
```

> **注意：**回滚快照就是将当前镜像的数据用快照重新替换。回滚的执行时间随着镜像大小的增加而增加。克隆将会比回滚快照更花时间。


### 分层快照

Ceph支持创建许多copy-on-write(COW)的克隆快照。 分层快照支持Ceph快设备的客户端快速创建镜像。例如，你可能创建了一个虚拟机的镜像并写入了一些数据。然后，做一个快照，保护这个快照，按照需要创建许多copy-on-write的克隆。一个快照是只读的，所以克隆快照语义上的简化让克隆的速度很快。

![Ceph COW](http://ceph.com/docs/master/_images/ditaa-b4c8b30123e5581e44b87f1836b96a869c4898b6.png)

> **笔记**：Parent和Child意味着一个snapshot是parent，克隆镜像是child。


每一个child存储着一个与之相关的parent镜像，能够让克隆镜像打开parent快照并从中读数据。

一个COW的克隆快照的表现形式是一个Ceph的块设备。你可以读、写、克隆和调整镜像的大小。对克隆的快照并没有更多的限制，但是克隆镜像依赖于一个之前的快照，所以你必须在克隆镜像之前保护快照。下面的图表描述了这个过程。

> **注意** Ceph仅仅支持两种形式的镜像。1）通过rbd创建的镜像。2）不被kernel的rbd模式支持。你必须使用qemu/kvm或者librbd直接通过克隆的方式进行。

#### 制作分层快照

Ceph的分层快设备是一个简单的过程。你必须首先有一个镜像、创建一个这个镜像的快照，保护这个快照。一旦你做完了这三个步骤，你能够开始克隆快照。

![Ceph Layering](http://ceph.com/docs/master/_images/ditaa-bbd86247e30fd4ca83550176ab1bf5a39ab46c6d.png)

克隆镜像必须制定一个parent快照，包括这个快照所在的pool ID、image ID和snapshot ID。包含pool ID意味着可以从另外一个pool中克隆快照。

1. **镜像模版：** 一种通用的使用分层快照的方法是创建一个主镜像和一个快照提供克隆的模版。例如，一个用户可以创建一个Linux发行版的镜像，安装完成后生成一个快照。之后对快照进行保护后克隆快照。
2. **扩展模版：** 一种更为高级的方式制作模版。例如，一个用户可能克隆了一个镜像并且安装了其他软件（例如数据库等），之后对这个镜像做快照。之后这个镜像又变成了一个新的基本镜像。 
1. **模版池：** 一种使用分层块设备的方法是创建一个存储池包括主镜像，并进行各种快照扩展，用户可以以只读方式访问这个模版池。
2. **镜像迁移和恢复** 还有一种使用分层块设备的方法就是用于从不同的镜像池中迁移或者恢复数据

#### 分层快照

克隆镜像需要访问parent快照，如果用户无意中删除了parent快照，所有的克隆将会无法恢复，所以克隆之前必须重新保护快照。

```
rbd --pool rbd snap protect --image my-image --snap my-snapshot
rbd snap protect rbd/my-image@my-snapshot
```

> 被保护的快照不可以删除

#### 克隆快照

```
rbd clone rbd/my-image@my-snapshot rbd/new-image
```

> 你可以从一个存储池克隆快照到另一个存储池。例如，你可以修复一个只读的镜像，并且在一个存储池做一个快照模版。在另一个存储池中克隆。

#### 取消保护快照

```
rbd --pool rbd snap unprotect --image my-image --snap my-snapshot
rbd snap unprotect rbd/my-image@my-snapshot
```

#### 列出快照的儿子

```
rbd --pool rbd children --image my-image --snap my-snapshot
rbd children rbd/my-image@my-snapshot
```

#### 合并镜像
克隆镜像存在依赖关系。取消这种依赖关系叫做合并镜像。合并镜像花费的时间和镜像的大小成正比。 要删除一个快照模版，必须先合并子镜像。

```
rbd --pool rbd flatten --image my-image
rbd flatten rbd/my-image
```

> **注意：**合并的镜像会占据更多的存储空间。




