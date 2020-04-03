# Window10 密钥激活及相关问题处理

## 系统激活
1. 以管理员身份运行：`C:\Windows\System32\cmd.exe`

2. 执行如下命令:
```
slmgr.vbs /upk
slmgr /ipk W269N-WFGWX-YVC9B-4J6C9-T83GX 
slmgr /skms zh.us.to
slmgr /ato
```
> 以上命令中的激活码可能需要替换，激活码获取地址：[http://www.win10w.net/windows10jihuo](http://www.win10w.net/windows10jihuo)



## 问题处理
- __错误:0xC0000022 在运行 Microsoft Windows 非核心版本的计算机上,运行”slui.exe 0x2a 0xC0000022″以显示错误文本__
    1. 打开注册表至：`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SoftwareProtectionPlatform`
    2. 将：`SkipRearm` 的值修改为 1
    3. 管理员权限运行cmd输入：`slmgr -rearm`
    4. 重启电脑
    5. 重新执行激活操作

- __错误：0xC004F025拒绝访问：所请求的操作需要提升特权__
> 以管理员身份运行：`C:\Windows\System32\cmd.exe`