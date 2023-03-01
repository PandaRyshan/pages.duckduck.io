---
title: "使用 Jekyll 部署 Github Pages"
author: aold619
date: 2022-08-25 22:00 +0800
categories: [Tutorial]
tags: [tutorial, github pages]
---

## Jekyll 是什么

[Jekyll](https://jekyllrb.com/docs/) 是一个静态网站生成器，支持常用的标记语言，
如 markdown。因此可以迅速建立网站并专注内容。本篇文章旨在介绍 jekyll 的安装，
并将 jekyll 部署到 github pages。

## 1. 安装环境

### 1.1 本地安装

> 环境是可选项，旨在从本地预览，如果已有完整的 jekyll 文件结构，可以直接上传项目
并部署到 github pages。
{: .prompt-tip}

根据不同操作系统跟随官网的 [Guide](https://jekyllrb.com/docs/)进行操作。

在这个过程中会安装 Ruby 语言环境, RubyGems, Bundler, Jekyll 几个软件。

   * RubyGems: Ruby 程序包管理器，类似于 Python 的 Pip；
   * Bundler: 负责包依赖解析；

### 1.2 使用 Docker

[官方 docker 镜像](https://hub.docker.com/r/jekyll/jekyll) 和[使用文档](https://
github.com/envygeeks/jekyll-docker/blob/master/README.md)。

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

   > 最新稳定版是 4, 也可以使用 stable 或 latest 标签
   {: .prompt-tip}

   ```shell
export JEKYLL_VERSION=3.8
docker run --rm \
  --volume="$PWD:/srv/jekyll:Z" \
  -it jekyll/jekyll:$JEKYLL_VERSION \
  jekyll build
   ```

   > 可以使用 `--volume="$PWD/vendor/bundle:/usr/local/bundle:Z" \` 开启构建缓存

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

示例 devbox.json：

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
        "jekyll new myblog && cd myblog",
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

   1. 新建一个目录并使用 `devbox init` 初始化一个配置文件，使用 `devbox add <package_name>` 
   添加 `ruby_3_1`，`bundler`，`libffi`。或使用示例 devbox.json 配置：

   2. 使用 `devbox shell` 切换到新环境. 如使用示例配置, 可使用 `devbox run generate` 
   通过脚本生成名为 myblog 的网站, 并使用 `devbox run serve` 来开启本地 http 服务。

   3. 访问 [http://localhost:4000](http://localhost:4000) 查看页面。

## 2. 安装主题

jekyll 官网的 resources 链接：[Resources](https://jekyllrb.com/resources/)，
其中最后一个站点发布的是付费主题。

找到心仪的主题后进入主题的 github 页面，大多都会在 README 内有详细的说明。
按着流程走就行。下面会讲一下多数主题的两种安装方式。

1. 本地安装

* 下载作者的 repo 或 release，并基于此修改自己的配置进行部署。这种方式通常 Gem 依赖等所需的
  文件都是已经准备好的，只需要少数配置改动，如填写自己的 pages 地址即可。缺点是不方便拉取最新
  的主题变更，未来有可能需要使用 github 提供的 compare 工具比照改动。

2. 启用远程主题插件

这种方式适用于多数主题。只需要修改 jekyll 的 `Gemfile` 和 `_config.yml` 即可。

  * 在 Gemfile 中增加 `gem "jekyll-remote-theme"`
  * 在 `_config.yml` 中的 `plugins` 中启用 `jekyll-remote-theme`
  ```yaml
plugins:
  - jekyll-remote-theme
  ```
  * 在 `_config.yml` 中加载远程主题
  ```yaml
remote_theme: <user>/<theme-repo-name>
  ```

## 3. 安装插件

TODO

## 4. 配置 github pages

1. 在本地 jekyll 目录中初始化 git 并添加自己 repo 的 remote url，push

2. 在 github 的 repo 的页面中找到 `Settings`，点左侧的 `Pages`

  * 更改 `Build and deployment` 为 Github Actions.
  * 更改 Custom domain 为自己的自定义域名, 不填则默认为 `<your_username>.github.io`.
  * 如需自定义域名, 需要点 github 账号头像找到 settings 中 page 选项, 根据提示把验证用的 txt 
  记录及访问路径的 A 记录添加到域名托管方的 DNS 设置中

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
