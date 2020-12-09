### Ceph简单使用

- 更新系统、安装阿里云的epel源
    ```bash
    # yum update -y
    # wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
    # yum makecache
    [yum repolist检查一下,并不必须]
    ```
- 添加阿里云的ceph源
    ```bash
    # cat << EOM > /etc/yum.repos.d/ceph.repo
    [ceph-noarch]
    name=Ceph noarch packages
    baseurl=https://mirrors.aliyun.com/ceph/rpm-nautilus/el7/noarch/
    enabled=1
    gpgcheck=1
    type=rpm-md
    gpgkey=https://mirrors.aliyun.com/ceph/keys/release.asc
    EOM
    ```

- 安装ceph-deploy
    ```bash
    # yum install ceph-deploy -y
    ```

- 创建用于ceph部署和管理用户
    ```bash
    # useradd ceph-admin
    # echo "123456" | passwd ceph-admin --stdin
    # echo "ceph-admin ALL = (root) NOPASSWD:ALL" | tee /etc/sudoers.d/ceph-admin
    # chmod 0440 /etc/sudoers.d/ceph-admin
    ```

- ssh免密登陆
    ```bash
    两个节点的ceph-admin用户下执行：
    $ ssh-keygen -t rsa -P ''
    $ ssh-copy-id -i ~/.ssh/id_rsa.pub ceph-admin@ceph-1
    $ ssh-copy-id -i ~/.ssh/id_rsa.pub ceph-admin@ceph-2
    [ssh ceph-1 && ssh ceph-2检查一下是否生效,并不必须]
    ```

- 防火墙放行ceph服务
    ```bash
    $ sudo firewall-cmd --zone=public --add-service=ceph-mon --permanent
    $ sudo firewall-cmd --zone=public --add-service=ceph --permanent
    $ sudo firewall-cmd --reload
    ```

- 关闭SELinux
    ```bash
    $ sudo setenforce 0
    $ sudo sed -i 's@SELINUX=enforcing@SELINUX=disabled@' /etc/sysconfig/selinux
    ```

+ 开始部署ceph集群

    - 创建保存配置文件所需的目录
        ```bash
        $ mkdir -pv my-ceph
        $ cd my-ceph
        ```
    - 创建一个ceph集群
        ```bash
        $ ceph-deploy new ceph-1

        有可能报错：ImportError: No module named pkg_resources
        安装python-pip包即可
        ```
    - 修改ceph.conf文件
        ```bash
        增加:
        public network = 192.168.186.0/24
        cluster network = 192.168.100.0/24
        ```
    - 安装ceph包
        ```bash
        $ ceph-deploy install --no-adjust-repos ceph-1 ceph-2

        这里可能需要修改一下ceph源:
        $ sudo yum install ceph-release
        $ cd /etc/yum.repos.d/
        $ sudo mv ceph.repo.rpmnew ceph.erpo
        $ sudo sed -i 's@download.ceph.com@mirrors.aliyun.com/ceph@' ceph.repo
        ```
    - 初始化ceph monitor(s)
        ```bash
        $ ceph-deploy mon create-initial
        ```
    - 复制配置文件到各节点
        ```bash
        $ ceph-deploy admin ceph-1 ceph-2
        ```
    - 创建ceph manager
        ```bash
        $ ceph-deploy mgr create ceph-1
        ```
    - 添加OSD
        ```bash
        $ ceph-deploy osd create --data /dev/sdb ceph-1
        $ ceph-deploy osd create --data /dev/sdc ceph-1
        $ ceph-deploy osd create --data /dev/sdb ceph-2
        $ ceph-deploy osd create --data /dev/sdc ceph-2
        ```
    - 检查集群健康状态
        ```bash
        $ sudo ceph -s
        或者
        $ sudo ceph health
        ```
    
+ 客户端挂载
    - 服务端创建存储池
        ```bash
        $ sudo ceph osd pool create PoolName 128
        [sudo ceph osd lspools检查一下,并不必须]
        ```
    - 服务端创建用户
        ```bash
        $ sudo ceph auth get-or-create client.rbd mon 'allow r' osd 'allow class-read object_prefix rbd_children,allow rwx pool=PoolName' > /etc/ceph/ceph.client.rbd.keyring
        ```
    - 客户端安装ceph
        ```bash
        # yum install ceph ceph-radosgw -y
        ```
    - 服务端复制ceph配置文件和keyring文件到客户端
        ```bash
        $ scp /etc/ceph/ceph.conf root@client:/etc/ceph/
        $ scp /etc/ceph/ceph.client.rbd.keyring root@client:/etc/ceph/
        ```
    - 客户端查看ceph集群信息
        ```bash
        # ceph -s --name client.rbd
        ```
    - 客户端创建iamges
        ```bash
        # rbd create ImageName --size 10240 --name client.rbd
        ```
    - 客户端映射image到本机
        ```bash
        # rbd map --image ImageNmae --name client.rbd
        CentOS不支持feature报错,需要执行以下命令:
        # rbd feature disable rbd1 object-map fast-diff deep-flatten --name client.rbd
        [rbd showmapped检查映射,不是必须]
        ```
    - 客户端挂载使用
        ```bash
        # mkfs.xfs -i size=512 /dev/rbd0
        # vim /etc/fstab
        /dev/rbd0 /data xfs defaults 0 0
        # mount -a
        ```