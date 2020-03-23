---
title: TravisCi/Gradle部署SpringBoot项目到aliyun ECS
tags:
  - Springboot
  - TravisCI
  - Gradle
originContent: "# 项目新建\n\n新建一个Springboot项目,IDEA/SpringInitializer或者其他方法建立一个项目,将项目上传到Github,因为TravisCI和Github配合食用风味更佳\U0001F601\n\n登录到[TravisCI官网](http://travis-ci.org),可以直接用Github登录,然后给该项目配置TravisCI,网上教程一大把我这就不赘述了,我们主要看下`.travis.yml`文件内容,以及ssh登录到我们的阿里云ECS的实现步骤(主要是为了scp我们的build生成的jar包)\n\n## Worker环境\n先确定travis的worker配置,java语言,jkd8,其他也就不用配置了,默认是ubuntu xenial,没什么特殊要求就用这个了\n```yml\nlanguage: java\njdk:\n  - openjdk8\n\n```\n\n## 目标ECS\n先给ECS绑定一个秘钥对,可以看看阿里云的[文档](https://help.aliyun.com/document_detail/51793.html?spm=a2c4g.11186623.2.12.49d511c8EAOf1z#concept-wy4-th1-ydb),然后先用自己的电脑登陆一次ECS,然后安装下travis-cli工具\n```\nssh -i pem文件路径 用户名@ecs的ip\n```\ntravis-cli需要ruby环境,先用brew安装下gem(ruby的包管理器)\n```\nbrew install ruby@2.4\n```\n再用gem安装travis-cli\n```\ngem install travis \n```\n\n\n接着登录到travis,用你的Github登录就行,\n```\ntravis login\n```\n进入你的项目根目录,运行命令,按提示操作会在你的.travis.yml文件中生成一段openssl命令,并生成一个id_rsa.enc文件\n```\ntravis encrypt-file ~/.ssh/id_rsa --add\n```\n这个文件需要被纳入到版本控制中,travis会通过它解密出必要的ssh身份信息\n\n\n## 基本成果\n\n```yml\nlanguage: java\nbranches:\n  only:\n    - master\njdk:\n  - openjdk8\naddons:\n  ssh_known_hosts: \"$SERVER_IP:$SSH_PORT\"\nbefore_cache:\n  - rm -f  $HOME/.gradle/caches/modules-2/modules-2.lock\n  - rm -fr $HOME/.gradle/caches/*/plugin-resolution/\ncache:\n  directories:\n    - \"$HOME/.gradle/caches/\"\n    - \"$HOME/.gradle/wrapper/\"\nbefore_install:\n  - openssl aes-256-cbc -K $encrypted_c1911571b64a_key -iv $encrypted_c1911571b64a_iv\n    -in id_rsa.enc -out ~/.ssh/id_rsa -d\n  - chmod 600 ~/.ssh/id_rsa\n  - echo -e \"Host $SERVER_IP\\n\\tStrictHostKeyChecking no\\n\" >> ~/.ssh/config\n  - chmod +x ./gradlew\ninstall:\n  - \"./gradlew assemble\"\nafter_success:\n  - echo $PATH\n  - scp -p $SSH_PORT build/libs/demo-0.0.1-SNAPSHOT.jar root@$SERVER_IP:/tmp/\n\n```\n\n\n"
categories:
  - 笔记
toc: false
date: 2020-03-23 21:47:45
---

# 项目新建

新建一个Springboot项目,IDEA/SpringInitializer或者其他方法建立一个项目,将项目上传到Github,因为TravisCI和Github配合食用风味更佳😁

登录到[TravisCI官网](http://travis-ci.org),可以直接用Github登录,然后给该项目配置TravisCI,网上教程一大把我这就不赘述了,我们主要看下`.travis.yml`文件内容,以及ssh登录到我们的阿里云ECS的实现步骤(主要是为了scp我们的build生成的jar包)

## Worker环境
先确定travis的worker配置,java语言,jkd8,其他也就不用配置了,默认是ubuntu xenial,没什么特殊要求就用这个了
```yml
language: java
jdk:
  - openjdk8

```

## 目标ECS
先给ECS绑定一个秘钥对,可以看看阿里云的[文档](https://help.aliyun.com/document_detail/51793.html?spm=a2c4g.11186623.2.12.49d511c8EAOf1z#concept-wy4-th1-ydb),然后先用自己的电脑登陆一次ECS,然后安装下travis-cli工具
```
ssh -i pem文件路径 用户名@ecs的ip
```
travis-cli需要ruby环境,先用brew安装下gem(ruby的包管理器)
```
brew install ruby@2.4
```
再用gem安装travis-cli
```
gem install travis 
```


接着登录到travis,用你的Github登录就行,
```
travis login
```
进入你的项目根目录,运行命令,按提示操作会在你的.travis.yml文件中生成一段openssl命令,并生成一个id_rsa.enc文件
```
travis encrypt-file ~/.ssh/id_rsa --add
```
这个文件需要被纳入到版本控制中,travis会通过它解密出必要的ssh身份信息


## 基本成果

```yml
language: java
branches:
  only:
    - master
jdk:
  - openjdk8
addons:
  ssh_known_hosts: "$SERVER_IP:$SSH_PORT"
before_cache:
  - rm -f  $HOME/.gradle/caches/modules-2/modules-2.lock
  - rm -fr $HOME/.gradle/caches/*/plugin-resolution/
cache:
  directories:
    - "$HOME/.gradle/caches/"
    - "$HOME/.gradle/wrapper/"
before_install:
  - openssl aes-256-cbc -K $encrypted_c1911571b64a_key -iv $encrypted_c1911571b64a_iv
    -in id_rsa.enc -out ~/.ssh/id_rsa -d
  - chmod 600 ~/.ssh/id_rsa
  - echo -e "Host $SERVER_IP\n\tStrictHostKeyChecking no\n" >> ~/.ssh/config
  - chmod +x ./gradlew
install:
  - "./gradlew assemble"
after_success:
  - echo $PATH
  - scp -p $SSH_PORT build/libs/demo-0.0.1-SNAPSHOT.jar root@$SERVER_IP:/tmp/

```


