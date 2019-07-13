本文作者: **wuXing**
QQ: **1226032602**
E-mail: **1226032602@qq.com**

# glusterfs
https://www.gluster.org/
https://docs.gluster.org/en/latest/Install-Guide/Setup_virt/

# glusterfs概述
## 1. 什么是Gluster
Gluster是一个横向扩展的分布式文件系统，可将来自多个服务器的磁盘存储资源整合到一个全局名称空间中，可以根据存储消耗需求快速调配额外的存储。它将自动故障转移作为主要功能

当您修复发生故障的服务器并使其恢复联机状态时，除了等待外，您无需执行任何操作即可恢复数据。与此同时，您的数据的最新副本继续从仍在运行的节点获取。

Gluster数据可以从几乎任何地方访问，可以使用传统的NFS，Windows客户端的SMB / CIFS或我们自己的本地GlusterFS（客户机上需要一些附加软件包）

## 2. 企业应用场景
媒体数据：文档、图片、音频、视频
共享存储：云储存、虚拟化存储、HPC（高性能计算）
大数据：日志文件、RFID（射频识别）数据

## 3.  优点
缩放到几PB
处理数千个客户
POSIX兼容
可以使用任何支持扩展属性的ondisk文件系统
使用NFS和SMB等行业标准协议访问
提供复制，配额，地理复制，快照和bitrot检测
允许优化不同的工作量
开源

## 4.  缺点
不适用于存储大量小文件的场景，因为GlusterFS的设计之初就是用于存储大数据的，对小文件的优化不是很好，推荐保存单个文件至少1MB以上的环境，如果是大量小文件的场景建议使用FastDFS、MFS等

#  gluster安装配置

https://docs.gluster.org/en/latest/Quick-Start-Guide/Quickstart/
https://wiki.centos.org/SpecialInterestGroup/Storage/gluster-Quickstart

##  环境准备
两台名为server1和server2的服务器上的CentOS 7
两个虚拟磁盘，一个用于OS安装（sda），另一个用于服务GlusterFS存储（sdb）
##  格式化磁盘
CentOS6需要安装
```
yum install xfsprogs
```
在两个节点上都执行
```
mkfs.xfs -i size=512 /dev/sdb
mkdir -p /bricks/brick1
```
##  /etc/fstab
```
[root@server1 ~]# tail -1 /etc/fstab
/dev/sdb /bricks/brick1       xfs   defaults        0 0
```
##  挂载
```
mount -a
```
##  安装GlusterFS并启动
三台服务器上都安装
安装gluster源
```
yum install centos-release-gluster
```
安装glusterfs
```
yum install glusterfs-server
```
启动
```
systemctl start glusterd.service
```
##  hosts解析
三台都做
/etc/hosts
```
10.0.0.201 server1
10.0.0.202 server2
```
```
172.16.1.51     db01
172.16.1.31     nfs01
172.16.1.41     backup
```

##  添加服务器
nfs01:
```
gluster peer probe backup
gluster peer probe db01
```
backup:
```
gluster peer probe nfs01
gluster peer probe db01
```
db01:
```
gluster peer probe nfs01
gluster peer probe backup
```
查看对等状态
```
gluster peer status
```
删除服务器
```
gluster peer detach <server>
```
查看服务器列表
```
gluster pool list
```
gluster UUID
```
[root@server1 ~]# cat /var/lib/glusterd/glusterd.info
UUID=9c8f138a-d502-4b69-9b84-61bd81286ea6
operating-version=31202
```
##  建立GlusterFS卷
https://docs.gluster.org/en/latest/CLI-Reference/cli-main/

三台机器都执行
```
mkdir /bricks/brick1/gv0
mkdir /bricks/brick1/gv1
```
任意一台机器执行
创建复制卷
```
gluster volume create gv0 replica 2 nfs01:/bricks/brick1/gv0 db01:/bricks/brick1/gv0
gluster volume create gv1 replica 3 nfs01:/bricks/brick1/gv1 db01:/bricks/brick1/gv1 backup:/bricks/brick1/gv1
gluster volume create gv1 replica 3 arbiter 1 nfs01:/bricks/brick1/gv1 db01:/bricks/brick1/gv1 backup:/bricks/brick1/gv1
```
启动卷
```
gluster volume start gv0
gluster volume start gv1
```
查看信息
```
gluster volume info
```
日志
```
tail -f /var/log/glusterfs/glusterd.log
```
https://docs.gluster.org/en/latest/Administrator%20Guide/
##  glusterfs客户端
安装gluster源
```
yum install centos-release-gluster
```
安装
```
yum install glusterfs glusterfs-fuse glusterfs-rdma
```
hosts解析
/etc/hosts
```
10.0.0.201     server1
10.0.0.202     server2
```
```
172.16.1.51     db01
172.16.1.31     nfs01
172.16.1.41     backup
```
挂载
```
mount -t glusterfs server1:/gv0 /mnt/

mount -t glusterfs nfs01:/gv1 /mnt/
mount -t glusterfs backup:/gv1 /mnt/
```


