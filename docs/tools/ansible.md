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