# ELK 搭建

## 安装 ElasticSearch

```bash
# 下载安装包，官网地址：https://www.elastic.co/cn/downloads/elasticsearch
[root@localhost opt]# wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.14.1-linux-x86_64.tar.gz
# 解压
[root@localhost opt]# tar -zxvf elasticsearch-7.14.1-linux-x86_64.tar.gz
# 修改配置
[root@localhost opt]# vim elasticsearch-7.14.1/config/elasticsearch.yml
```

```yaml
# 配置如下：
cluster.name: my-application

node.name: node-1

path.data: /opt/elasticsearch-7.14.1/data

path.logs: /opt/elasticsearch-7.14.1/logs

network.host: 0.0.0.0

http.port: 9200

cluster.initial_master_nodes: ["node-1"]
```

```bash
# vm.max_map_count 参数配置
[root@localhost opt]# vim /etc/sysctl.conf
```

```conf
# 加上以下配置 max_map_count 文件包含限制一个进程可以拥有的VMA(虚拟内存区域)的数量。
vm.max_map_count=262144
```

```bash
# 使配置立即生效
[root@localhost opt]# sysctl -p
# limits.conf 参数配置
[root@localhost opt]# vim /etc/security/limits.conf
```

```conf
# limits.conf文件限制着用户可以使用的最大文件数，最大线程，最大内存等资源使用量。
# soft是一个警告值，而hard则是一个真正意义的阀值，超过就会报错
* soft nofile 655350  #任何用户可以打开的最大的文件描述符数量，默认1024，这里的数值会限制tcp连接
* hard nofile 655350
* soft nproc  655350  #任何用户可以打开的最大进程数
* hard nproc  650000
```

```bash
# 添加用户[elastic]，ElasticSearch 不可用 root 用户启动
[root@localhost opt]# useradd elastic
# 修改用户[elastic] 的密码
[root@localhost opt]# passwd elastic
更改用户 elastic 的密码 。
新的 密码：
无效的密码： 密码少于 8 个字符
重新输入新的 密码：
passwd：所有的身份验证令牌已经成功更新。
# 授权
[root@localhost opt]# chown -R elastic:elastic elasticsearch-7.14.1
# 切换用户 elastic
[root@localhost opt]# su elastic
[elastic@localhost opt]$
# 启动 ElasticSearch
[elastic@localhost opt]$ nohup elasticsearch-7.14.1/bin/./elasticsearch &
[1] 13252
[elastic@localhost opt]$ nohup: 忽略输入并把输出追加到"/home/elastic/nohup.out"
# 测试链接
[elastic@localhost opt]$ curl localhost:9200
# 返回以下信息则启动成功
{
  "name" : "node-1",
  "cluster_name" : "my-application",
  "cluster_uuid" : "LP_P_0XNThyYTKj8wpJ8xA",
  "version" : {
    "number" : "7.14.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "66b55ebfa59c92c15db3f69a335d500018b3331e",
    "build_date" : "2021-08-26T09:01:05.390870785Z",
    "build_snapshot" : false,
    "lucene_version" : "8.9.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
[elastic@localhost opt]$
```

## 安装 Kibana

```bash
# 下载安装包，官网地址：https://www.elastic.co/cn/downloads/kibana
[root@localhost opt]# wget https://artifacts.elastic.co/downloads/kibana/kibana-7.14.1-linux-x86_64.tar.gz
# 解压
[root@localhost opt]# tar -zxvf kibana-7.14.1-linux-x86_64.tar.gz
# 修改配置文件
[root@localhost opt]# vim kibana-7.14.1-linux-x86_64/config/kibana.yml
```

```yaml
# 配置如下
# Kibana 开放端口
server.port: 5601
# 指定 Kibana 服务器绑定到的地址
server.host: "0.0.0.0"
# Elasticsearch 地址,因为我装同一台机制上，所以用 localhost
elasticsearch.hosts: ["http://localhost:9200"]
```

```bash
# 和elasticSearch一样，Kibana 不能使用root用户启动，这里就授权我们之前创建的用户 elastic
[root@localhost opt]# chown -R elastic:elastic kibana-7.14.1-linux-x86_6
# 切换用户
[root@localhost opt]# su elastic
# 后台启动 Kibana
[elastic@localhost opt]$ nohup kibana-7.14.1-linux-x86_64/bin/kibana &
[1] 14610
[elastic@localhost opt]$ nohup: 忽略输入并把输出追加到"/home/elastic/nohup.out"
# 浏览器访问: http://x.x.x.x:5601
# 可以看到 Kibana 的 web 交互界面，其中x.x.x.x为自己服务地址
```

## 安装 Logstash

```bash
# 下载安装包 官网地址：https://www.elastic.co/cn/downloads/logstash
[root@localhost opt]# wget https://artifacts.elastic.co/downloads/logstash/logstash-7.14.1-linux-x86_64.tar.gz
# 解压
[root@localhost opt]# tar -zxvf logstash-7.14.1-linux-x86_64.tar.gz
# 添加 logstash-logback.conf 配置文件
[root@localhost opt]# touch logstash-7.14.1/config/logstash-logback.conf
# 修改配置
[root@localhost opt]# vim logstash-7.14.1/config/logstash-logback.conf
```

```yaml
# 配置如下：
# Sample Logstash configuration for creating a simple
# Beats -> Logstash -> Elasticsearch pipeline.
# input表示输入源，可以有file，tcp，beats 等输入源，可以下去自己了解
# host => "0.0.0.0" 你自己服务器ip，port => 9600 端口
input {
  tcp {
    host => "0.0.0.0"
    port => 9600
    codec => json_lines
  }
}
# output表示输出
# hosts => ["http://localhost:9200"] 你自己的 elasticsearch 地址
output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "logstash-%{+YYYY.MM.dd}"
    #user => "elastic"
    #password => "changeme"
  }
  #stdout {codec => rubydebug }
}
```

```bash
# 启动 Logstash，-f 命令指定我们刚才新建的配置文件启动
[root@localhost opt]# nohup logstash-7.14.1/bin/logstash -f logstash-7.14.1/config/logstash-logback.conf &
[1] 14913
[root@localhost opt]# nohup: 忽略输入并把输出追加到"nohup.out"
```
