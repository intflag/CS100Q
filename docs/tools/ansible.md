# Ansible
> 教程：https://www.w3cschool.cn/automate_with_ansible/

## 安装部署
### 安装
```bash
# 新增 epel-release 第三方套件来源
sudo yum install -y epel-release

# 安装 Ansible
sudo yum install -y ansible
```

### 配置
- 1、配置主控端到被控端 SSH 免密登录
- 2、配置 /etc/ansible/hosts

```
[test]
192.168.0.1
192.168.0.2

[test:vars]
ansible_ssh_private_key_file=/root/.ssh/id_rsa
```

### Hello World
```bash
ansible test -m command -a 'echo hello'

192.168.0.1 | CHANGED | rc=0 >>
Hello World
192.168.0.2 | CHANGED | rc=0 >>
Hello World
```

## PlayBook 剧本
### 编辑剧本
- vim ansible-hello.yml

```yml
- name: say 'hello world'
  hosts: test
  tasks:
    - name: echo 'hello world'
      command: echo 'hello world'
      register: result

    - name: print stdout
      debug:
        msg: '{{result}}'
```

### 执行剧本
```bash
ansible-playbook ansible-hello.yml

PLAY [say 'hello world'] ***************************************************************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************************************************
ok: [192.168.0.1]
ok: [192.168.0.2]

TASK [echo 'hello world'] **************************************************************************************************************************************************************************************
changed: [192.168.0.1]
changed: [192.168.0.2]

TASK [print stdout] ********************************************************************************************************************************************************************************************
ok: [192.168.0.1] => {
    "msg": {
        "changed": true, 
        "cmd": [
            "echo", 
            "hello world"
        ], 
        "delta": "0:00:00.021317", 
        "end": "2021-06-03 15:21:49.982764", 
        "failed": false, 
        "rc": 0, 
        "start": "2021-06-03 15:21:49.961447", 
        "stderr": "", 
        "stderr_lines": [], 
        "stdout": "hello world", 
        "stdout_lines": [
            "hello world"
        ]
    }
}
ok: [192.168.0.2] => {
    "msg": {
        "changed": true, 
        "cmd": [
            "echo", 
            "hello world"
        ], 
        "delta": "0:00:00.003884", 
        "end": "2021-06-03 15:29:19.994776", 
        "failed": false, 
        "rc": 0, 
        "start": "2021-06-03 15:29:19.990892", 
        "stderr": "", 
        "stderr_lines": [], 
        "stdout": "hello world", 
        "stdout_lines": [
            "hello world"
        ]
    }
}

PLAY RECAP *****************************************************************************************************************************************************************************************************
192.168.0.1              : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
192.168.0.2              : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

## 常用配置及命令
### 常用命令
```bash
# 查看插件信息
ansible-doc -t callback -l

# 查看插件详细信息
ansible-doc -t callback timer
ansible-doc -t module synchronize

# 调试模式运行
ansible-playbook -vvv xxx.yml
```

### 常用配置
- vim /etc/ansible/ansible.cfg

```bash
# 1、开启缓存

gathering = smart
#缓存时间，单位为秒
fact_caching_timeout = 86400    
fact_caching = jsonfile
#指定ansible包含fact的json文件位置，如果目录不存在，会自动创建
fact_caching_connection = /tmp/ansible_fact_cache

# 2、修改输出格式

stdout_callback = json
# 默认情况下只对 playbook 生效，想让 ad-hoc 生效可以修改以下配置
bin_ansible_callbacks = True

# 3、启用其他内部插件

callback_whitelist = timer, profile_roles
# timer 插件可以计算整个 playbook 执行时间
# profile_roles 插件可以在执行中添加执行时间

```

## synchronize 文件同步模块
- ansible-sync.yml

```yml
- name: synchronize dir
  hosts: test
  tasks:
    - name: synchronize dir
      synchronize: 
        src: /root/ansibleTest
        dest: "/root/ansibleTest/{{inventory_hostname}}"
        mode: pull
      register: result

    - name: print stdout
      debug:
        msg: '{{result}}'
```