# gluster Volume Commands
```
volume create volname [options] bricks    创建卷

volume start volname [force]            启动卷

volume stop volname                  停止卷

volume info [volname]                 查看卷信息

volumes status[volname]               查看卷状态

volume list                          查看卷列表

volume add-brick brick-1 ... brick-n      扩容

volume remove-brick brick-1 ... brick-n \<start|stop|status|commit|force>     缩减

volume delete volname                删除卷
```
#  glusterfs volume 模式
##  默认模式
默认模式，既DHT, 也叫 分布卷: 将文件已hash算法随机分布到 一台服务器节点中存储
```
gluster volume create test-volume server1:/exp1 server2:/exp2
```
![图片](https://upload-images.jianshu.io/upload_images/16826267-d635e21574598cd3?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##  复制模式
复制模式，既AFR, 创建volume 时带 replica x 数量: 将文件复制到 replica x 个节点中。
```
gluster volume create test-volume replica 2 transport tcp server1:/exp1 server2:/exp2
```
![图片](https://upload-images.jianshu.io/upload_images/16826267-0760d3458a15e6c6?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 条带模式
条带模式，既Striped, 创建volume 时带 stripe x 数量： 将文件切割成数据块，分别存储到 stripe x 个节点中 ( 类似raid 0 )。
```
gluster volume create test-volume stripe 2 transport tcp server1:/exp1 server2:/exp2
```
![图片](https://upload-images.jianshu.io/upload_images/16826267-cef31070f2818e35?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##  分布式条带模式

分布式条带模式（组合型），最少需要4台服务器才能创建。 创建volume 时 stripe 2 server = 4 个节点： 是DHT 与 Striped 的组合型。
```
gluster volume create test-volume stripe 2 transport tcp server1:/exp1 server2:/exp2 server3:/exp3 server4:/exp4
```
![图片](https://upload-images.jianshu.io/upload_images/16826267-0a6ce3eb942a4fa7?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##  分布式复制模式

分布式复制模式（组合型）, 最少需要4台服务器才能创建。 创建volume 时 replica 2 server = 4 个节点：是DHT 与 AFR 的组合型。
```
gluster volume create test-volume replica 2 transport tcp server1:/exp1 server2:/exp2　server3:/exp3 server4:/exp4
```
![图片](https://upload-images.jianshu.io/upload_images/16826267-f0ae8d88ed2b03f1?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##  条带复制卷模式

条带复制卷模式（组合型）, 最少需要4台服务器才能创建。 创建volume 时 stripe 2 replica 2 server = 4 个节点： 是 Striped 与 AFR 的组合型。
```
gluster volume create test-volume stripe 2 replica 2 transport tcp server1:/exp1 server2:/exp2 server3:/exp3 server4:/exp4
```
![图片](https://upload-images.jianshu.io/upload_images/16826267-d84ef11f41c0576d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##  三种模式混合

三种模式混合, 至少需要8台 服务器才能创建。 stripe 2 replica 2 , 每4个节点 组成一个 组。
```
gluster volume create test-volume stripe 2 replica 2 transport tcp server1:/exp1 server2:/exp2 server3:/exp3 server4:/exp4 server5:/exp5 server6:/exp6 server7:/exp7 server8:/exp8
```
![图片](https://upload-images.jianshu.io/upload_images/16826267-1987b7c354dcaba7?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#  修复

#查看这个卷是否在线
```
gluster volume status test2
```

#启动完全修复
```
gluster volume heal test2 full
gluster volume heal gv1 full
```

#查看需要修复的文件
```
gluster volume heal test2 info
```

#查看修复成功的文件
```
gluster volume heal test2 info healed
```

#查看修改失败的文件
```
gluster volume heal test2 info heal-failed
```

#查看脑裂的文件
```
gluster volume heal test2 info split-brain
```
