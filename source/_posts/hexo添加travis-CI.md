---
title: hexo添加travis CI
date: 2017-12-06 15:25:29
tags:
---

## 前言
多人写博客，操作步骤繁琐。写一篇博客不仅需要推送博客源码到master分支，而且还要部署到gh-pages分支。

所以，为了解决这个痛点。引入Travis CI来做部署这件事情。

## 添加组织授权
如果看不到Organizations，说明还没有授权
![](https://ws1.sinaimg.cn/large/006DVXJ3gy1fm799f6kclj31co13gagz.jpg)

进入到settings下面，申请授权，通过后，在travis就可以看到仓库信息了
![](https://ws1.sinaimg.cn/large/006DVXJ3gy1fm7e8b9enmj30s20i1jue.jpg)

## 创建SSH key
`ssh-keygen -t rsa -C "youremail@example.com"`

## 加入deploy keys
将上一步生成的id_rsa.pub，填入到 repo的 Deploy key
![](http://ww1.sinaimg.cn/large/006DVXJ3gy1fm7emvzpeuj30sb0bhdhk.jpg)
记得要将 Allow write access 的选项选上，这样 Travis CI 才能获得 push 代码的权限

## 加密私钥
`gem install travis`
`travis login --auto`
`travis encrypt-file id_rsa --add`
会生成加密之后的秘钥文件 id_rsa.enc，原来的文件 id_rsa 就可以删掉了

## SSH Config
为了上git默认连接ssh
创建一个.travis文件，放置刚才生成的id_rsa.enc
并且新建ssh_config文件，写入下面内容
```
Host github.com
    User git
    StrictHostKeyChecking no
    IdentityFile ~/.ssh/id_rsa
    IdentitiesOnly yes
```

## 编写travis配置文件
```bash
language: node_js
node_js: stable

# 只监听 master 分支的改动
branches:
  only:
  - master

# 缓存依赖，节省持续集成时间
cache:
  npm: true
  directories:
    - node_modules


before_install:
  - openssl aes-256-cbc -K $encrypted_3031c5337047_key -iv $encrypted_3031c5337047_iv -in .travis/id_rsa_repo.pub.enc -out ~/.ssh/id_rsa -d
  # 改变文件权限
  - chmod 600 ~/.ssh/id_rsa
  # 配置 ssh
  - eval $(ssh-agent)
  - ssh-add ~/.ssh/id_rsa
  - cp .travis/ssh_config ~/.ssh/config
  # 配置 git 替换为自己的信息
  - git config --global user.name 'Travis'
  - git config --global user.email Travis@hemayun.com

install:
  - npm install hexo-cli -g
  - npm install

script:
  # 生成静态页面
  - hexo clean
  - hexo g -d
```
现在只要向项目 push 代码就可以触发travis部署了
进入https://travis-ci.org就可以看到部署的过程了。
并且build通过会有邮件通知。

## 最后
.travis.yml 的完整代码可以看我的 .travis.yml 文件，完整的代码看这里<https://github.com/Hema-FE/hema-fe-blog/blob/master/.travis.yml>

## 参考文章
1. 用 Travis CI 自动部署 hexo <http://blog.acwong.org/2016/03/20/auto-deploy-hexo-with-travis-CI/>