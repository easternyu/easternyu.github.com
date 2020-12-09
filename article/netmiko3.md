### netmiko-备份华为交换机配置到tftp服务器

### 代码如下

```python
#!/bin/bash python3
#filename:savetotftp.py
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
print("*****saved config*****")
print(output)
command = 'tftp 192.168.107.1 put vrpcfg.zip ' + huawei['host'] + '.zip'
print("*****transmission to tftp server*****")
output = Conn.send_command(command)
print(output)


Conn.disconnect()
```