# 一.RabbitMQ单机安装
## 1.下载erlang yum仓库源
[https://packagecloud.io/rabbitmq/erlang/install#bash-rpm](https://packagecloud.io/rabbitmq/erlang/install#bash-rpm)
`curl -s https://packagecloud.io/install/repositories/rabbitmq/erlang/script.rpm.sh | sudo bash`
## 2.下载rabbitmq-server yum仓库源
`curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash`
## 3.安装ErLang
`yum  -y install erlang`
## 4.安装rabbitmq-server
`yum -y install rabbitmq-server`
## 5.启动rabbitmq-server
`systemctl start rabbitmq-server`
## 6.安装rabbitmq管理插件
`rabbitmq-plugins enable rabbitmq_management`
## 7.创建用户
rabbitmqctl add_user [username] [password]
`rabbitmqctl add_user root 1234`
_# tag the user with "administrator" for full management UI and HTTP API access_
`rabbitmqctl set_user_tags [username] administrator`
`rabbitmqctl set_user_tags root administrator`
## 8.开放端口
`firewall-cmd --add-port=15672/tcp --permanent`
`firewall-cmd --reload`
![image.png](https://cdn.nlark.com/yuque/0/2022/png/2473493/1655109317814-006372e5-f383-4c86-b2df-009f97f88440.png#clientId=u6a264ce1-3ef7-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=736&id=u5f2044e5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=736&originWidth=966&originalType=binary&ratio=1&rotation=0&showTitle=false&size=132067&status=done&style=none&taskId=u95ba8e49-845f-47d1-8b16-0acdbec71f8&title=&width=966)
## 9.使用
ip:15672打开web管理

# 二.RabbitMQ单主机集群安装
参考文档：[https://www.rabbitmq.com/clustering.html](https://www.rabbitmq.com/clustering.html)
## 1.关闭RabbitMQ
`systemctl stop rabbitmq-server`
## 2.关闭管理插件(不关闭的话不能单机不能启动多个实例)
`rabbitmq-plugins enable rabbitmq_management`
## 3.启动多个实例
`RABBITMQ_NODE_PORT=5672 RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port,15672}]" RABBITMQ_NODENAME=rabbit rabbitmq-server -detached `
`RABBITMQ_NODE_PORT=5673 RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port,15673}]" RABBITMQ_NODENAME=hare rabbitmq-server -detached`
> RABBITMQ_NODE_PORT：节点服务端口
> RABBITMQ_SERVER_START_ARGS：启动参数
> RABBITMQ_NODENAME：节点名称
> rabbitmq-server -detached ：以独立的方式启动

## 4.设置主从
默认启动都是主节点，把其他节点都设置为其中一个的从节点即可
```shell
rabbitmqctl -n hare stop_app
rabbitmqctl -n hare join_cluster rabbit@`hostname -s`
rabbitmqctl -n hare start_app
```
