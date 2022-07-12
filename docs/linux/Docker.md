# Docker

## 一.安装docker
```shell
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
# 启动docker
systemctl start docker
# 设置开机启动
systemctl enable docker
```

### 2.1配置国内镜像源
```powershell
# vi /etc/docker/daemon.json
{
    "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
# systemctl restart docker.service
```

开启2375端口

1. 修改配置文件`docker.service`,ExecStart添加`-H tcp://0.0.0.0:2375`

`vi /usr/lib/systemd/system/docker.service`
```yaml
[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -H tcp://0.0.0.0:2375
```

2. 重启docker

`systemctl daemon-reload`
`systemctl restart docker.service`
### 3.docker-compose安装
```shell

curl -SL https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
## 二.常用命令
> 1. docker官方文档:[https://docs.docker.com/engine/reference/commandline/run/](https://docs.docker.com/engine/reference/commandline/run/)

```powershell
# 使用镜像开启一个新的容器并建立一个shell -d是后台
docker run -itd node bash

# 进入指定容器(f7是运行的容器前两位),并打开一个shell,使用这个命令退出(exit)容器不会关闭
docker exec -it f7 bash

# 查看正在运行的容器
docker ps

# 查看所有容器
docker ps -a

# linux文件复制到docker容器内
docker cp /etc/docker/node-serve f7:/usr/src

# 指定映射端口以指定镜像在后台开启一个容器
# -p 宿主机端口:容器端口 这里是把宿主机的80端口映射到容器8080端口上
docker run -p 80:80 -it -d nginx

# 启停指定id容器
docker start/stop f7

# 删除指定id的容器
docker rm ff

# linux和容器传文件
docker cp /etc/docker/node-serve/server.js 1e2:/usr/src/node-serve

# 开启一个容器并挂载文件(容器内使用主机配置文件) -v 是挂载命令,可以挂载文件或文件夹
docker run -p 80:80 -p 8081:8081 -v /mnt/nginx/nginx.conf:/etc/nginx/nginx.conf -d nginx 

 容器目录不能为相对目录,以下命令会出错
docker run -it -v /test:soft centos /bin/bash
 宿主机目录可以为相对目录,但相对的是/var/lib/docker/volumes/，与宿主机的当前目录无关.
 docker run -it -v test1:/soft centos /bin/bash
 要查看宿主机相对目录创建在哪可以使用 docker inspect 命令
docker inspect test1 

# 查看镜像信息
docker inspect redis

# 查询镜像
docker images | grep <REPOSITORY> | awk '{print $3}'

# 批量删除镜像
docker rmi $(docker images | grep "none" | awk '{print $3}')

启动        systemctl start docker
守护进程重启   sudo systemctl daemon-reload
重启docker服务   systemctl restart  docker
重启docker服务  sudo service docker restart
关闭docker service docker stop
关闭docker systemctl stop docker
```



> 在阿里云控制台添加过端口以后，创建容器可能会报错：iptables: No chain/target/match by that name
> 此时应该重启docker容器

> 安装Docker 报错公钥尚未安装
cat /etc/redhat-release
`rpm --import [http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7](https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7)`

防火墙关闭，端口仍不能访问，需要先打开防火墙，添加端口，再关闭防火墙即可
修改镜像名和tag
```shell
docker tag nav-iamge:latest nav-iamge:old
```



## 五.问题处理
```powershell
# 1. 输入ip addr看不到ip地址的问题
vi ect/sysconfig/network-scripts
ONBOOT=no改为ONBOOT=yes

# 2.docker 容器挂载配置文件时,如果配置文件为空,则启动不了容器

```

## 六.docker数据卷
如果你有一些需要持续更新的数据并且希望**持久化数据**，或者需要在不同的容器之间**共享数据**，再者需要主机与容器之间共享数据，那么你可以使用数据卷来满足这些需求
### 6.1数据卷定义
数据卷是一个可供一个或多个容器使用的特殊目录,它绕过 UFS,可以提供很多有用的特性:
数据卷可以在容器之间共享和重用。
对数据卷的修改会立马生效。
数据卷默认会一直存在，即使容器被删除。

### 6.2使用数据卷
数据卷有两种创建方式一是创建容器时创建数据卷，二是先创建好数据卷，然后在创建容器时挂载这个数据卷，两种方式均可以。

1. 创建一个数据卷
```shell
docker volume create demo-data
demo-data
```
2. 创建容器使用-v(--volume)参数来挂载数据卷
```shell
docker run --name demo1 -d -v demo-data:/var/www/html nginx:alpine
docker run --name demo2 -d -v demo-data:/var/www/html nginx:alpine
```

3. 列出数据卷
```shell
docker volume ls
DRIVER      VOLUME NAME
local       demo-data
```

4. 查看数据卷详细信息
```shell
docker volume inspect demo-data
[
  {
    "Driver": "local",
    "Labels": {},
    "Mountpoint": "/var/lib/docker/volumes/demo-data/_data",
    "Name": "demo-data",
    "Options": {},
    "Scope": "local"
  }
]
```

5. 删除数据卷
```shell
docker volume rm demo-data
Error response from daemon: remove demo-data: volume is in use - [#省略..]
```
注意: 由于有容器正在使用数据卷，提示无法删除数据卷。

1. 宿主机写入数据
```shell
hostname > /var/lib/docker/volumes/demo-data/_data/hosts.txt
```
注意: 这个目录对应创建的 demo-data 数据卷。

2. 容器写入数据
```shell
docker exec -ti demo1 sh -c 'hostname >> /var/www/html/hosts.txt'
docker exec -ti demo2 sh -c 'hostname >> /var/www/html/hosts.txt'
```
3. 读取数据
```shell
cat /var/lib/docker/volumes/demo-data/_data/hosts.txt
node0
87c60cbe6147
a6bc3c00c790
```
4. 删除容器数据卷仍然保留
```shell
docker stop demo1
docker stop demo2
docker rm demo1
docker rm demo2
```
数据卷仍然存在
```shell
docker volume ls
DRIVER      VOLUME NAME
local       demo-data
```
如果需要可以使用 rm 选项删除数据卷
```shell
docker volume rm demo-data
```

数据卷已不存在
```shell
docker volume inspect demo-data
[]
Error: No such volume: demo-data
```


### 命令帮助

1. 创建容器时挂载数据卷参数
```shell
docker run --help | grep '-v,'
-v, --volume list     Bind mount a volume
```

2. 删除容器时一并删除数据卷参数(慎用)
```shell
docker rm --help | grep '-v'

-v, --volumes   Remove the volumes associated with the container
```

3. 数据卷管理命令
```shell
docker volume --help

Usage:docker volume COMMAND

Manage volumes

Commands:
create      Create a volume
inspect     Display detailed information on one or more volumes
ls          List volumes
prune       Remove all unused local volumes
rm          Remove one or more volumes
```


### 小结

最后来总结下文章中的知识点

数据卷 是被设计用来持久化数据的,它的生命周期独立于容器,Docker 不会在容器被删除后自动删除数据卷 。
如果需要在删除容器的同时移除数据卷。可以在删除容器的时候使用docker rm -v这个命令。
