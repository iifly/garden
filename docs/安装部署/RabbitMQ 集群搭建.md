# RabbitMQ 集群搭建

## 单机搭建

1. Erlang 安装

   官网下载地址：https://www.erlang.org/downloads，注意Erlang 与 RabbitMQ 之间的版本对应关系，本教程选择的是 Erlang 24 与 RabbitMq 3.9.7。

   但是 Erlang 官网实在下载太慢了，可以选择 RabbitMQ 提供的解决方案 ，见如下网址：https://www.erlang-solutions.com/downloads/

   ```bash
   cd /opt
   ```

   1. 添加仓库入口，要将 Erlang Solutions 存储库（包括我们用于验证签名包的公钥）添加到您的系统，请调用以下命令：

   ```bash
   wget https://packages.erlang-solutions.com/erlang-solutions-2.0-1.noarch.rpm

   rpm -Uvh erlang-solutions-2.0-1.noarch.rpm
   ```

   2. 安装 Erlang，调用以下命令安装“erlang”包：

   ```bash
   sudo yum install erlang
   ```

   3. 验证安装

   ```bash
   erl
   ```

2) 安装 RabbitMQ， 这里选择 GitHub 下载, 地址： https://github.com/rabbitmq/rabbitmq-server/releases， 一般选择 rabbitmq-server-generic-unix 版本

   ```bash
   # 下载
   wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.9.7/rabbitmq-server-generic-unix-3.9.7.tar.xz

   # 先用 xz 命令解压为 .tar 文件
   xz -d rabbitmq-server-generic-unix-3.9.7.tar.xz

   # 再用 tar 命令解压
   tar -xvf rabbitmq-server-generic-unix-3.9.7.tar

   # /etc/profile 文件添加环境变量
   vim /etc/profile

   # 末尾添加以下内容
   export PATH=$PATH:/opt/rabbitmq_server-3.9.7/sbin

   # 使配置生效
   source /etc/profile

   # 启动 RabbitMQ
   rabbitmq-server -detached

   # 查看 RabbitMQ 状态
   rabbitmqctl status
   ```

## 集群搭建

准备 3 台分别装好 RabbitMQ 的机器，192.168.0.11，192.168.0.12，192.168.0.13

```bash
# 分别修改主机名（不用修改也行，只有 hosts 配置时对应即可）
hostnamectl set-hostname rabbit-node1 # hostnamectl set-hostname rabbit-node2; hostnamectl set-hostname rabbit-node3
# 修改每台机器的 /etc/hosts 文件
vim /etc/hosts
# 都添加以下内容，ip 对应自己主机名
192.168.0.11 rabbit-node1
192.168.0.12 rabbit-node2
192.168.0.13 rabbit-node3

# 读取其中一个节点的cookie, 并复制到其他节点（节点之间通过 cookie 确定相互是否可通信）。
# cookie存放在/var/lib/rabbitmq/.erlang.cookie或者 $HOME/.erlang.cookie中，保证 3 台机器上的 cookie 相同。

# 建立集群，先把服务都启动好
rabbitmq-server -detached

# 以 rabbit-node1 为主节点，在 rabbit-node2 上执行命令：
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster rabbit@rabbit-node1
rabbitmqctl start_app
#rabbit-node3 上的操作与 rabbit-node2 的相同。

# 最后通过rabbitmqctl cluster_status查看集群的状态信息：
rabbitmqctl cluster_status
```
