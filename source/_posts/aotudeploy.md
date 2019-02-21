---
title: 一个简易的前端自动化构建例子
# date: 2019-02-19 20:33:26      #用命令会自动生成，也可以自己写，所以文章时间可以改
categories: 自动化工具         #文章的分类，这个可以自己定义
tags: [前端,自动化构建]        #tag，为文章添加标签，方便搜索
---

## 写在前面

前阵子一直在赶项目，写需求，沉淀的东西大多是跟业务相关。团队写需求时一般使用 athena + gitlab CI 自动化生成模板，自动化检测代码格式，自动化打包和自动化部署到服务器，全自动一条龙服务。这对于之前都是单枪匹马的我来说，是套极具诱惑力的流程工具。趁着现在没什么需求，我便借着搭建博客的例子来学习如何搭建简易版的自动化构建工具。同时搭建好的博客也更督促了自己 2019 年多点沉淀，多点输出，多发博客。

## 从哪里入手

作为一个简单的示例，示例所要实现的核心功能是 -- 当把代码 push 到 github 远程仓库时，自动打包代码并将部署到服务器生成博客。
1. 申请服务器，寻找博客代码的承载体。
    我们新建好一个 github 远程仓库，并将代码托管到仓库的 gh-pages 分支，之后 github 就会帮我们生成静态网页了。因此我们可以省下申请服务器，申请域名，解析域名，搭建服务器等一系列操作了。这也是为了简化我们的操作，毕竟我们的重心是自动化构建流程。
2. 利用 hexo 搭建好博客框架
3. 添加 Travis CI 增加自动化打包和部署功能

## 实操流程

### 1. 安装 hexo-cli

`npm install -g hexo-cli`

### 2. 安装完成后，执行以下命令，hexo 将会在指定文件夹中新建所需要的文件

```shell
    hexo init <folder>
    cd <folder>
    git init
    git remote add <仓库地址>
    npm install
```

### 3. 新建完成后，指定文件夹的目录如下：

```tree
    ├── _config.yml
    ├── package.json
    ├── scaffolds
    ├── source
    |   └── _posts
    └── themes
```

### 4. 配置

这里要注意的是 url 处，需要填上生成博客的地址，我的是 https://cssie.github.io/cssieblog ，如果是二级地址，在下面的 root 选项处需要填上 /cssieblog/ 。一般情况下，生成博客的地址可以从这个项目的远程仓库地址获得。

 例如：

 我的项目地址：https://github.com/cssie/cssieblog.git
 对应的博客地址：https://cssie.github.io/cssieblog

 *生成的博客地址可以在此处查找到：github 远程仓库 >> Settings >> GitHub Pages*

```yml
    # URL
    ## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
    url: https://cssie.github.io/cssieblog
    root: /cssieblog/
    permalink: :year/:month/:day/:title/
    permalink_defaults:
    ...
    ...
    ## 部署博客，在 repo 处填写好远程仓库地址，并将 branch 设置成 gh-pages 表示代码将发布到此分支上。
    deploy:
    type: git
    repo: https://github.com/cssie/cssieblog.git
    branch: gh-pages #published
    message:
```

### 5. 试运行

常用的 hexo 命令行

```shell
    Usage: hexo <command>

    Commands:
    clean     清除打包文件和缓存
    deploy    部署
    generate  打包静态资源
    init      创建一个新的 hexo 文件夹
```

接下来我们执行以下命令进行打包

`hexo g / hexo generate`

这个过程中有可能会报错，提示某个插件没有安装,如下

`ERROR Plugin load failed: hexo-renderer-marked`

解决方法：执行以下命令查看哪些插件没有安装成功，之后逐一安装缺失的包即可

`npm ls --depth 0`
`npm install hexo-renderer-marked --save`

打包之后在发布前，我们需要先安装 hexo-deployer-git, 并确保在 _config.yml 中 deploy 的设置为 type: git

```js
npm install hexo-deployer-git --save
...
hexo d / hexo deploy
```

