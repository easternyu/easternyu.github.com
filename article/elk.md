## ELK的简单使用
### 环境说明
> host: CentOS Linux release 7.6.1810 (Core) Minimal

> elasticsearch: version 6.3.0

> kibana: version 6.3.0

> logstash: version 6.3.0

---------------------------------------------

### 系统初始化
```bash
# hostnamectl set-hostname elk
# useradd elkuser
# echo "password" | passwd --stdin elkuser
# cp /etc/yum.repo.d/CentOS-Base.repo /etc/yum.repo.d/CentOS-Base.repo
# sed -i "s/#baseurl=http:\/\/mirror.centos.org/baseurl=http:\/\/mirrors.aliyun.com" /etc/yum.repo.d/CentOS-Base.repo
# yum update -y
# yum install wget vim net-tools java-1.8.0-openjdk -y
# systemctl stop firewalld
# systemctl disable firewalld
# sed -i "s/SELINUX=enforcing/SELINUX=disabled/" /etc/sysconfig/selinux
# setenforce 0
# mkdir -pv ~/elk-src
# cd ~/elk-src
# wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.3.0.tar.gz
# wget https://artifacts.elastic.co/downloads/kibana/kibana-6.3.0-linux-x86_64.tar.gz
# wget https://artifacts.elastic.co/downloads/logstash/logstash-6.3.0.tar.gz
# tar -zxvf elasticsearch-6.3.0.tar.gz 
# tar -zxvf kibana-6.3.0-linux-x86_64.tar.gz
# tar -zxvf logstash-6.3.0.tar.gz
# vim /etc/security/limits.conf
末尾增加:
*       hard    nofile  65536
*       soft    nofile  65536
*       soft    nproc   4096
*       hard    nproc   4096
重新登陆或者重启生效
# vim /etc/sysctl.conf
末尾增加:
vm.max_map_count = 655360
sysctl -p
# mv elasticsearch-6.3.0 /usr/local/src/elasticsearch
# mv kibana-6.3.0-linux-x86_64 /usr/local/src/kibana
# mv logstash-6.3.0 /usr/local/src/logstash
```
### 安装Elasticsearch
```bash
# cd /usr/local/src
# chown -R elkuser:elkuser elasticsearch
# vim /usr/local/src/elasticsearch/config/elasticsearch.yml
末尾增加:
network.host: 0.0.0.0
http.port: 9200
# su elkuser
$ sh /usr/local/src/elasticsearch/bin/elasticsearch -d
$ curl http://localhost:9200
```
### 安装Kibana
```bash
# vim /usr/local/src/kibana/config/kibana.yml
末尾增加:
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.url: "http://localhost:9200"
nohup sh /usr/local/src/kibana/kibana &
```
### 安装Logstash
```
主要是config文件的编写
目前参考搜索引擎编写的config如下:
```
```bash
# vim /usr/local/src/logstash/config/syslog.conf

input {
    syslog {
        port => "514"
    }
}
output {
    elasticsearch {
        hosts => ["http://localhost:9200"]
    }
    stdout {}
}
```
```bash
启动Logstash:
# nohup sh /usr/local/src/logstash/bin/logstash -f /usr/local/src/logstash/config/syslog.conf &
```
## 华为交换机侧配置info-center
```bash
[huawei] info-center enable
[huawei] info-center source default channel 2 log level debugging trap state off
info-center loghost ELK-Server-IP
```
### 参考
[ELK日志系统浅析与部署](https://blog.csdn.net/qq_22211217/article/details/80764568)
[centos7下部署elasticsearch常见错误](https://blog.csdn.net/qq_22211217/article/details/80740873)