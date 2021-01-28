# Hugo使用algolia搜索

## 背景

在将hexo迁移到hugo后我就开始折腾博客搜索了，下面的配置正对loveit主题配置

## 开启algolia搜索

在`config.toml`中添加下面字段，`xxxxx`是必填，你可以参考`loveit`主题中的`config.toml`

```toml
      # 搜索配置
      [languages.zh-cn.params.search]
        enable = true
        # 搜索引擎的类型 ("lunr", "algolia")
        type = "algolia"
        # 文章内容最长索引长度
        contentLength = 4000
        # 搜索框的占位提示语
        placeholder = ""
        # 最大结果数目
        maxResultLength = 10
        # 结果内容片段长度
        snippetLength = 50
        # 搜索结果中高亮部分的 HTML 标签
        highlightTag = "em"
        # 是否在搜索索引中使用基于 baseURL 的绝对路径
        absoluteURL = false
        [languages.zh-cn.params.search.algolia]
        # algolia注册的索引名称
          index = "xxxxx"      
        # 在你注册完成后，点击API Keys就能看见下面的参数          
          appID = "xxxxx"
          searchKey = "xxxxx"

```

## 自动提交索引到algolia

又又用到了`npm`,好在集成到`travis`中眼不见为净。

- 在`config.toml`同级目录下运行`npm init`，一路回车即可。

- 修改`npm int`生成的`package.json`添加下面字段
  
  ```json
   "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1",
      "algolia": "atomic-algolia"
    },
  ```

- 在`config.toml`同级目录添加`.env`文件并添加下面字段
  
  ```bash
  ❯ cat .env
  ALGOLIA_APP_ID=U9QMQ70DKL
  ALGOLIA_INDEX_NAME=gaojila.github.io
  ALGOLIA_INDEX_FILE=public/index.json
  ```

- 修改`.travis.yml`文件如下
  
  ```yaml
  language: go
  
  go:
    - "1.8"  # 指定Golang 1.8
  
  install:
    # 安装最新的hugo
    - wget https://github.com/gohugoio/hugo/releases/download/v0.71.1/hugo_0.71.1_Linux-64bit.deb
    - sudo dpkg -i hugo*.deb
    # 安装搜索插件
    - npm install atomic-algolia --save-dev
  
  script:
    # 运行hugo命令
    - hugo
    # 生成索引命令
    - echo "ALGOLIA_ADMIN_KEY=$ALGOLIA_ADMIN_KEY" >> .env
    - npm run algolia
  
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
  
  

- 在`travis`中添加变量`$ALGOLIA_ADMIN_KEY`
  
  ![20200606184854-image.png](https://raw.githubusercontent.com/gaojila/images/master/yosoro/20200606184854-image.png)

## 参考链接

- aligolia的其他配置可以看下面的链接
  
  [dreamsafari.info](https://www.dreamsafari.info/2020/04/hugo-loveit-mod/)
  
  [nashome.cn](https://www.nashome.cn/post/hugo-algolia/)



