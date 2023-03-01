---
title: "使用 Jekyll 部署 Github Pages"
author: aold619
date: 2022-08-25 22:00 +0800
categories: [Tutorial]
tags: [tutorial, github pages]
---

## Jekyll 是什么

[Jekyll](https://jekyllrb.com/docs/) 是一个静态网站生成器, 支持常用的标记语言, 
如 markdown. 因此可以迅速建立网站并专注内容. 本篇文章旨在介绍 jekyll 的安装, 
并将 jekyll 部署到 github pages.

## 1. 安装环境

### 1.1 本地安装

> 环境是可选项, 旨在从本地预览, 如果已有完整的 jekyll 文件结构, 可以直接上传项目
并部署到 github pages.
{: .prompt-tip}

根据不同操作系统跟随官网的 [Guide](https://jekyllrb.com/docs/)进行操作.

在这个过程中会安装 Ruby 语言环境, RubyGems, Bundler, Jekyll 几个软件.

   * RubyGems: Ruby 程序包管理器，类似于 Python 的 Pip.
   * Bundler: 负责包依赖解析.

### 1.2 使用 Docker

[官方 docker 镜像](https://hub.docker.com/r/jekyll/jekyll) 和[使用文档](https://
github.com/envygeeks/jekyll-docker/blob/master/README.md).

主要使用方法:

   1. 初始化一个网站目录

   ```shell
export site_name="my-blog" && export MSYS_NO_PATHCONV=1
docker run --rm \
  --volume="$PWD:/srv/jekyll" \
  -it jekyll/jekyll \
  sh -c "chown -R jekyll /usr/gem/ && jekyll new $site_name" \
  && cd $site_name
   ```

   2. 构建

   > 最新稳定版是 4, 也可以使用 stable 标签
   {: .prompt-tip}
   > 可以使用 `--volume="$PWD/vendor/bundle:/usr/local/bundle:Z" \` 开启构建缓存

   ```shell
export JEKYLL_VERSION=3.8
docker run --rm \
  --volume="$PWD:/srv/jekyll:Z" \
  -it jekyll/jekyll:$JEKYLL_VERSION \
  jekyll build
   ```

   3. 为项目的 Gemfile 添加 webrick

   > [Ruby 3.0 后需要添加 webrick](https://jekyllrb.com/docs/)
   {: .prompt-tip}

   ```shell
docker run --rm \
  --volume="$PWD:/srv/jekyll:Z" \
  -it jekyll/builder:$JEKYLL_VERSION \
  bundle add webrick
   ```

   4. 开启服务

   ```shell
docker run --rm \
  --volume="$PWD:/srv/jekyll:Z" \
  --publish [::1]:4000:4000 \
  jekyll/jekyll \
  jekyll serve
   ```

   5. 访问 [http://localhost:4000](http://localhost:4000) 查看页面

### 1.3 使用 [Devbox](https://www.jetpack.io/devbox/docs/devbox_examples/stacks/jekyll/)

   1. 新建一个目录并使用 `devbox init` 初始化一个配置文件, 使用 `devbox add 
   <package_name>` 添加 `ruby_3_1`, `bundler`, `libffi`. 或使用示例 devbox.json 配置:

   ```json
   {
     "packages": [
       "ruby_3_1",
       "bundler",
       "libffi"
     ],
     "shell": {
       "init_hook": [],
       "scripts": {
         "generate": [
           "gem install jekyll --no-document",
           "gem new myblog && cd myblog",
           "bundle add webrick",
           "bundle update",
           "bundle lock",
           "bundle package",
           "rm -rf vendor"
         ],
         "serve": [
           "cd myblog",
           "bundler exec $GEM_HOME/bin/jekyll serve --trace"
         ]
       }
     },
     "nixpkgs": {
       "commit": "f80ac848e3d6f0c12c52758c0f25c10c97ca3b62"
     }
   }
   ```

   2. 使用 `devbox shell` 切换到新环境. 如使用示例配置, 可使用 `devbox run generate` 
   通过脚本生成名为 myblog 的网站, 并使用 `devbox run serve` 来开启本地 http 服务.

   3. 访问 [http://localhost:4000](http://localhost:4000) 查看页面

## 2. 安装主题

jekyll 官网的 resources 链接: [Resources](https://jekyllrb.com/resources/)

在 theme 链接中寻找喜欢的主题，根据 github repo 的说明进行安装。

多数主题只需要修改 jekyll 的 `Gemfile` 和 `_config.yml` 即可，也有像我使用的 
[chirpy theme](https://github.com/cotes2020/jekyll-theme-chirpy/) 主题这样，
repo 中已经包含了 jekyll 的所有文件，直接从主题的 repo 生成自己的 repo 即可。

其中在 `_config.yml` 中，关于 github pages 的地址只需要配置在 `url` 里即可，
`baseurl` 是指 url 中的 sub path，不用填写的。

## 3. 安装插件

TODO

## 4. 配置 github pages

1. 在本地 jekyll 目录中初始化 git 并添加自己 repo 的 remote url，push

2. 在 github 的 repo 的页面中找到 `Settings`，点左侧的 `Pages`

  * 更改 `Build and deployment` 为 Github Actions，repo 会提供一个持续部署的 workflow 脚本，直接编辑并提交该文件即可
  * 更改 Custom domain 为自己的自定义域名，不填则默认为 github 自己的 `username.github.io`
  * 如果用自己的域名，还需要点自己头像中的 settings 找到 page 根据提示给域名的 DNS 添加 txt 和 A 记录

在 Action 卡片中查看部署情况，没问题的话，访问自己的 github pages 地址，就能看到 blog 了

## 5. 写文章

创建 markdown 文本在 jekyll 下的 _posts 目录下，文件名需要遵循 `yyyy-MM-dd-post-title-name.md`。

md 文件中的 meta 信息示例：

```
---
title: Post Title Name
author: xxx
date: yyyy-MM-dd HH:mm:ss
categories: [c1, c2]
tags: [t1, t2, t3]
---
```
