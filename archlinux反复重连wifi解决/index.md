# ArchLinux反复重连wifi解决


 ## 解决ArchLinux用NetworkManager和wpa_suppicant连接wifi反复重连

 ### 背景

 家用路由器改为ap模式后，wifi反复重连，正常路由模式未出现这种现象。

### 原因
ArchLinux尽然默认使用NetworkManager到内置到dhcp，内置dhcp服务和ArchLinux本地的dhcp服务冲突。

### 解决方法
- 更改dhcpcd的配置文件/etc/dhcpcd.conf，将duid改为clientid，然后重启服务。
```bash
systemctl restart dhcpcd@[interface].service
```

- 安装上dhclient。
```bash
sudo pacman -S dhclient
```

- /etc/NetworkManager/NetworkManager.conf 这个配置文件，增加。
```bash
[main]
dhcp = dhclient
```

- 重启服务。
```bash
sudo systemctl restart NetworkManager
```


