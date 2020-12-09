## netmiko-保存华为交换机配置
> OS:Fedora release 30 (Thirty)

> Python版本:Python 3.7.3

> netmiko版本:netmiko 2.4.1
### 使用netmiko自带的sava_config()方法操作华为交换机配置保存动作不生效，所以用其他方法实现华为交换机配置的保存

#### 代码编写：
```python
#!/bin/bash python3
#filename:save.py
from netmiko import ConnectHandler
huawei = {
    'device_type':'huawei',
    'host':'192.168.107.11',
    'username':'admin',
    'password':'123456',
}
Conn = ConnectHandler(**huawei)
commands = ['return','save','y']
output = Conn.send_config_set(commands)
print(output)
```

> commands命令执行说明：

> 因为send_config_set()方法默认进入系统视图，所以首先执行'return'返回用户视图，再执行'save'命令保存配置后键入'y'确认