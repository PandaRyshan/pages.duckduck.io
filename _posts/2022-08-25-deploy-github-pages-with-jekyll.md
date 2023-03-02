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

根据不同操作系统跟随官网的 [Guide](https://jekyllrb.com/docs/) 进行操作。

在这个过程中会安装 Ruby 语言环境, RubyGems, Bundler, Jekyll 几个软件。

* RubyGems: Ruby 程序包管理器，类似于 Python 的 Pip；
* Bundler: 负责包依赖解析；

### 1.2 使用 Docker

[官方 docker 镜像](https://hub.docker.com/r/jekyll/jekyll) 和
[使用文档](https://github.com/envygeeks/jekyll-docker/blob/master/README.md) 。

主要使用方法:

1. 初始化一个网站目录

   ```bash
   export site_name="my-blog" && export MSYS_NO_PATHCONV=1
   docker run --rm \
     --volume="$PWD:/srv/jekyll" \
     -it jekyll/jekyll \
     sh -c "chown -R jekyll /usr/gem/ && jekyll new $site_name" \
     && cd $site_name
   ```

2. 构建

   ```bash
   export JEKYLL_VERSION=3.8
   docker run --rm \
     --volume="$PWD:/srv/jekyll:Z" \
     -it jekyll/jekyll:$JEKYLL_VERSION \
     jekyll build
   ```

   > 最新稳定版是 4，也可以使用 stable 或 latest 标签。
   {: .prompt-tip}
   > 可以使用 `--volume="$PWD/vendor/bundle:/usr/local/bundle:Z" \` 开启构建缓存。

3. 为项目的 Gemfile 添加 webrick

   > [Ruby 3.0 后需要添加 webrick](https://jekyllrb.com/docs/)
   {: .prompt-tip}

   ```bash
   docker run --rm \
     --volume="$PWD:/srv/jekyll:Z" \
     -it jekyll/builder:$JEKYLL_VERSION \
     bundle add webrick
   ```

4. 开启服务

   ```bash
   docker run --rm \
     --volume="$PWD:/srv/jekyll:Z" \
     --publish [::1]:4000:4000 \
     jekyll/jekyll \
     jekyll serve
   ```

5. 访问 <http://localhost:4000> 查看页面

### 1.3 使用 [Devbox](https://www.jetpack.io/devbox/docs/devbox_examples/stacks/jekyll/)

   1. 新建一个目录并使用 `devbox init` 初始化配置文件 `devbox.json`，
      使用 `devbox add <package_name>`
      添加 `ruby_3_1`，`bundler`，`libffi`。或使用示例配置：

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

      > * 示例中包含 generate 和 serve 两个脚本
      > * 可在 devbox 环境下使用 `devbox run <script-name>` 运行脚本
      > * generate 脚本用来安装 jekyll 并生成名为 myblog 的工程目录
      > * serve 脚本用来启动 http；
          `bundler exec jekyll serve` 命令可以在启动 server 时自动加载未列出的依赖

   2. 使用 `devbox shell` 切换到开发环境，使用脚本或命令生成工程目录并启动 http

   3. 访问 <http://localhost:4000> 查看页面。

## 2. 安装插件

通常 `Gemfile` 中添加依赖只需要添加包名和版本号即可，没有指定版本号时默认安装最新版。
插件依赖则需要放在 plugins 分组内。之后使用 `bundle` 命令进行安装即可。示例：

```shell
gem "jekyll", "~> 4.3.2"
gem "minima", "~> 2.5"

group :jekyll_plugins do
  gem "jekyll-remote-theme"
  gem "jekyll-compose"
end
```

> * jekyll-remote-theme 插件可以让网站从 repo 中直接拉取主题而不需要下载到本地
> * [jekyll-compose](https://github.com/jekyll/jekyll-compose)
  是一个用于快速生成 post 的工具，可以配置模板并使用命令简化操作

## 3. 安装主题

jekyll 官网的 resources 链接：[Resources](https://jekyllrb.com/resources/)，
其中最后一个站点发布的是付费主题。

找到心仪的主题后进入主题的 github 页面，大多都会在 README 内有详细的说明。
按着流程走就行。下面会讲一下多数主题的两种安装方式。

1. 启用远程主题插件（推荐）

   这种方式适用于多数主题。但除需改动 `_config.yml` 外也需要改动 `Gemfile` 来满足主题的依赖。

   * 在 Gemfile 中添加依赖 `jekyll-remote-theme` 依赖

     ```bash
     gem "jekyll-remote-theme"
     ```

   * 在 `_config.yml` 中的 `plugins` 中启用 `jekyll-remote-theme`，并加载主题的 repo

     ```yaml
     remote_theme: <user>/<theme-repo-name>

     plugins:
       - jekyll-remote-theme
     ```

   以安装 [leaf](https://github.com/supun-io/jekyll-theme-leaf) 主题为例：

   ```shell
   group :jekyll_plugins do
     gem "jekyll-remote-theme"
     gem "kramdown-parser-gfm"
   end
   ```
   {: file="Gemfile"}

   ```yaml
   remote_theme: supun-io/jekyll-theme-leaf

   plugins:
     - jekyll-remote-theme
   ```
   {: file="_config.yml"}

2. 本地安装

   下载作者的 repo 或 release，并基于此修改自己的配置进行部署。这种方式通常 Gem 依赖等所需的
   文件都是已经准备好的，只需要改动 `_config.yml`，如填写自己的 repo 和 host 即可。缺点是不方便拉取
   最新的主题变更，未来有可能需要使用 github 提供的 [compare 工具](https://docs.github.com/en/repositories/releasing-projects-on-github/comparing-releases) 比照改动。

## 4. 配置 github pages

1. 在本地 jekyll 目录中初始化 git 并添加自己 repo 的 remote url，push

2. 添加 github workflows，说明参见 [链接](https://docs.github.com/zh/actions/using-workflows)。
   [这里是一个可以工作的示例配置](https://github.com/duckduckio/pages.duckduck.io/blob/main/.github/workflows/pages.yml)，将该 yaml 放在目录 `.github/workflows/` 下，push 后即可让配置内的分支发生改变时触发工作流

3. 在 repo 页面右上角的 `···` 中找到 `Settings`，点左侧的 `Pages`

   * 更改 `Build and deployment` 为 Github Action
   * 更改 Custom domain 为自己的自定义域名, 不填则默认为 `<your_username>.github.io`，
     并启用 HTTPS
   * 如需自定义域名, 需要点 github 账号头像找到 settings 中 page 选项, 根据提示把验证用的 txt
     记录及访问路径的 A 记录添加到域名托管方的 DNS 设置中。设置自定义域后需要等通过了 github 的
     域验证后才能启用 HTTPS

在 Action 卡片中查看部署情况，没问题的话，访问自己的 github pages 地址，就能看到网站了

## 5. 写文章

Jekyll 文章构成包含文件名，元信息，正文三部分，且需要放在 _post 目录下。
如果启用了 [jekyll-compose](https://www.markdownguide.org/basic-syntax/) 插件的话，
可以使用命令快速生成。

* 文件名需要遵循 `yyyy-MM-dd-post-title-name.md` 规则，文件名后缀也可以是其他被支持的标记语言，
  同时 _post-title-name_ 部分也是文章的访问路径

* 元信息要放在文章开头，其中包含了文章名，作者，日期，分类，标签等信息。示例如下：

  ```text
  ---
  title: Post Title Name
  author: xxx
  date: yyyy-MM-dd HH:mm:ss
  categories: [c1, c2]
  tags: [t1, t2, t3]
  ---
  ```

* 正文默认使用 Markdown 标记语言。语法可参见 [链接](https://www.markdownguide.org/basic-syntax/)
