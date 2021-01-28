# linux系统安全设置


## 禁用 root 密码登录

- 禁用密码登录

```
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```

systemctl restart sshd

## 配置会话自动登出

- 修改.bashrc 或者.profile

```
echo "TMOUT=90">>.bashrc
source .bashrc
```

- 修改 ssh 配置

```
ClientAliveInterval 90
ClientAliveCountMax 2
```

systemctl restart sshd

## 设置帐号锁定次数，锁定时间


