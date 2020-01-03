# Linux 各种问题处理

## 开机自动激活网卡
1. 打开文件
> `vim /etc/sysconfig/network-scripts/ifcfg-ens33`

2. 编辑文件
> ONBOOT="no" 改为 ONBOOT="yes" 

3. 重启网络服务
> `systemctl restart network.service`


## 修改 Linux 终端命令行样式
1. 打开文件
> `vim /root/.bashrc`

2. 文件末尾追加
> `export PS1="[\[\e[32;1m\]\u\e[33;1m@\e[36;1m\h\[\e[0m\] \[\e[31;1m\]\W\[\e[0m\]]\\$ "`
> 
> 或
> 
> `export PS1="\[\e[32;1m\]\W \e[32;5m\]→ \e[0m\] "`

3. 编译文件
> source /root/.bashrc