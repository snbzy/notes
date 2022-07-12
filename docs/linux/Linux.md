# Linux

## 一.Linux常用命令
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

## 二.linux目录结构
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

