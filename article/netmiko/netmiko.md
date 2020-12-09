### netmiko简单使用
> OS:Fedora release 30 (Thirty)

> Python版本:Python 3.7.3

> netmiko版本:netmiko 2.4.1

1. 因为是最小化安装的Fedora 30 Server版，所以首先安装pip3然后安装netmiko
    ```bash
    [root@localhost script]# dnf install python3-pip -y
    [root@localhost script]# pip3 install netmiko
    ```
2. 下面是测试代码及执行结果:
    ```python
    #!/bin/bash python3
#filename:router1.py
    from netmiko import ConnectHandler
    
router1 = {
        'device_type':'huawei',
        'host':'192.168.107.12',
        'username':'admin',
        'password':'123456',
    }
    
router1Conn = ConnectHandler(**router1)
    
# 测试添加一个LoopBack口，并配置IP地址
    config_command = ['int lo 0','ip add 2.2.2.2 32']
    output = router1Conn.send_config_set(config_command)
    print(output)
    
print("========================")//分割线
    
# 验证一下结果
    output = router1Conn.send_command("dis cu int lo 0")
    print(output)
    
router1Conn.disconnect()
    ```
    执行结果：
    ```bash
    [root@localhost script]# python3 router1.py 
    system-view
    Enter system view, return user view with Ctrl+Z.
    [Huawei]int lo 0
    [Huawei-LoopBack0]ip add 2.2.2.2 32
    [Huawei-LoopBack0]return
    <Huawei>
    ======================================
    [V200R003C00]
    #
    interface LoopBack0
    ip address 2.2.2.2 255.255.255.255 
    #
    return
    ```

> 脚本执行起来有点慢，不知道是不是因为路由器是ENSP模拟的原因