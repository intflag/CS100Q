# Mac

## 开发环境搭建

### 软件列表

| 软件名称                                              | 下载地址                                                        | 备注         |
| :---------------------------------------------------- | :-------------------------------------------------------------- | :----------- |
| IDEA                                                  | https://www.jetbrains.com/idea/                                 |              |
| JDK                                                   |                                                                 |              |
| Maven                                                 | https://maven.apache.org/                                       |              |
| VsCode                                                | https://code.visualstudio.com/                                  |              |
| Navicat Premium                                       | https://www.macwk.com/soft/navicat-premium                      |              |
| Another Redis Desktop Manager                         | https://github.com/qishibo/AnotherRedisDesktopManager           |              |
| Docker                                                | https://hub.docker.com/editions/community/docker-ce-desktop-mac |              |
| iTerm2                                                | https://iterm2.com/downloads.html                               |              |
| CheatSheet                                            | https://www.macwk.com/soft/cheatsheet                           |              |
| ClashX                                                | https://www.wxret.xyz/                                          |              |
| Notion                                                | https://www.notion.so/product                                   |              |
| Chrome                                                | https://www.google.com/intl/zh-CN/chrome/                       |              |
| 微信                                                  | App Store                                                       |              |
| 飞书                                                  | https://www.feishu.cn/                                          |              |
| Foxmail                                               | https://www.foxmail.com/                                        |              |
| WPS                                                   | https://www.wps.cn/                                             |              |
| Typora                                                | https://typora.io/                                              |              |
| NeatDownloadManager                                   | https://www.neatdownloadmanager.com/index.php/en/               | 下载器       |
| 自动切换输入法、超级右键、better and better、键指如飞 | https://www.better365.cn/apps.html                              |              |
| Postman                                               |                                                                 |              |
| PyCharm                                               |                                                                 |              |
| SwitchHosts                                           | https://github.com/oldj/SwitchHosts                             | Host配置     |
| iStat Menus                                           | https://www.macwk.com/soft/istat-menus                          |              |
| EZip                                                  | https://ezip.awehunt.com/                                       | 只查看不解压 |
| Bartender4                                            |                                                                 | 状态栏工具   |
| Apifox                                                |                                                                 |              |
| Patina                                                |                                                                 | 画图         |
| 网易邮箱大师                                          |                                                                 |              |
| 网易有道词典                                          |                                                                 |              |
| Listen1                                               |                                                                 | 听歌         |
| RDM                                                   |                                                                 | Redis        |
| Robo 3T                                               |                                                                 | MongoDB      |
| 赤友NTFS助手                                          |                                                                 |              |
| Clipy                                                 |                                                                 |              |
| XMind                                                 |                                                                 |              |
| 腾讯柠檬清理                                          |                                                                 |              |
| Snipaste                                              |                                                                 |              |
| Docker                                                |                                                                 |              |

### Mac 系统快捷键

| 功能        | 快捷键                |
| :---------- | :-------------------- |
| 锁屏        | Control + Command + Q |
| 跳转行首/尾 | Control + A/E         |

### IDEA Mac 快捷键

参考：https://blog.csdn.net/qq_35246620/article/details/78263380

| 功能                     | 快捷键                        |
| :----------------------- | :---------------------------- |
| 重命名文件               | Shift + F6                    |
| 单行注释                 | Command + /                   |
| 多行注释                 | Command + Option + /          |
| 历史切换                 | Command + Option + 左右方向键 |
| 格式化代码               | Command + Option + L          |
| 全局查找                 | Shift + Command + F           |
| 全局替换                 | Shift + Command + R           |
| 超级键                   | Option + Enter                |
| 上下移动当前行           | Shift + Command + 方向上下键  |
| 删除当前行或选定的块的行 | Command + delete              |
| 自动整理包               | Control + Option + O          |
| 切换大小写               | Shift + Command + U           |
| 提取方法                 | Option + Command + M          |
| 环绕方式                 | Option + Command + T          |

### IDEA 配置

- 编辑器 -> 常规 -> 使用 Command + 鼠标滚轮更改字体大小
- 编辑器 -> 常规 -> 自动导入
- 编辑器 -> 常规 -> 选项卡 -> 多行
- 编辑器 -> 文件和代码模版 -> 选项卡包含 -> File Header

```
/**
 * @author liuguoxin
 * @date Created in ${YEAR}-${MONTH}-${DAY} ${TIME}
 * @description 
 * @modified By 
 * @version v1.0
 */
```

### VsCode

```
git config --add user.name intflag
git config --add user.email 1598749808@qq.com
git config --add http.proxy http://127.0.0.1:7890
git config --add https.proxy https://127.0.0.1:7890
```

### Python3

```bash
brew search python
brew install python版本
```

### Chrome 配置

```bash
# 关闭双指左右滑动触发前进后退
defaults write com.google.Chrome AppleEnableSwipeNavigateWithScrolls -bool false
```

### DNS 配置

```bash
# 配置 DNS
networksetup -setdnsservers (Network Service) (DNS IP)
networksetup -setdnsservers Wi-Fi 172.25.248.124 8.8.8.8

# 获取 DNS
networksetup -getdnsservers Wi-Fi
```

### Parallels Desktop 命令行操作
```
prlctl create：创建一个新的虚拟机。
prlctl start：启动一个虚拟机。
prlctl stop：停止一个运行的虚拟机。
prlctl suspend：暂停一个运行的虚拟机。
prlctl resume：恢复一个暂停的虚拟机。
prlctl list：列出所有可用的虚拟机。
prlctl delete：删除一个虚拟机。
prlctl snapshot-list：列出一个虚拟机的所有快照。
prlctl snapshot-create：创建一个虚拟机的快照。
prlctl snapshot-delete：删除一个虚拟机的快照。
prlctl snapshot-switch：将虚拟机恢复到一个指定的快照。
prlctl set：设置虚拟机的参数，例如内存大小、CPU数目等。
prlctl exec：在虚拟机中执行一个命令。
prlsrvctl start：启动Parallels服务。
prlsrvctl stop：停止Parallels服务。

例如：
prlctl start "Ubuntu Linux" && prlctl start "K8S Master" && prlctl start "K8S Worker01"
prlctl stop "Ubuntu Linux" && prlctl stop "K8S Master" && prlctl stop "K8S Worker01"
prlctl restart "Ubuntu Linux" && prlctl restart "K8S Master" && prlctl restart "K8S Worker01"

prlctl suspend "Ubuntu Linux" && prlctl suspend "K8S Master" && prlctl suspend "K8S Worker01"
prlctl resume "Ubuntu Linux" && prlctl resume "K8S Master" && prlctl resume "K8S Worker01"
```
