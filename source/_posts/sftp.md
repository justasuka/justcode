---
title: sftp
date: 2021-10-20 17:56:41
tags: ['ssh']
cover: https://s3.bmp.ovh/imgs/2021/10/52fd491754edacb4.png
---

### ssh密钥在这里的作用

1. 配置本机

> 在本机上生成的密钥对，公钥放到远程主机，并追加到~/.ssh/authorized_keys 。这样本机就可以访问到远程主机而不需要密码（部分配置需要改动/etc/ssh/sshd_config）

2. 配置多台机器

>怎么样使用一个密钥就能在不同的主机上访问远程主机呢？
>
>在配置GitHub Actions 的时候遇到了这个问题
>
>先在远程主机生成密钥对，公钥追加到~/.ssh/authorized_keys 。用私钥在本机上做认证
>
>（这样只需要拿到一个私钥应该可以在不同的主机上访问到远程主机了maybe）

~~仅个人经历，未查证，疏漏待指出~~



附上GitHub Actions的workflow

~~~yaml
name: Hexo Deploy

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-18.04
    if: github.event.repository.owner.id == github.event.sender.id

    steps:
      - name: Checkout source
        uses: actions/checkout@v2
        with:
          ref: main

      - name: Setup Node.js
        uses: actions/setup-node@v1
        with:
          node-version: '12'

      - name: Setup Hexo
        env:
          ACTION_DEPLOY_KEY: ${{ secrets.HEXO_DEPLOY_KEY }}
        run: |
          mkdir -p ~/.ssh/
          echo "$ACTION_DEPLOY_KEY" > ~/.ssh/id_rsa
          chmod 700 ~/.ssh
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts
          git config --global user.email "848494675@qq.com"
          git config --global user.name "reol"
          npm install hexo-cli -g
          npm install

      - name: Deploy
        run: |
          hexo clean
          hexo deploy

      - name: 📂Sync files
        # uses: swillner/sftp-sync-action@v1.0 
        uses: wlixcc/SFTP-Deploy-Action@v1.0 
        with:
          server: '${{ secrets.FTP_SERVER }}'
          username: 'yours'
          ssh_private_key: ${{ secrets.SSH_PRIVATE_KEY }}
          local_path: 'yours' 
          remote_path: 'yours'
~~~

在git push后打包并使用hexo deploy 根据里面配置的地址，以及两个仓库配置的密钥将，自动打包后的build文件夹下的内容推送到GitHub pages 仓库

然后使用sftp同步到远程主机，因为使用的是GitHub提供的服务器打包sftp的所以得使用远程主机生成的密钥对中的私钥，放在settings里的secrets里面，才能认证。



其中参考的[GitHub Actions](https://github.com/marketplace/actions/sftp-deploy) 地址

主要参考的[文章](https://zhuanlan.zhihu.com/p/107545396)

密钥操作所参考的[文章](https://www.cnblogs.com/yhaing/p/7921666.html)

Permission denied (publickey). [解决方法](https://www.cnblogs.com/guodavid/p/11004499.html)



