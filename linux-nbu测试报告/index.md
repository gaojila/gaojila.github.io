# linux-nbu测试报告


# NetBackup 测试

## 客户端安装(供应商暂时提供的方法)

- 供应商提供安装包(NetBackup_8.2_CLIENTS_debian.tar.gz)
- Linux 下需要手动配置 hosts 文件

  ```
  client 端指定 master 端 ip
  10.1.1.75 C01R7NBU01
  master 端指定 client 端 ip
  10.1.7.71 nj-zabbix-2-te
  ```

- 在 client 端解压压缩包开始安装(比较繁琐)
  ![](https://raw.githubusercontent.com/gaojila/images/master/linux-nbu%E6%B5%8B%E8%AF%95%E6%8A%A5%E5%91%8A/20200102-132007.png)
  ![](https://raw.githubusercontent.com/gaojila/images/master/linux-nbu%E6%B5%8B%E8%AF%95%E6%8A%A5%E5%91%8A/20200102-132432.png)
  ![](https://raw.githubusercontent.com/gaojila/images/master/linux-nbu%E6%B5%8B%E8%AF%95%E6%8A%A5%E5%91%8A/20200102-132609.png)
  ![](https://raw.githubusercontent.com/gaojila/images/master/linux-nbu%E6%B5%8B%E8%AF%95%E6%8A%A5%E5%91%8A/20200102-132749.png)
  ![](https://raw.githubusercontent.com/gaojila/images/master/linux-nbu%E6%B5%8B%E8%AF%95%E6%8A%A5%E5%91%8A/20200102-133213.png)

## 备份

### linux 文件备份

- 将刚自动发现的客户端加到对应的策略组
  ![](https://raw.githubusercontent.com/gaojila/images/master/linux-nbu%E6%B5%8B%E8%AF%95%E6%8A%A5%E5%91%8A/20200102-134934.png)
  ![](https://raw.githubusercontent.com/gaojila/images/master/linux-nbu%E6%B5%8B%E8%AF%95%E6%8A%A5%E5%91%8A/20200102-135208.png)
  ![](https://raw.githubusercontent.com/gaojila/images/master/linux-nbu%E6%B5%8B%E8%AF%95%E6%8A%A5%E5%91%8A/20200102-135343.png)
- 配置备份策略类型(attributes)
  ![](https://raw.githubusercontent.com/gaojila/images/master/linux-nbu%E6%B5%8B%E8%AF%95%E6%8A%A5%E5%91%8A/20200102-140028.png)
  ![](https://raw.githubusercontent.com/gaojila/images/master/linux-nbu%E6%B5%8B%E8%AF%95%E6%8A%A5%E5%91%8A/20200102-140209.png)
- 配置自动备份策略
- 配置文件备份路径

### mysql 备份(无法进行依赖脚本)

- master 如何调用脚本
- 全备份
- 增量备份

### postgresql 备份(无法进行依赖脚本)

- master 如何调用脚本
- 全备份
- 增量备份

### redis 文件备份(官网为提供可使用文件备份来备份)

- 使用 linux 文件备份

## api 基础测试

### 获取认证 token

```bash
url="https://c01r7nbu01.genscript.com:1556/netbackup/login"
tokenget(){
        curl -k -X POST -H 'Content-type: application/vnd.netbackup+json;version=1.0' -d '
        {
                "userName":"root",
                "password":"*********"
        }' $url
}
tokenget
```

### 新增备份服务器

```bash
url="https://c01r7nbu01.genscript.com:1556/netbackup/config/hosts"
token="*************"
curl -k -X POST $url \
  -H 'Content-type: application/vnd.netbackup+json;version=3.0' \
  -H 'Authorization: '$token'' \
  -d '{"hostName": "nj-lab-1-it"}'
```

### 新增备份任务

```bash
url="https://c01r7nbu01.genscript.com:1556/netbackup/config/policies/Linux_Bakcup/clients/nj-zabbix-1-te"
token="*************"
curl -k -X PUT \
  -H 'Content-type: application/vnd.netbackup+json;version=3.0' \
  -H 'authorization: '$token'' -d '
{
  "data": {
    "type": "client",
    "id": "nj-zabbix-1-te",
    "attributes": {
      "OS": "Debian2.6.32",
      "hardware": "Linux"
    }
  }
}' $url
```

### 未完待续(api 足够丰富)。。。。。。

