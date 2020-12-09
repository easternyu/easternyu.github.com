### RHCS集群简单使用(基于CentOS+httpd)

+ 添加主机名解析
    ```bash
    [ALL-node][root@node1 ~]#cat << EOF >> /etc/hosts
    192.168.186.149  node1
    192.168.186.150  node2
    EOF
    ```
+ 设置SSH互信
    ```bash
    [ALL-node][root@node1 ~]# ssh-keygen -t rsa -P ''
    [ALL-node][root@node1 ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub node1
    [ALL-node][root@node1 ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub node2
    ```
+ 安装集群软件
    ```bash
    [ALL-node][root@node1 ~]# yum install pcs -y
    ```
+ 防火墙规则放行
    ```bash
    [ALL-node][root@node1 ~]# firewall-cmd --permanent --add-service=high-availability 
    [ALL-node][root@node1 ~]# firewall-cmd --reload
    ```
+ 启动并设置开机自启pcsd服务
    ```bash
    [All-node][root@node1 ~]# systemctl start pcsd
    [All-node][root@node1 ~]# systemctl enable pcsd
    ```
+ 设置hacluster用户密码
    ```bash
    [All-node][root@node1 ~]# echo "123456" | passwd --stdin hacluster
    ```
+ 节点认证
    ```bash
    [root@node1 ~]# pcs cluster auth node1 node1
    //按提示输入hacluster用户名和密码
    ```
+ 添加集群
    ```bash
    [root@node1 ~]# pcs cluster setup --name web node1 node2
    ```
+ 因为不涉及fence，所以关闭stonith，不然会有警告，资源启动也会不正常
    ```bash
    [root@node1 ~]# pcs property set stonith-enabled=false
    ```
+ 添加VIP资源
    ```bash
    [root@node1 ~]# pcs resource create VIP ocf:heartbeat:IPaddr2 ip=192.168.186.254 cidr_netmask=32 op monitor interval=30s
    ```
+ 安装httpd
    ```bash
    [ALL-node][root@node1 ~]# yum install httpd -y
    ```
+ 防火墙规则放行
    ```bash
    [ALL-node][root@node1 ~]# firewall-cmd --permanent --add-service=http
    [ALL-node][root@node1 ~]# firewall-cmd --reload
    ```
+ 添加测试页文件
    ```bash
    [ALL-node] [root@node1 ~]# cat << EOF > /var/www/html/index.html
    <html>
    <body>My Test Website - $(hostname)</body>
    </html>
    EOF
    ```
+ 添加status-server配置
    ```bash
    [ALL-node][root@node1 ~]# cat << EOF > /etc/httpd/conf.d/status.conf
    <Location /server-status>
       SetHandler server-status
       Require local
    </Location>
    EOF
    ```
+ 添加apache资源
    ```bash
    [root@node1 ~]# pcs resource create website ocf:heartbeat:apache \
    configfile=/etc/httpd/conf/httpd.conf \
    statusurl="http://localhost/server-status" \
    op monitor interval=1min
    ```
+ 添加资源约束，不然VIP和httpd资源会分别启动在两个节点上
    ```bash
    [root@node1 ~]# pcs constraint colocation add VIP with website INFINITY //使两个资源在一个节点上启动
    [root@node1 ~]# pcs constraint order VIP then website //先启动VIP资源再启动website资源
    ```
### 参考:
>[Pacemaker Documentmation](https://www.clusterlabs.org/pacemaker/doc/en-US/Pacemaker/2.0/html/Clusters_from_Scratch/index.html "Clusters from Scratch")