# Linux 使用笔记
## 常用命令
|功能|命令|
|:----|:----|
|查看机器配置|系统版本：cat /etc/redhat-release<br>CPU位数：uname -a<br>逻辑CPU个数：cat /proc/cpuinfo \| grep "processor" \| wc -l<br>物理CPU个数：cat /proc/cpuinfo \| grep "physical id" \| sort \| uniq \| wc -l<br>物理CPU中Core的个数：cat /proc/cpuinfo \| grep "cpu cores" \| wc -l<br>内存：free -h<br>硬盘：df -hT<br>默认语言：echo $LANG $LANGUAGE|
|时间逆序查看|ls -lrt|
|推送拉取文件|推送：scp /opt/soft/nginx-0.5.38.tar.gz root@10.10.10.10:/opt/soft/<br>拉取：scp root@10.10.10.10:/opt/soft/test.xml ./|
|查看进程|ps -ef \| grep 程序名称|
|查看端口占用情况|netstat -nltp|
|主机端口连通测试|telnet 127.0.0.1 8080|
|创建解压 tar 包|tar -cf archive.tar foo bar<br>tar -zxvf xxx.tar.gz -C 保存路径|
|创建解压 zip 包|zip -r xxx.zip xxx<br>unzip xxx.zip -d 保存路径|
|创建解压 xz 包|xz -zk 要压缩的文件<br>xz -dk 要解压的文件|
|创建解压 gzip 包|gzip -c filename > filename.gz<br>gunzip -c filename.gz > filename|
|建立多级目录|mkdir -p /opt/m1/m2/m3|
|查找文件|find . ".jar" \| xargs grep "CodeManager" 查找某个类在哪个jar包中<br>grep -rn "关键字符串" /opt/ 查找包含字符串的文件|
|查看Linux程序的工作目录|pwdx 266469|
|查看文件夹大小|目录总大小：du -sh /opt<br>直接子目录文件及文件夹大小：du -h --max-depth=0 opt/|
|定时任务配置|crontab -l 查看用户定时任务列表，-e 编辑用户定时任务|
|查看命令所在位置|which java|
|软链接的创建和删除|创建：sudo ln -s /usr/local/node-v10.15.3-linux-x64/bin/node /usr/local/bin/<br>删除：rm –rf /usr/local/redis (最后不要加/，否则会删除实际文件)<br>更新：ln –snf  /opt/apps/redis-5.0.4 /usr/local/redis|
|改变文件所有者|chown -R root:root app.jar（-R表示递归）|
|查看系统所有用户|cat /etc/passwd|


## 常用脚本
### 1、快速修改tomcat端口
```
sed -i 's/port="8005"/port="18005"/g' server.xml
sed -i 's/port="8080"/port="18080"/g' server.xml
sed -i 's/port="8443"/port="18443"/g' server.xml
sed -i 's/port="8009"/port="18009"/g' server.xml
sed -i 's/redirectPort="8443"/redirectPort="18443"/g' server.xml
```
### 2、Linux中没有`ll`命令
```bash
sudo vi ~/.bashrc

加入
alias ll='ls -lrt'

然后刷新配置
source  ~/.bashrc
```
### 3、Linux下批量杀掉筛选进程
```bash
例如批量删掉flume指定了myconf/.*properties下配置文件的进程
ps -ef|grep flume|grep -E "myconf/.*properties"|awk '{print $2}'|xargs kill -9
```
### 4、设置DNS
```bash
vi /etc/resolv.conf

加入或修改
nameserver 114.114.114.114
```
### 5、Linux下使用timedatectl命令操作时间时区
```bash
# 显示当前时间信息
timedatectl status
# 显示所有可用时区
timedatectl list-timezones
# 设置时区
timedatectl set-timezone "Asia/Shanghai"
# 设置日期和时间
timedatectl set-time '2020-02-14 12:13:24'
```
### 6、JAVA程序快速启动停止脚本
```bash
# 启动
#!/bin/bash
nohup java -classpath './lib/*:./conf/' com.intflag.TestApplication >run.log 2>&1 &

# 停止
#!/bin/bash
ID=`ps -ef | grep TestApplication | grep java | awk '{print $2}'`
echo ${ID}
kill -9 ${ID}

# 重启
#!/bin/bash
ID=`ps -ef | grep TestApplication | grep java | awk '{print $2}'`
echo ${ID}
kill -9 ${ID}
nohup java -classpath './lib/*:./conf/' com.intflag.TestApplication >run.log 2>&1 &
```

### 7、JAVA 配置环境变量
```bash
编辑：
vim /etc/profile

输入：
#JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.7.0_79
export PATH=$PATH:$JAVA_HOME/bin

刷新：
source  /etc/profile
```

### 8、端口扫描工具 nmap
```bash
# 安装
yum install nmap

# 使用
nmap 目标主机IP

# 扫描禁用 ping 命令的主机
nmap -Pn 目标主机IP
```
## 常见问题
### 1、无法使用 root 进行 ssh 登录
- 原因：禁止了 root 用户通过 ssh 登录系统的功能

```bash
# 编辑ssh配置
vi /etc/ssh/sshd_config

# 修改 PermitRootLogin 值为 yes
PermitRootLogin yes

# 重启 ssh 服务
# centos 6
service ssh restart
# centos 7
systemctl restart sshd
```
### 2、找不到 netstat 命令
```bash
# 安装网络工具
sudo yum -y install net-tools
```
### 3、ssh提示 IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY
```bash
rm -rf ~/.ssh/known_hosts
```