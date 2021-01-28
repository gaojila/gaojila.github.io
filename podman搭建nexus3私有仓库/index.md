# podman搭建nexus3私有仓库


# 前言

使用新工具 podman 搭建 docker 私有仓库下面简述 podman 的优点

- podman 无守护进程
- 每一个容器就是一个进程
- 非 root 用户也可执行
- podman 命令和 docker 命令大体一致

# 方法

## ubuntu 安装 podman

```bash
sudo apt update
sudo apt -y  install software-properties-common
sudo add-apt-repository -y ppa:projectatomic/ppa
sudo apt install podman -y
```

## podman 启动 nexus 容器

```bash
podman run -d -p 8081:8081 -p 8082:8082 -p 8083:8083  -v /data/nexus3_podman:/nexus-data --name nexus3 docker.io/sonatype/nexus3
```

## podman 查看容器启动日志

```bash
podman logs -f nexus3
```

## 添加仓库(hosted,proxy,group)

- 样列
  ![](https://raw.githubusercontent.com/gaojila/images/master/podman%E6%90%AD%E5%BB%BAnexus3%E7%A7%81%E6%9C%89%E4%BB%93%E5%BA%93/2019-08-17-11-46-16.png)
- 配置
  ![](https://raw.githubusercontent.com/gaojila/images/master/podman%E6%90%AD%E5%BB%BAnexus3%E7%A7%81%E6%9C%89%E4%BB%93%E5%BA%93/2019-08-17-11-52-28.png)
  ![](https://raw.githubusercontent.com/gaojila/images/master/podman%E6%90%AD%E5%BB%BAnexus3%E7%A7%81%E6%9C%89%E4%BB%93%E5%BA%93/2019-08-17-11-52-44.png)
  ![](https://raw.githubusercontent.com/gaojila/images/master/podman%E6%90%AD%E5%BB%BAnexus3%E7%A7%81%E6%9C%89%E4%BB%93%E5%BA%93/2019-08-17-11-51-54.png)

## 使用 nginx 代理方式将 pull 和 push 合并成一个端口

```nginx
upstream nexus_docker_get {
    server 192.168.157.110:8082;
}

upstream nexus_docker_put {
    server 192.168.157.110:8083;
}
server {
    listen 80;
    listen 443 ssl;
    server_name idocker.io;
    access_log /var/log/nginx/idocker.io.log;
    # 证书
    ssl_certificate /usr/local/nginx/conf/ssl/out/idocker.io/idocker.io.crt;
    ssl_certificate_key /usr/local/nginx/conf/ssl/out/cert.key.pem;
    ssl_protocols TLSv1.1 TLSv1.2;
    ssl_ciphers '!aNULL:kECDH+AESGCM:ECDH+AESGCM:RSA+AESGCM:kECDH+AES:ECDH+AES:RSA+AES:';
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    # disable any limits to avoid HTTP 413 for large image uploads
    client_max_body_size 0;
    # required to avoid HTTP 411: see Issue #1486 (https://github.com/docker/docker/issues/1486)
    chunked_transfer_encoding on;
    # 设置默认使用推送代理
    set $upstream "nexus_docker_put";
    # 当请求是GET，也就是拉取镜像的时候，这里改为拉取代理，如此便解决了拉取和推送的端口统一
    if ( $request_method ~* 'GET') {
        set $upstream "nexus_docker_get";
    }
    index index.html index.htm index.php;
    location / {
            proxy_pass http://$upstream;
            proxy_set_header Host $host;
            proxy_connect_timeout 3600;
            proxy_send_timeout 3600;
            proxy_read_timeout 3600;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_buffering off;
            proxy_request_buffering off;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto http;
    }
}
```

# 坑

- 在映射外部路径时需要将映射目录的 owner 改为 200(nexus 的 uid 和 gid)

