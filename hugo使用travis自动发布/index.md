# Hugo使用travis自动发布


## 背景

从`hexo`迁移到`hugo`后，发布博客开始变的繁琐，没有`hexo -d`这样的快捷部署，但是好在有`travis`这样的免费`CI`平台，在使用`travis`来部署博客的确快捷了很多，只需要发布源码即可。

## Github获取token

![20200605161952-image.png](https://raw.githubusercontent.com/gaojila/images/master/yosoro/20200605161952-image.png)

![20200605162023-image.png](https://raw.githubusercontent.com/gaojila/images/master/yosoro/20200605162023-image.png)

![20200605162123-image.png](https://raw.githubusercontent.com/gaojila/images/master/yosoro/20200605162123-image.png)

记下 `Token` 的值 (一定要记下来，因为离开这个页面之后就没有机会再次查看了)

![20200605162340-image.png](https://raw.githubusercontent.com/gaojila/images/master/yosoro/20200605162340-image.png)

## 设置Travis CI

使用`github`帐号注册一个`travis`帐号，登录在`hugo`仓库上打上，然后再点击`setting`![20200605163532-image.png](https://raw.githubusercontent.com/gaojila/images/master/yosoro/20200605163532-image.png)

然后填写 **Environment Variables**。

- **`Name`** 填写： `GITHUB_TOKEN`
- **`Value`** 填写：刚刚在 GitHub 申请到的 Token 的值

![20200605163827-image.png](https://raw.githubusercontent.com/gaojila/images/master/yosoro/20200605163827-image.png)

点击`add`

## 编写.trabis.yml

```yaml
language: go

go:
  - "1.8"  # 指定Golang 1.8

install:
  # 安装最新的hugo
  - wget https://github.com/gohugoio/hugo/releases/download/v0.71.1/hugo_0.71.1_Linux-64bit.deb
  - sudo dpkg -i hugo*.deb

script:
  # 运行hugo命令
  - hugo

after_script:
  # 部署
  - cd ./public
  - git init
  - git config user.name "[gaojila]"
  - git config user.email "[redgaojila@gmail.com]"
  - git add .
  - git commit -m "Update Blog By TravisCI With Build $TRAVIS_BUILD_NUMBER"
  # Github Pages
  - git push --force --quiet "https://$GITHUB_TOKEN@${GH_REF}" master:master
  # Github Pages
  - git push --quiet "https://$GITHUB_TOKEN@${GH_REF}" master:master --tags

env:
  global:
   # Github Pages
    - GH_REF: "github.com/gaojila/gaojila.github.io"

deploy:
  provider: pages # 重要，指定这是一份github pages的部署配置
  skip-cleanup: true # 重要，不能省略
  local-dir: public # 静态站点文件所在目录
  # target-branch: master # 要将静态站点文件发布到哪个分支
  github-token: $GITHUB_TOKEN # 重要，$GITHUB_TOKEN是变量，需要在GitHub上申请、再到配置到Travis
  # fqdn:  # 如果是自定义域名，此处要填
  keep-history: true # 是否保持target-branch分支的提交记录
  on:
    branch: master # 博客源码的分支

```

将上面的配置文件按照你的实际情况更改。

然后将代码提交到 **hugo 仓库** 里。等个一两分钟，就可以在 [Travis CI](https://travis-ci.org/) 上查看部署情况了

<font color=green>绿色</font> 代表部署成功  <font color=yellow>黄色</font>代表正在部署  <font color=red>红色</font> 代表部署失败  <font color=gray>灰色</font> 代表部署被取消

![20200605164520-image.png](https://raw.githubusercontent.com/gaojila/images/master/yosoro/20200605164520-image.png)

## 相关文章

- [使用 Travis CI 自动部署 Hugo 博客](https://mogeko.me/2018/028/)

