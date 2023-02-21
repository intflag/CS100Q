# FRP 内网穿透
## 1、内网穿透搭建
- 配置服务端
1）下载服务端对应的安装包：https://github.com/fatedier/frp/releases

2）frps.ini 配置
```
[common]
bind_port = 7000
dashboard_port = 7500
token = 123456
dashboard_user = admin
dashboard_pwd = '控制台密码'
vhost_http_port = 10080
vhost_https_port = 10443
```

3）新建启动脚本
```
#!/bin/bash
nohup /root/frp/frp_0.46.1_linux_amd64/frps -c /root/frp/frp_0.46.1_linux_amd64/frps.ini >run.log 2>&1 &
```
- 配置客户端
1）下载客户端对应的安装包：https://github.com/fatedier/frp/releases

2）frpc.ini 配置
```
[common]
server_addr = 服务器IP
server_port = 7000
token = 123456
[rdp]
type = tcp
local_ip = 127.0.0.1           
local_port = 3389
remote_port = 7001  
[smb]
type = tcp
local_ip = 127.0.0.1
local_port = 445
remote_port = 7002
```

3）新建启动脚本
- 在任何一个目录下新建一个文本文件并将其重命名为“frpc.bat”，编辑，粘贴如下内容并保存
```
@echo off
if "%1" == "h" goto begin
mshta vbscript:createobject("wscript.shell").run("""%~nx0"" h",0)(window.close)&&exit
:begin
REM
cd C:\frp
frpc -c frpc.ini
exit
```

将cd后的路径更改为你的frpc实际存放的目录。