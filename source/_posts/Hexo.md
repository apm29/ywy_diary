---
title: Hexo
tags:
  - Tools
  - NPM
  - travisci
categories:
  - 笔记
toc: false
date: 2019-12-19 17:16:54
---

# Hexo,Github/TravisCi and AWS

## 目的
使用TravisCI把Hexo博客项目自动提交到github pages 和 AWS EC2服务器

## 1.建立Hexo项目
具体参考Hexo文档建立一个自己的博客项目,提交到Github

## 2.配置TravisCI

首先用Github账户登陆[TravisCI官网](https://travis-ci.org/)

然后将项目打开集成开关,在settings中设置具体参数
![image.png](/images/2019/12/18/3adad000-218f-11ea-9a07-ffc81ee08653.png)

在Github中[DeveloperSettings/PersonalAccessToken](https://github.com/settings/tokens)生成一个新的token(注意给到Token必要的权限),设置为GH_TOKEN,再新加入两个环境变量,一个是项目的Git地址,一个是GithubPage的Git地址
![image.png](/images/2019/12/18/8bd34460-218f-11ea-9a07-ffc81ee08653.png)

在项目根目录下新建一个`.travis.yml`文件

```
# 使用语言
language: node_js
# node版本
node_js: stable
# 设置只监听哪个分支
branches:
  only:
  - master
# 缓存，可以节省集成的时间，这里我用了yarn，如果不用可以删除
cache:
  directories:
    - node_modules
# tarvis生命周期执行顺序详见官网文档
before_install:
# 刚开始是没有这几行的,这是使用travis生成的本地ssh密匙对(登录AWS服务器时有用)
- openssl aes-256-cbc -K $encrypted_771eb07e0b57_key -iv $encrypted_771eb07e0b57_iv
  -in id_rsa.enc -out ~/.ssh/id_rsa -d
- chmod 600 ~/.ssh/id_rsa
- echo -e "Host 54.248.204.0\n\tStrictHostKeyChecking no\n" >> ~/.ssh/config
- npm install -g hexo-cli
install:
# 不用yarn的话这里改成 npm i 即可
- npm install
script:
#- git clone https://github.com/Shen-Yu/hexo-theme-ayer.git themes/ayer
- hexo clean
- hexo generate
after_success:
# 推送到aws服务器的Nginx文件夹
# - scp -o stricthostkeychecking=no -r ./public ubuntu@ec2-54-248-204-0.ap-northeast-1.compute.amazonaws.com:/home/
- cd ./public
- ls
- git init
- git add --all .
- git commit -m "Travis CI Auto Builder"
# 这里的 REPO_TOKEN 即之前在 travis 项目的环境变量里添加的
- git push --quiet --force https://$GH_TOKEN@$GH_REF master:pages
- git push --quiet --force https://$GH_TOKEN@$GH_PAGE_REF master:master
```
上传到服务器需要的准备

我使用的方法是先把自己本地\~/.ssh/id_rsa.pub内容添加到服务器的\~/.ssh/authorized_keys,也就是吧这个ssh公钥加入服务器的信任列表,那样本机登录就不要登录密码了,但是如何让TravisCI的Worker机器登录呢?

首先安装一个travis-cli工具
```
# 安装travis命令行工具，如无法使用gem指令须先安装ruby
gem install travis
# --auto自动登录github帐号
travis login --auto
# 此处的--add参数表示自动添加脚本到.travis.yml文件中
travis encrypt-file ~/.ssh/id_rsa --add
# 这个命令会自动把 id_rsa 加密传送到 .git 指定的仓库对应的 travis 中去
```
执行完以后会发现在travis网站项目里面的环境变量里多了两个参数,项目的.travis.yml文件也多了两行
```
- openssl aes-256-cbc -K $encrypted_771eb07e0b57_key -iv $encrypted_771eb07e0b57_iv
  -in id_rsa.enc -out ~\/.ssh/id_rsa -d
```

再加入如下几行,修改Worker机器的id_rsa文件权限,去掉ssh登录的主机检查
```
- chmod 600 ~/.ssh/id_rsa
- echo -e "Host 主机IP地址\n\tStrictHostKeyChecking no\n" >> ~/.ssh/config
```

最后可以用scp命令吧public目录传到服务器了
```
- scp -o stricthostkeychecking=no -P $PORT -r public/* 用户@域名:/路径
```