## GlusterFS存储集群(双节点)

### 环境说明:
```
CentOS 7.7.1908
配置:双核CPU1G内存20G系统盘20Gsdb
```

### 部署集群
+ 更新系统
    ```bash
    # yum update -y
    ```

+ 更改主机名
    ```bash
    # hostnamectl set-hostname nodeN.yu.com
    ```

+ 写入hosts文件映射
    ```bash
    # vim /etc/hosts 
    192.168.186.219  node1.yu.com  node1
    192.168.186.220  node2.yu.com  node2
    ```

+ 安装GlusterFS源
    ```bash
    # yum install centos-release-gluster6
    # yum makecache
    ```

+ 安装并启动GluterFS-server
    ```bash
    # yum install glusterfs-server -y
    # systemctl start glusterd
    ```

+ 防火墙添加服务
    ```bash
    # firewall-cmd --permanent --add-service=glusterfs
    # firewall-cmd --reload
    ```

+ 添加对等体
    ```bash
    # gluster peer probe node2.yu.com
    # gluster peer status //检查一下是否添加成功
    ```

### 集群的基本使用
+ 使用第二块20G的硬盘创建brick
    ```bash
    # fdisk /dev/sdb //将所有空间都划分给sdb1
    # pvcreate /dev/sdb1
    # vgcreate vgpool /dev/sdb1
    # lvcreate -L 15G --type thin-pool -n vgpool/lvpol
    # lvcreate -V 5G -T vgpool/lvpool -n bricklv
    # mkfs.xfs -i size=512 /dev/vgpool/bricklv
    # mkdir -pv /brick-nodeN
    # vim /etc/fstab
    /dev/vgpool/bricklv  /brick-nodeN  xfs defaults 0 0
    # mount -a
    # mkdir -pv /brick-nodeN/brick
    ```

+ 创建卷
    - 创建分布式卷
    ```bash
    # gluster volume create disvol node1:/brick-node1/brick node2:/brick-node2/brick
    # gluster volume start disvol
    ```
    - 创建复制卷
    ```bash
    # gluster volume create repvol replica 2 node1:/brick-node1/brick node2:/brick-node2/brick
    # gluster volume start repvol
    ```

+ 查看卷
    - 列出所有卷
    ```bash
    # gluster volume list
    ```
    - 查看卷详细信息
    ```bash
    # gluster volume info [vol-name]
    ```

+ 删除卷
    ```bash
    # gluster volume stop vol-name
    # gluster volume delete vol-name
    ```
### 客户端使用
+ 客户端安装Glusterfs-fuse
    ```bash
    # yum install glusterfs-fuse -y
    ```
+ 创建挂载点
    ```bash
    mkdir -pv /data
    ```
+ 挂载使用
    ```bash
    # vim /etc/fstab
    node1:/vol-name  /data glusterfs defaults,_netdev,backupvolfile-server:node2 0 0
    # mount -a
    ```