至此，搭建服务器就已经大功告成啦。当然，别忘了去远程仓库查看 gh-pages 分支是否已更新代码了。
![结果图](https://img14.360buyimg.com/ling/jfs/t1/18594/12/7856/275573/5c6e5d03E9e72a835/7833011da0df6d83.jpg)

### 6. 添加自动化部署功能

跟到这一步的小伙伴可能很困惑。远程仓库的 master 主支竟然是空的！其实上面几步我们只是把代码部署到远程仓库的 gh-pages 分支上，并没有上传主要代码到 master 哦。那自然而然的一种想法是：那我把主代码再 push 一次到 master 就好了。但这样子每次更新本地代码后，都需要俩个步骤：1，hexo deploy 将打包好的代码部署到 gh-pages；2，git 提交源代码到 master。
那可不可以把这俩步合并一下呢？答案是可以的。

#### 添加 Travis CI 持续集成服务

> Travis CI 提供的是持续集成服务（Continuous Integration，简称 CI）。它绑定 Github 上面的项目，只要有新的代码，就会自动抓取。然后，提供一个运行环境，执行测试，完成构建，还能部署到服务器。
持续集成的好处在于，每次代码的小幅变更，就能看到运行结果，从而不断累积小的变更，而不是在开发周期结束时，一下子合并一大块代码。

1. 访问 [Travis CI 官网](https://travis-ci.org/)，使用 Github 账户登入 Travis CI，点击右上角的个人头像 >> settting。Travis 会列出 Github 上面你的所有仓库，以及你所属于的组织。此时，选择你需要 Travis 帮你构建的仓库，打开仓库旁边的开关。一旦激活了一个仓库，Travis 会监听这个仓库的所有变化。
2. github 和 Travis 配置

- 进入 [github](https://github.com/) 首页，点击右上角头像 >> settings >> Developer settings >> Personal access tokens >> 点击 Generate new token 按钮 >> 点击全选 repo选项和 user 选项 >> 点击 Generate token 按钮以生成 Travis 需要的 access token。
***注意：复制下生成的token（只允许看见一次），在Travis那边可以使用***
- 进入 [Travis CI 官网](https://travis-ci.org/),点击 激活仓库开关 右边的 settings，在 Environment Variables 选项中添加一个变量 HEXO_TOKEN，并将刚刚复制的 token 填进去，点击 add 按钮。
到此 github 和 Travis 的配置就 ok 了。

3. 在项目根目录下，创建配置文件，并命名为 <font color=ff0000>.travis.yml</font>。一旦代码仓库有新的 Commit，Travis 就会去找这个文件，执行里面的命令。

4. .travis.yml 文件配置

```yml
    language: node_js
    node_js: stable
    cache:
    directories:
    - node_modules
    install:
    - npm install
    script:
    - hexo clean
    - hexo g
    after_script:
    - git clone https://${GH_REF} .deploy_git
    - cd .deploy_git
    - git checkout gh-pages
    - cd ../
    - mv .deploy_git/.git/ ./public/
    - cd ./public
    - git config user.name "cssie"
    - git config user.email <youremail>
    - git add .
    - git commit -m "Travis CI Auto Builder at `date +"%Y-%m-%d %H:%M"`"
    - git push --force --quiet "https://${HEXO_TOKEN}@${GH_REF}"
    branches:
    only:
    - master
    env:
    global:
    - GH_REF: github.com/cssie/cssieblog.git
    notifications:
    email:
    - <youremail>
    on_success: change
    on_failure: always
```

5. git 提交文件到 github 上，在 Travis CI 官网左边处点击项目，看到以下截图，就说明 Travis CI 已正常运行。这时只要刷新博客地址：https://cssie.github.io/cssieblog/ 即可看到页面内容已刷新（一般会有点延迟时间），同时到远程仓库地址 https://github.com/cssie/cssieblog.git 处可以看到 master 和 gh-pages 均已更新代码。
![成功截图](https://img10.360buyimg.com/ling/jfs/t1/7239/38/15203/462920/5c6e693fE4f4a4ad0/8637f24a392cd503.jpg)

## 拓展

到这里我们已经完成了一开始的诉求，然而这还远远只是一个开头。通过这个例子我们还可以去拓展很多，譬如如果我们想把博客搭建在自己的服务器上，我们应该如何去配置 Travis ，让其在部署的时候，同时部署到 gh-pages 和自己的服务器。再如目前博客是没有评论功能的，如何去添加评论功能。又如或许我们可以在打包代码前先检测下代码格式是否正确等等。

## 参考文章

[Hexo 官方文档](https://hexo.io/zh-cn/docs/)
[持续集成服务 Travis CI 教程](http://www.ruanyifeng.com/blog/2017/12/travis_ci_tutorial.html)
[Travis CI 持续部署Hexo博客到GitHub Page和VPS服务器](http://www.yanglangjing.com/2018/08/28/travis_ci_auto_deploy_hexo_to_vps/)
[搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)