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
### 1）导出 Nginx 镜像
- 查看镜像 ID

```bash
sudo docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              6678c7c2e56c        5 months ago        127MB
```
- 导出镜像

```bash
sudo docker save 6678c7c2e56c -o /home/intflag/backup/nginx.tar
```

### 2）导入 Nginx 镜像
- 导入

```bash
sudo docker load -i /data1/ftp/intell/nginx.tar 

f2cb0ecef392: Loading layer [==================================================>] 72.48 MB/72.48 MB
71f2244bc14d: Loading layer [==================================================>] 58.11 MB/58.11 MB
55a77731ed26: Loading layer [==================================================>] 3.584 kB/3.584 kB
Loaded image ID: sha256:6678c7c2e56c970388f8d5a398aa30f2ab60e85f20165e101053c3d3a11e6663

sudo docker images

REPOSITORY                            TAG                 IMAGE ID            CREATED             SIZE
<none>                                <none>              6678c7c2e56c        5 months ago        127 MB
```

- 导入后 REPOSITORY 和 TAG 都为none

### 3）重新标记

```bash
sudo docker tag 6678c7c2e56c nginx:latest

sudo docker images

REPOSITORY                            TAG                 IMAGE ID            CREATED             SIZE
nginx                                 latest              6678c7c2e56c        5 months ago        127 MB
```

### 4）启动 Nginx
```bash
sudo docker run --name nginx -d -p 8090:80 nginx

sudo docker ps -a

CONTAINER ID        IMAGE                            COMMAND                  CREATED             STATUS              PORTS                        NAMES
d7dcb79f447b        nginx                            "nginx -g 'daemon ..."   22 seconds ago      Up 21 seconds       0.0.0.0:8090->80/tcp         nginx
```

## 部署 Nginx

### 1）建立目录映射
```bash
# 外部主机中建立目录
sudo mkdir -p ./nginx/www ./nginx/logs ./nginx/conf 
```

### 2）拷贝容器 Nginx 的配置文件
```bash
sudo docker cp d7c2dfc6cd87:/etc/nginx/nginx.conf /data1/nginx/conf/
```

### 3）重新部署 Nginx

```bash
sudo docker run -d -p 10012:80 \
--name nginx-v1 \
-v /data1/nginx/www:/usr/share/nginx/html \
-v /data1/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /data1/nginx/conf.d:/etc/nginx/conf.d \
-v /data1/nginx/logs:/var/log/nginx \
nginx

-p 10012:80 表示将本地 10012 端口映射到容器 80 端口
--name nginx-v1 表示将容器命名为 nginx-v1
-v /data1/nginx/www:/usr/share/nginx/html 表示将本地目录 /data1/nginx/www 映射到容器内部目录 /usr/share/nginx/html
```

- 进www文件目录创建一个index.html

```bash
# 创建文件
sudo vi index.html

# 写入信息，然后保存

# 测试
curl 172.17.38.1:10012

Hello My Nginx!!!
```


### 参考资料
- [Docker离线安装Nginx镜像](https://zhouyanwei.cn/2019/08/13/2019-8-13-DockerNginx/)