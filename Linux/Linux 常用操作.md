# Linux 常用操作

## 软件安装

### 上传下载文件
```sh
[root@iZ2ze31uq228425f2tvytnZ /]# yum install lrzsz           安装
[root@iZ2ze31uq228425f2tvytnZ tools]# rz                      上传
[root@iZ2ze31uq228425f2tvytnZ tools]# szfilename.txt          下载
```
### yum 安装 MySQL
```sh
yum -y install mysql-community-server                使用yum的方式安装MySQL
systemctl enable mysqld                              加入开机启动
systemctl start mysqld                               启动MySQL服务进程
mysql_secure_installation                            重置密码
mysql -u root -p                                     登录MySQL
mysql> grant all privileges on *.* to root@"%" identified by "root" with grant option;          为root用户授权后才可远程访问
mysql> flush privileges;                             刷新系统权限表

```
> - 配置阿里yum源 : [https://developer.aliyun.com/mirror/centos?spm=a2c6h.13651102.0.0.3e221b11KmNBpQ](https://developer.aliyun.com/mirror/centos?spm=a2c6h.13651102.0.0.3e221b11KmNBpQ)
> - 其他安装教程：[https://blog.csdn.net/wohiusdashi/article/details/89358071](https://blog.csdn.net/wohiusdashi/article/details/89358071)

### 安装Redis并配置服务
#### 安装
```sh
cd /tools/redis                                      安装下载目录
wget http://download.redis.io/releases/redis-3.2.6.tar.gz                               获取安装资源
tar -zxvf redis-3.2.6.tar.gz                         解压
cd redis-3.2.6                                       进入解压后的目录
make                                                 安装

```
> 如果make失败，一般是你们系统中还未安装gcc,那么可以通过yum安装：

```sh
yum install gcc                          安装完成gcc后再执行make

```
> 在安装redis成功后，你将可以在/usr/local/redis看到一个bin的目录，里面包括了以下文件：

```sh
ls redis-*                               查看文件
redis-benchmark  redis-check-aof  redis-check-dump  redis-cli  redis-server
```
#### 做成服务
```sh
cp /tools/redis/redis-3.2.6/utils/redis_init_script /etc/rc.d/init.d/redis  将redis_init_script复制到/etc/rc.d/init.d/，同时易名为redis
chkconfig --add redis    注册服务
```
- 如果这是注册服务报 “service redis does not support chkconfig”,则更改redis脚本：

```sh
vim /etc/rc.d/init.d/redis              查看脚本信息
```

- 看到的内容如下(下内容是更改好的信息)：
```sh
#!/bin/sh 
#chkconfig: 2345 80 90 
# Simple Redis init.d script conceived to work on Linux systems 
# as it does use of the /proc filesystem. 
   
REDISPORT=6379 
EXEC=/usr/local/redis/bin/redis-server 
CLIEXEC=/usr/local/redis/bin/redis-cli 
   
PIDFILE=/var/run/redis_${REDISPORT}.pid 
CONF="/etc/redis/${REDISPORT}.conf" 
   
case "$1" in 
    start) 
        if [ -f $PIDFILE ] 
        then 
                echo "$PIDFILE exists, process is already running or crashed" 
        else 
                echo "Starting Redis server..." 
                $EXEC $CONF & 
        fi 
        ;; 
    stop) 
        if [ ! -f $PIDFILE ] 
        then 
                echo "$PIDFILE does not exist, process is not running" 
        else 
                PID=$(cat $PIDFILE) 
                echo "Stopping ..." 
                $CLIEXEC -p $REDISPORT shutdown 
                while [ -x /proc/${PID} ] 
                do 
                    echo "Waiting for Redis to shutdown ..." 
                    sleep 1 
                done 
                echo "Redis stopped" 
        fi 
        ;; 
    *) 
        echo "Please use start or stop as first argument" 
        ;; 
esac 
```

- 和原配置文件相比： 
    1. 原文件是没有以下第2行的内容的，
    ```sh
    #chkconfig: 2345 80 90
    ```
    2. 原文件EXEC、CLIEXEC参数，也是有所更改
    ```sh
    EXEC=/usr/local/redis/bin/redis-server   
    CLIEXEC=/usr/local/redis/bin/redis-cli 
    ```
    3. redis开启的命令，以后台运行的方式执行
    ```sh
    $EXEC $CONF &
    ```
> ps:注意后面的那个“&”，即是将服务转到后面运行的意思，否则启动服务时，Redis服务将


#### 将Redis的命令所在目录添加到系统参数PATH中
```sh
vi /etc/profile                                         修改profile文件
export PATH="$PATH:/tools/redis/redis-3.2.6/src/"       在最后行追加
. /etc/profile                                          然后马上应用这个文件
```
> 然后就可以用：redis-cli  命令调用redis


### Centos 7配置阿里云yum源
- 获取阿里云 repo：`wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo`
- 清理缓存：`yum clean all`
- 重新加载缓存：`yum makecache`

## 系统配置
### 防火墙端口配置
```shell
firewall-cmd --zone=public --add-port=5672/tcp --permanent   # 开放5672端口

firewall-cmd --zone=public --remove-port=5672/tcp --permanent  #关闭5672端口

firewall-cmd --reload   # 配置立即生效

firewall-cmd --zone=public --list-ports     # 查看防火墙所有开放的端口

systemctl start firewall        # 开启防火墙

systemctl stop firewalld        # 关闭防火墙

firewall-cmd --state        # 查看防火墙状态

netstat -lnpt       # 查看监听的端口

netstat -lnpt | grep 5672        # 检查端口被哪个进程占用
```