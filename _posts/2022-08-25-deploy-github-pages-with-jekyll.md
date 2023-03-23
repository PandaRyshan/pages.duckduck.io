---
title: "使用 Jekyll 部署 Github Pages"
author: aold619
date: 2022-08-25 22:00 +0800
categories: [Tutorial]
tags: [tutorial, github pages]
---

## Jekyll 是什么

[Jekyll](https://jekyllrb.com/docs/) 是一个静态网站生成器，支持常用的标记语言，如
markdown。因此可以迅速建立网站并专注内容。本篇文章旨在介绍 jekyll 的安装，并将 jekyll
部署到 github pages。

## gem 介绍

在开始安装之前需要了解一下 Ruby 语言环境：

* gem：用来管理库的工具 RubyGems，类似于 Python 的 Pip。可以使用 `gem install <package-name>` 命令安装包
* bundler：一个依赖管理器，可以确保 gem 中的库符合正确的依赖关系。通常我们使用 `bundler` 其提供的 `bundle` 程序 `add / install / lock / package` 来管理 Gemfile。使用 `bundler exec <cmd-from-other-packages>` 在执行 ruby 程序之前自动对依赖进行检查。
  * `bundle install`：根据目录下 Gemfile 安装指定依赖，如果存在 Gemfile.lock 则只安装 lock 内的依赖，不存在 lock 文件则会生成一个
  * `bundle update`：根据 Gemfile 更新并安装所有依赖的最新版本，同时也会更新 Gemfile.lock
  * `bundle lock`：生成 Gemfile.lock，使用 lock 文件以保证部署时的依赖与开发环境一致，同时可以作为依赖的一个备份，以方便回滚
  * `bundle package`：会把项目中所有的依赖打包到本地，以便在无网络环境下进行部署
* Gemfile：用来描述包的依赖关系，指定包的版本。类似于 Python 的 requirements.txt
* 无论是 Jekyll 还是它的主题或插件，本质上都是一个 gem 包。因此只要发布在 RubyGems 的包都可以使用 `gem` 命令进行安装

以 Jekyll 的默认 Gemfile 为例：

```shell
# jekyll package
gem "jekyll", "~> 4.3.2"

# theme
gem "minima", "~> 2.5"

# jekyll plugins group
group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.12"
end

# Windows and JRuby does not include zoneinfo files, so bundle the tzinfo-data gem
# and associated library.
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.1.1", :platforms => [:mingw, :x64_mingw, :mswin]

# Lock `http_parser.rb` gem to `v0.6.x` on JRuby builds since newer versions of the gem
# do not have a Java counterpart.
gem "http_parser.rb", "~> 0.6.0", :platforms => [:jruby]
```
{: file="Gemfile"}

在这个配置文件中，主要就是使用 `gem "<package-name>", "<version>"` 的方式描述依赖，`"-> 4.3.2"` 是指定这个包的版本范围仅为 4.3.x，但最低为 4.3.2。也可以用 `>`, `=>`，`<`，`=<` 表达式来规定版本号范围。如果没有指定版本号，则会在不冲突的前提下安装最新的版本。

## 安装环境

### 本地安装

本地安装是最直接的一种方式，根据不同操作系统跟随官网的 [Guide](https://jekyllrb.com/docs/) 进行操作。

> 环境是可选项，旨在从本地预览主题，或者文章的变更等。如果已经有稳定的 blog 环境，Gemfile 中的依赖也都注明了版本范围。只需要对文章进行操作，那么有一个支持 markdown 的文本编辑器即可。完全是可以直接部署的。
{: .prompt-tip}

### 使用 Docker

这种方式虽然可以避免环境污染，但缺点显而易见，命令太长，且每次容器启动都需要额外耗时。更多详情请参考 [官方 docker 镜像](https://hub.docker.com/r/jekyll/jekyll) 和 [使用文档](https://github.com/envygeeks/jekyll-docker/blob/master/README.md) 。

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

   > root 用户需要加上 `-e JEKYLL_UID=1001 -e JEKYLL_GID=1001`
   {: .prompt-tip}

2. 为项目的 Gemfile 添加 webrick

   ```shell
   echo 'gem "webrick", "~> 1.8"' >> Gemfile
   ```

   > [Ruby 3.0 后依赖 webrick](https://jekyllrb.com/docs/)。
   >
   > 挂载一个本地目录保留构建缓存可以加速镜像启动。
   {: .prompt-tip}

3. 安装依赖

   ```shell
   docker run --name bundle \
     --volume="$PWD:/srv/jekyll:Z" \
     --volume="$PWD/vendor/bundle:/usr/local/bundle:Z" \
     -it jekyll/jekyll \
     bundle install
   ```

   > 如要更新依赖的新版本请使用 `bundle update`
   >
   > 可以保留安装和服务两个容器循环使用。也可以把挂载路径改为绝对路径。
   {: .prompt-tip}

4. 开启服务

   ```shell
   docker run -d --name jekyll \
     --volume="$PWD:/srv/jekyll:Z" \
     --volume="$PWD/vendor/bundle:/usr/local/bundle:Z" \
     --publish 4000:4000 \
     jekyll/jekyll \
     jekyll serve
   ```

5. 访问 <http://localhost:4000> 查看页面

### 使用 Devbox

[Devbox](https://github.com/jetpack-io/devbox/tree/main/examples/stacks/jekyll) 是一个基于 Nix 包管理器和构建工具的虚拟 shell 环境。根据不同项目的配置文件来提供隔离的 shell 环境，并继承当前的 shell 配置。也是一个不错的选择，当前版本 0.4.4 还是有一些 bug，但总的来说不影响使用。

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
              "jekyll new my-blog && cd my-blog",
              "bundle add webrick",
              "bundle update",
              "bundle lock",
              "bundle package",
              "rm -rf vendor"
            ],
            "serve": [
              "bundler exec $GEM_HOME/bin/jekyll serve --trace"
            ]
          }
        },
        "nixpkgs": {
          "commit": "f80ac848e3d6f0c12c52758c0f25c10c97ca3b62"
        }
      }
      ```
      {: file="devbox.json"}

      > 示例中包含 generate 和 serve 两个脚本，devbox 环境下使用 `devbox run <script-name>` 运行脚本。
      >
      > generate 脚本用来安装 jekyll 并生成名为 myblog 的工程目录。
      >
      > serve 脚本需要在 blog 根目录使用，用来启动 http；命令中使用 `bundler` 运行可以在启动前检查依赖。

   2. 使用 `devbox shell` 切换到开发环境，使用脚本或命令生成工程目录并启动 http

   3. 访问 <http://localhost:4000> 查看页面。

## 安装插件（可选）

插件和主题也都是 gem 包，通常在 `Gemfile` 中只需要添加包名和版本号即可，没有指定版本号时默认安装最新版。通常我们会把插件放进一个 group 内，group 是一个更灵活的用法。便于定位，同时可以防止和其他 gem 产生冲突，因为 group 可以针对不同的环境来定制不同版本的依赖，而写在外层的包只能出现一次。示例：

```shell
... other requirements ...

gem "jekyll", "~> 4.3.2"

group :jekyll_plugins do
  gem "jekyll-compose"
  gem "jekyll-feed"
end

... other requirements ...
```
{: file="Gemfile"}

> jekyll-compose 是一个用于快速生成 post 的工具，可以配置模板并使用命令简化操作
>
> jekyll-feed 可以为博客提供订阅功能

## 安装主题（可选）

jekyll 官网的 [Resources](https://jekyllrb.com/resources/) 链接包含了几个主题站点，其中最后一个站点发布的是付费主题。

找到心仪的主题后进入主题的 github 页面，大多都会在 README 内有详细的说明。这里说一下适用于大多数主题的两种安装方式。

### 安装 gem 中已提供的主题包

以我当前正在使用的 [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) 主题为例：

1. 在 Jekyll 默认的 Gemfile 中添加依赖和版本范围

   ```shell
   ... other requirements ...

   # 注释或删掉原有的主题包，添加新的主题包依赖
   # 不加版本信息表示每次 bundle 都使用最新版，但不指定版本有可能会引起预期外的变化
   # gem "jekyll-theme-chirpy", "~> 5.6", ">= 5.6.0"

   gem "jekyll-theme-chirpy"

   ... other requirements ...
   ```
   {: file="Gemfile"}

2. `bundle update`

3. 在 _config.yml 中启用主题

   ```yaml
   ... other configuration ...

   theme: jekyll-theme-chirpy

   ... other configuration ...
   ```
   {: file="_config.yml"}

### 远程主题插件

适用于多数 repo 为公开的主题。这种方式可以一直保持主题为最新状态。如需对原主题进行自定义，则在需要的自定义的路径创建同名文件，即可覆盖原主题文件。缺点是主题没有显式的依赖关系，可能需要手动添加一些依赖。

1. 在 Gemfile 中添加插件依赖

  ```shell
  ... other requirements ...

  gem "jekyll-remote-theme"

  ... other requirements ...
  ```
  {: file="Gemfile"}

2. 在 `_config.yml` 中的 `plugins` 中启用 `jekyll-remote-theme`，并加载主题的 repo

  ```yaml
  ... other configuration ...

  remote_theme: <github-user>/<theme-repo-name>

  plugins:
    - jekyll-remote-theme
  
  ... other configuration ...
  ```
  {: file="_config.yml"}

以安装 [leaf](https://github.com/supun-io/jekyll-theme-leaf) 主题为例，我们还需额外 kramdown-parser-gfm，否则无法顺利开启主题：

```shell
... other requirements ...

group :jekyll_plugins do
  gem "jekyll-remote-theme"
  gem "kramdown-parser-gfm"
end

... other requirements ...
```
{: file="Gemfile"}

```yaml
remote_theme: supun-io/jekyll-theme-leaf

plugins:
  - jekyll-remote-theme
```
{: file="_config.yml"}

## 配置 github pages

1. 在本地 jekyll 目录中初始化 git 并添加自己 repo 的 remote url，push

2. 添加 github workflows，说明参见 [链接](https://docs.github.com/zh/actions/using-workflows)。
   [这里是一个可以工作的示例配置](https://github.com/duckduckio/pages.duckduck.io/blob/main/.github/workflows/pages.yml)，
   将该 yaml 放在目录 `.github/workflows/` 下，push 后即可让配置内的分支发生改变时触发工作流

3. 在 repo 页面右上角的 `···` 中找到 `Settings`，点左侧的 `Pages`

   * 更改 `Build and deployment` 为 Github Action
   * 更改 Custom domain 为自己的自定义域名，不填则默认为 `<your_username>.github.io`，
     并启用 HTTPS
   * 如需自定义域名，需要点 github 账号头像找到 settings 中 page 选项，根据提示把验证用的 txt
     记录及访问路径的 A 记录添加到域名托管方的 DNS 设置中。设置自定义域后需要等通过了 github
     的域验证后才能启用 HTTPS

在 Action 卡片中查看部署情况，没问题的话，访问自己的 github pages 地址，就能看到网站了

## 写文章

首先说一下 Jekyll 博客包含的几个要素：

* `page`：页面。是博客最大的分类，可以根据自己的需要创建大的归类，往往用于站点的导航
* `post`：文章。默认支持 html 和 markdown，markdown 的渲染器是 Kramdown。需要放在 _posts 中
  * 文件名需要遵循 `yyyy-MM-dd-post-title-name.md` 规则，文件名后缀也可以是其他被支持的标记语言，同时
    _post-title-name_ 部分也是文章的访问路径
  * 元信息要放在文章开头，其中包含了文章名，作者，日期，分类，标签等信息。示例如下：
  
    ```text
    ---
    title: Post Title Name
    author: xxx
    date: yyyy-MM-dd HH:mm:ss
    image: # 题图
    categories: [c1, c2]
    tags: [t1, t2, t3]
    published: true # 是否发布，值为 false 时不会显示在博客列表
    ---
    ```
  
  * 默认使用 Markdown 标记语言。语法可参见 [链接](https://www.markdownguide.org/basic-syntax/)
* `draft`：草稿。也是 post 的一种，放在 _drafts 文件夹中
* `athor`：作者。文章的元信息之一，博客中点作者链接可以查询该作者所有文章
* `category`：分类。post 的元信息之一，当使用 `categories` 时可以使用数组标记多个分类。
* `tag`：标签。post 的元信息之一，当使用 `tags` 时可以使用数组标记多个标签。

这里推荐一个插件 [jekyll-compose](https://github.com/jekyll/jekyll-compose)，可以使用它方便地新建 post，新建 page，存草稿，发布草稿，重命名等功能。在 Gemfile 中添加 `gem "jekyll-compose"` 后使用 `bundle` 即可安装，具体用法见链接。

此外 Jekyll 还有模板的概念，可以在 `_data` 目录下创建模板来抽象使用某一要素，最常用的是 `authors.yml`：

```yaml
a1:
  name: Xiaohong Hu
  twitter: https://twitter.com/aold619
  url: https://pages.duckduck.io/
```

这样我们在 post 的元信息中，只要使用 a1 即可显示我们对应的作者名。

## 评论系统

常用的评论系统有 [giscus](https://giscus.app/) 和 [disqus](https://disqus.com/)。
二者都提供了 reactions 和 comments 功能，且使用都很方便。

* giscus 是使用 github 的 discussions 功能，由 giscus bot 把评论发表在 repo 的 discussions 中
* disqus 需要绑定自己 github pages 的地址到 disqus 账号内，并通过 disqus 提供的 shortname
  来配置，评论数据则保存在 disqus

如果你使用的主题支持这二者之一，只需要按照各自链接中的 guide 填写信息或创建资源，然后将生成的配置粘贴进
`_config.yml` 即可。如果不支持，二者也都提供现成的 js 代码，可以自己插入到
`_layouts/post.html` 需要的位置即可。

关于 layouts 可以查看 [这里](https://jekyllrb.com/docs/step-by-step/04-layouts/)。
