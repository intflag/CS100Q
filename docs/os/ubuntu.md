# Ubuntu
## 网卡配置
- 1 sudo vim /etc/netplan/01-network-manager-all.yaml
```
network:
  ethernets:
    enp0s5:
      dhcp4: true
    enp0s6:
      dhcp4: false
      addresses:
        - 192.168.10.11/24
      routes:
        - to: default
          via: 192.168.10.255
  version: 2
  renderer: NetworkManager
```
- 2 sudo vim /etc/netplan/01-network-manager-all.yaml
- 3 sudo netplan apply