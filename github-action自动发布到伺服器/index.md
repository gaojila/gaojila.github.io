# Github Action自动发布到伺服器

# 背景

1. 科学上网
2. 反复被ban
3. 符合自己气质的域名
4. 博客掩护科学上网
5. 免费证书自动续期

## 科学上网

v2ray 也不是那么安全了,每个月都会被ban一次

## 域名注册

阿里云注册域名,还好我这蹩脚的域名没人注册

## 自动发布博客到伺服器

1. ssh-keygen 生成密钥对
```bash
❯ ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f gh-pages -N ""
Generating public/private rsa key pair.
Your identification has been saved in gh-pages.
Your public key has been saved in gh-pages.pub.
The key fingerprint is:
SHA256:qm3ZlWXDEwfSwxmqMIQAzcDyasdfasdfuqioweqnslfansd0 redgaojila@gmail.com
The key's randomart image is:
+---[RSA 4096]----+
|+=.. ..   .oo+   |
|..+ ..     o* .  |
|.=    o   .. +   |
|.o+    o .  *    |
|*=..    S  + o   |
|==o    .  o      |
|*o   ..o .       |
|B.o .oE .        |
| = ..o.          |
+----[SHA256]-----+

❯ ll gh-pages*
Permissions Size User       Date Modified Name
.rw-------  3.4k redgaojila 29 1 14:00    gh-pages
.rw-r--r--   746 redgaojila 29 1 14:00    gh-pages.pub
```

2. 私钥配置和公钥配置

- 私钥配置

![private](https://raw.githubusercontent.com/gaojila/images/master/github-action自动发布到伺服器/private.png)

- 公钥配置

![pub](https://raw.githubusercontent.com/gaojila/images/master/github-action自动发布到伺服器/pub.png)

3. 配置Github Action(同时发布到Github Page)
```yaml
name: Hugo
on:
  push:
    branches:
      - master

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: checkout
        uses: actions/checkout@master
        with:
          submodules: true

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2.4.13
        with:
          hugo-version: '0.80.0'
          extended: true

      - name: Hugo Build
        run: hugo --gc --minify --buildFuture --cleanDestinationDir

      - name: Install algolia module
        run: npm install atomic-algolia --save-dev

      - name: Init algolia
        run: npm run algolia

      - name: Deploy Private
        env:
          ACTIONS_DEPLOY_KEY: ${{ secrets.PRIVATE_ACTIONS_DEPLOY_KEY }}
          HOST: www.gaojila.red
          USER: root
          HOME_PATH: /var/www
          DEVELOP_SH_PATH: /var/www/develop.sh
          PACKAGE_NAME: public.tar.gz
          DEVELOP_DIR: gaojila.red
          BACKUP_DIR: backup
        run: |
          SSH_PATH="$HOME/.ssh"
          mkdir -p $SSH_PATH
          touch "$SSH_PATH/known_hosts"
          echo "$ACTIONS_DEPLOY_KEY" > "$SSH_PATH/id_rsa"
          chmod 700 "$SSH_PATH"
          chmod 600 "$SSH_PATH/known_hosts"
          chmod 600 "$SSH_PATH/id_rsa"
          eval $(ssh-agent)
          ssh-add "$SSH_PATH/id_rsa"
          ssh-keyscan -t rsa $HOST >> "$SSH_PATH/known_hosts"
          cd public
          tar -cf $PACKAGE_NAME *
          scp $PACKAGE_NAME $USER@$HOST:$HOME_PATH
          ssh -o StrictHostKeyChecking=no -i $SSH_PATH/id_rsa -A -tt $USER@$HOST sh $DEVELOP_SH_PATH \
            -d $HOME_PATH/$DEVELOP_DIR -b $HOME_PATH/$BACKUP_DIR -f $HOME_PATH/$PACKAGE_NAME
          exit

      - name: Deploy Github
        uses: peaceiris/actions-gh-pages@v3.7.3
        with:
          DEPLOY_KEY: ${{ secrets.ACTIONS_DEPLOY_KEY }}
          EXTERNAL_REPOSITORY: gaojila/gaojila.github.io
          PUBLISH_BRANCH: master
          PUBLISH_DIR: ./public

```
4. 服务器内部的解包和备份由服务器内脚本develop.sh解决
```bash
root@private:/var/www# cat develop.sh
#!/bin/sh
set -e

FILE_NAME=`basename $0`

#说明
show_usage="usage:$FILE_NAME [-d develop_path,-b backup_path -f file_path]"

#参数
# 本地仓库目录
opt_develop_path=""

# 备份目录
opt_backup_path=""

# 部署文件
opt_file_path=""


GETOPT_ARGS=`getopt -o d:b:f: -al develop_path:,backup_path:,file_path: -- "$@"`
eval set -- "$GETOPT_ARGS"
#获取参数
while [ -n "$1" ]
do
        case "$1" in
                -d|--develop_path) opt_develop_path=$2; shift 2;;
                -b|--backup_path) opt_backup_path=$2; shift 2;;
                -f|--opt_file_path) opt_file_path=$2; shift 2;;
                --) break ;;
                *) echo $1,$2,$show_usage; break ;;
        esac
done

# 判断参数
if [[ -z $opt_develop_path || -z $opt_backup_path || -z $opt_file_path ]]; then
        echo -e $show_usage
        exit 0
fi

if [ "$opt_develop_path" = "$opt_backup_path" ]; then
  echo 'develop_path eq backup_path'
  exit 0
fi

# 判断部署文件是否存在
if [ ! -f $opt_file_path ]; then
    echo "$opt_file_path file does not exist"
    exit 0
fi

# 判断文件夹是否存在
if [ ! -x $opt_develop_path ]; then
  mkdir $opt_develop_path
fi

# 判断文件夹是否存在
if [ ! -x $opt_backup_path ]; then
  mkdir $opt_backup_path
fi

# 文件夹不是空的
if [ ! "`ls -A $opt_develop_path`" = "" ]; then
  cd $opt_develop_path
  tar -cf $opt_backup_path/$(date +%Y%m%d%H%M).tar.gz $opt_develop_path/*
  rm -rf $opt_develop_path/*
fi
# 解压文件
tar -xf $opt_file_path -C $opt_develop_path

echo "publish success!"
```

5. 配置服务器nginx

```nginx
server {
  listen 443 ssl;
  server_name  gaojila.red;
  ssl_certificate /etc/letsencrypt/archive/www.gaojila.red/fullchain1.pem;
  ssl_certificate_key /etc/letsencrypt/archive/www.gaojila.red/privkey1.pem;
  ssl_session_timeout 5m;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
  ssl_prefer_server_ciphers on;

  #默认请求
  location / {
  root /var/www/gaojila.red;
      index index.html index.xml;
  }

}
server
{
  listen 80;
  server_name gaojila.red;
  rewrite ^(.*) https://$host$1 permanent;
}
```
6. 获取免费的证书
  在这里我们使用Let’s Encrypt免费证书
```bash
# 安装certbot,这里我使用ubuntu apt安装
sudo apt install certbot
# 生成证书,注意路径
certbot certonly --webroot -w /var/www/gaojila.red -d www.redgaojila.red
# 证书生成完毕后，我们可以在 /etc/letsencrypt/live/ 目录下看到对应域名的文件夹
```
## 证书自动续期
添加如下定时任务,会自动检测到期
```bash
0 3 */7 * * /usr/bin/certbot renew --renew-hook "/usr/bin/nginx -s reload"
```

## 如何搭建新一代的`xray`科学上网本文不做解释

