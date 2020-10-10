---
title: hexo博客的备份和恢复
date: 2019-07-07 13:26:09
tags:
- hexo
---

1.先把github上的博客repo克隆下来

2.设置两个分支分别为`master`和`hexo`，`hexo`用来存放博客的配置文件， 主题文件等等

3.配置`hexo`分支中的`_config.yml`文件，修改如下，把hexo生成的静态页面存放到master分支
```
deploy:
  type: git
  repo: git@github.com:username/blog.github.io.git
  branch: master
```
4.切换到`hexo`分支，依次执行命令，不要执行 **npm install init**
```
npm install hexo
npm install
sudo npm install hexp-cli -g
npm install hexo-deployer-git --save
```

5.生成内容以及发布
```
hexo new "blog title"
git add .
git commit -m "something"
git push origin hexo

hexo g -d
```