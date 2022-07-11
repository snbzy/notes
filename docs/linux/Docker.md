# 一.安装docker
## 1.手动安装dcoker
```powershell
# 0.更新yum
yum update

# 1.下载旧版本的docker
yum remove docker docker-client docker-client-latest docker-common docker-latest \
    docker-latest-logrotate docker-logrotate docker-engine
    
# 2. 安装所需的软件包。yum-utils 提供了 yum-config-manager，
# 并且 device mapper 存储驱动程序需要 device-mapper-persistent-data 和 lvm2。
yum install -y yum-utils device-mapper-persistent-data lvm2

# 3.配置阿里云docker软件源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 4.安装 Docker Engine-Community 如果提示您接受 GPG 密钥，请选是。
yum install docker-ce docker-ce-cli containerd.io

# 5.启动docker
systemctl start docker

# 6.设置开机启动
systemctl enable docker

# 7.验证安装成功
docker version

```

## 2.使用脚本安装
```shell
 curl -fsSL https://get.docker.com -o get-docker.sh
 sudo sh get-docker.sh
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
## 3. docker-compose安装
```shell

curl -SL https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
# 二.常用命令
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

## 安装Docker 报错公钥尚未安装
cat /etc/redhat-release
`rpm --import [http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7](https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7)`

防火墙关闭，端口仍不能访问，需要先打开防火墙，添加端口，再关闭防火墙即可
修改镜像名和tag
```shell
docker tag nav-iamge:latest nav-iamge:old
```

# 三.Linux常用命令
```powershell
#修改配置文件
vi /etc/sysconfig/network-scripts/ifcfg-ens33

# centos7
service network restart

# 安装netstat
yum install net-tools

# 查看端口
netstat -tunlp | grep <端口>

# 查看防火墙状态
firewall-cmd --state

# 停止/开启防火墙
systemctl stop/start firewalld.service

# 关闭/启用开机启动防火墙
systemctl disable/enable firewalld.service

#重启防火墙
systemctl restart firewalld.service

# 查询指定端口是否已开 yes表示开放 no表示未开放
firewall-cmd --query-port=8080/tcp

# 添加指定需要开放的端口
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --add-port=8000-9999/tcp --permanent

# 移除指定端口
firewall-cmd --remove-port=8080/tcp --permanent 

# 重启防火墙
firewall-cmd --reload

# linux 使用lrzsz(linux软件) 和Secure-CRT(windows软件)传输文件
# 1.先在linux安装lrzsz
yum install lrzsz
# 1.1上传windows->linux:输入rz命令,Secure-CRT会弹出一个窗口选择文件
rz
# 1.2下载linux->windows:输入sz命令,
# Secure-CRT默认保存路径设置Options -> Session Options -> Terminal -> Xmodem/Zmodem ->Directories
sz /etc/docker/package-lock.json


# cat 查看文件内容
cat package.json

# 测试请求web服务器的工具
curl http://www.baidu.com

# 查看进程
ps aux

# 终止进程
kill [pid]

# 查看port对应的pid两种方式：
1. lsof -i:3010
2. netstat -tunlp | grep 3010

# 查看进场所在目录
ls -l /proc/${pid}

# 查看进程内存占用，按内存列排序
top -o %MEM

# 查看ssh最近登录成功记录
last -f /var/log/wtmp

# 查看ssh最近登录失败记录
last -f /var/log/btmp

# 清空登录记录
cat /dev/null > /var/log/wtmp
cat /dev/null > /var/log/btmp
```

2. linux目录结构
   | /boot | 系统启动相关的文件，如内核、initrd，以及grub（BootLoader） |
   | --- | --- |
   | /etc | 配置文件 |
   | /home | 用户的家目录，每一个用户的家目录通常默认为/home/USERNAME |
   | /root | 管理员的家目录 |
   | /lib | 库文件
   静态库：单在程序中的库，其他程序不能使用该库文件
   动态库：在内存中，任何用到该库的程序都可以使用
   /lib/modules：内核模块文件 |
   | /media | 挂载点目录，移动设备
   （在windows中，插入一张光盘，系统会自动读取光盘，用户可以直接执行，但在linux中，插入光盘后需要在挂载点挂载这个设备之后才可以使用这个设备。） |
   | /mnt | 挂载点目录，额外的临时文件系统 |
   | /opt | 可选目录，第三方程序的安装目录 |
   | /proc | 伪文件系统，内核映射文件 |
   | /sys | 伪文件系统，跟硬件设备相关的属性映射文件 |
   | /tmp | 临时文件，/var/tmp |
   | /var | 可变化的文件，经常发生变化的文件 |
   | /bin | 可执行文件，用户[命令](https://www.linuxcool.com/)
   ；其中用到的库文件可能在/lib，配置文件可能在/etc |
   | /sbin | 可执行文件，管理[命令](https://www.linuxcool.com/)
   ；其中用到的库文件可能在/lib，配置文件可能在/etc |
   | /usr | 只读文件，shared read-only
   /usr/local：第三方软件 |

# 
# 


# 五.问题处理
```powershell
# 1. 输入ip addr看不到ip地址的问题
vi ect/sysconfig/network-scripts
ONBOOT=no改为ONBOOT=yes

# 2.docker 容器挂载配置文件时,如果配置文件为空,则启动不了容器

```

# 六.docker数据卷
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
