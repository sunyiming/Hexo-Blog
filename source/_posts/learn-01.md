---
title: 学习笔记
date: 2019-10-15 15:19:23
tags:
---
其实在很早之前用过 Hexo，但之前一直没有成功配置过。这次好不容易成功配置好了。具体的基础操作在这里就不再赘述了。在这里就总结一下配置过程中我和其他同学的问题。
## Hexo 配置
### 插件安装
以前遇到过编译生成的 HTML 文件为 0 KB 的问题。可能跟缺少生成插件有关，可通过以下命令安装。
```bash
npm install hexo-renderer-scss --save
npm install hexo-deployer-git --save
```
### 你 打 字 不 带 空 格
Hexo 通常以`.yml`为扩展名的 [YMAL](https://zh.wikipedia.org/wiki/YAML) 文件作为配置文件。键值和数据由冒号及**空格**分开，即`key: value`的形式。许多同学在这个地方出了问题。幸好 VS Code 编辑器有语法高亮，能及时发现问题。
## 自定义域名与仓库
Github Pages 默认的创建方式是新建一个`<自定义名称>.github.io`为名的仓库并初始化 README.md 文件。当然你可以不按这个格式来。
先进入你想配置 Pages 服务的仓库。进入仓库设置（Settings）。找到 GitHub Pages 这一节，在 Source 中选定你想配置 Pages 服务的分支（branch）即可。
若需[配置自定义域名](https://help.github.com/en/articles/managing-a-custom-domain-for-your-github-pages-site)，在上述配置下面找到 Custom domain，填入你的域名。然后去你的域名提供商或 DNS 服务商那里修改 **CNAME** 记录为`<你的GitHub用户名>.github.io`，或修改 **A** 记录为以下任意IP
```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```
可以顺便把底下的 Enforce HTTPS 打上勾，这样GitHub会通过让老子加密（Let's Encrypt）自动为你的域名签发 SSL 证书。
## 使用 Travis CI 持续构建
Travis CI 简单来说就是可以让你只把 Markdown 文件推到 GitHub 上，他会自动把你的文章编译为 HTML 文件并部署到 Github Pages 上。我暂时还没弄明白这个比在本地编译完再推送到网站上有什么优点。但 Hexo 官网上都专门有[详细教程](https://hexo.io/zh-cn/docs/github-pages)，所以说可以没事配置上玩玩。
需要注意以下几点：
1. 一般下载 Hexo 主题的时候都会使用`git clone`命令，这会在目录下产生`.git`这个目录。如果将这个目录上传至 GitHub，会被自动识别为仓库嵌套。这会进一步影响到后面的编译问题（通常编译报错为找不到主题模板）。删除子目录中的`.git`目录，重新上传 GitHub 即可。
2. 编译时会覆盖掉 GitHub 中的 CNAME 文件导致自定义域名配置错误。在这里非常感谢隔壁技术部 [SpartaEN](https://github.com/SpartaEN) 大佬提供的[解决方案](https://github.com/SpartaEN/hexo-blog-demo/blob/master/.travis.yml)。将`.travis.yml`文件改为以下内容即可：
```yml
sudo: false
language: node_js
node_js:
  - 10 # use nodejs v10 LTS
cache: npm
branches:
  only:
    - master # build master branch only
script:
  - hexo generate # generate static files
  # Fix CNAME issue
  # - rm public\CNAME
  - echo "你的域名" >> public\CNAME
deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GH_TOKEN
  keep-history: true
  on:
    branch: master
  local-dir: public
  ```
  ## 回滚 Git 版本
  在使用 Git 时不免会遇到失误的提交，这时需要回滚到以前的版本。
  1. 使用`git log`列出之前的 commit 记录，找到你想回退到的版本，复制下其对应的 hash 代码（就像是`f7371d168201b65d68da67d1276c89369fd1fcad`一样的乱码）。
  2. 使用`git reset --hard 刚才复制的代码`回退到之前的版本。
  3. 使用`git push -f -u origin master`推送到远程仓库（注意：因为要覆盖掉远程的版本，必须要加上`-f`这个参数以强制推送）
