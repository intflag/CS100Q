# Docker
## 基本使用
### 1）离线部署
- 下载安装包，如：docker-17.03.2-ce.tgz，下载地址：https://download.docker.com/linux/static/stable/x86_64/
- 准备 docker.service 系统配置文件

```bash
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s

[Install]
WantedBy=multi-user.target
```

- 准备安装脚本和卸载脚本

安装脚本 install.sh
```bash
#!/bin/sh
echo '解压tar包...'
tar -xvf $1

echo '将docker目录移到/usr/bin目录下...'
cp docker/* /usr/bin/

echo '将docker.service 移到/etc/systemd/system/ 目录...'
cp docker.service /etc/systemd/system/

echo '添加文件权限...'
chmod +x /etc/systemd/system/docker.service

echo '重新加载配置文件...'
systemctl daemon-reload

echo '启动docker...'
systemctl start docker

echo '设置开机自启...'
systemctl enable docker.service

echo 'docker安装成功...'
docker -v
```

卸载脚本 uninstall.sh
```bash
#!/bin/sh

echo '删除docker.service...'
rm -f /etc/systemd/system/docker.service

echo '删除docker文件...'
rm -rf /usr/bin/docker*

echo '重新加载配置文件'
systemctl daemon-reload

echo '卸载成功...'
```

- 上传 docker 安装包、配置文件、安装及卸载脚本到目标主机

```bash
-rw-rw-r-- 1 deployer deployer 27821092 Aug 28 13:41 docker-17.03.2-ce.tgz
-rw-rw-r-- 1 deployer deployer     1140 Aug 28 13:42 docker.service
-rwxrwxr-x 1 deployer deployer      502 Aug 28 13:44 install.sh
-rwxrwxr-x 1 deployer deployer      218 Aug 28 13:44 uninstall.sh
```
- 安装卸载命令

```bash
授予脚本执行权限
chmod +x install.sh uninstall.sh

安装命令
sudo sh install.sh docker-17.03.2-ce.tgz

卸载命令
sudo sh uninstall.sh
```

## 离线安装 Nginx