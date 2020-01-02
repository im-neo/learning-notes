# Linux 各种问题处理

## 开机自动激活网卡
1. 打开文件
> `vim /etc/sysconfig/network-scripts/ifcfg-ens33`

2. 编辑文件
> ONBOOT="no" 改为 ONBOOT="yes" 

3. 重启网络服务
> `systemctl restart network.service`