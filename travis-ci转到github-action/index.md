# Travis Ci转到github Action


## 背景  
12月Travis Ci 不再为开源项目提供支持，故转战Github Action

## 准备工作
1. 生成对称密钥
2. 配置源码仓库和发布仓库密钥
3. 配置workflow
## 开始

### 生成对称密钥
```bash
❯ ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f gh-pages -N ""
Generating public/private rsa key pair.
Your identification has been saved in gh-pages.
Your public key has been saved in gh-pages.pub.
The key fingerprint is:
SHA256:qm3ZlWXDEwfSwxmqMIQAzcDypK3lieShzOY4J7CYd20 redgaojila@gmail.com
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

### 配置源码仓库和发布仓库密钥

![](https://raw.githubusercontent.com/gaojila/images/master/travis-ci转到github-action/Snipaste_2021-01-29_15-29-59.png)


