---
title: "使用 Jekyll 部署 Github Pages"
author: aold619
date: 2022-08-25 22:00 +0800
categories: [Tutorial]
tags: [tutorial, github pages]
---

## 安装 jekyll

Jekyll Blog 是基于 Ruby 的，所以要先安装 [Ruby](https://www.ruby-lang.org/en/) 和 [RubyGems](https://rubygems.org/pages/download)。Linux 或 MacOS 直接使用包管理器。

RubyGems 是一个程序包管理器，类似于 Python 的 Pip，但安装它主要是为了在本地 build 看页面是否正常，如果直接使用 github pages 部署，可以不用安装。

安装后使用 gem 安装 bundler 和 jekyll:

```shell
gem install bundler jekyll
```

或者直接使用 [docker 镜像](https://hub.docker.com/r/jekyll/jekyll) 使用 jekyll，但使用命令时需要加上环境变量，否则会提示没有权限：
```shell
-e JEKYLL_UID=1001 -e JEKYLL_GID=1001
```

jekyll 初始化博客程序
```shell
jekyll new your-blog-name
cd your-blog-name
```

jekyll 会自动生成项目文件，建议使用自己的 github pages repo 的名称，或将要绑定的自定义域名。github pages 的默认域名是 `username.github.io`，也可以使用它。

## 安装 theme

jekyll 官网的 resources 链接: [Resources](https://jekyllrb.com/resources/)

在 theme 链接中寻找喜欢的主题，根据 github repo 的说明进行安装。

多数主题只需要修改 jekyll 的 `Gemfile` 和 `_config.yml` 即可，也有像我使用的 [chirpy theme](https://github.com/cotes2020/jekyll-theme-chirpy/) 主题这样，repo 中已经包含了 jekyll 的所有文件，直接从主题的 repo 生成自己的 repo 即可。

其中在 `_config.yml` 中，关于 github pages 的地址只需要配置在 `url` 里即可，`baseurl` 是指 url 中的 sub path，不用填写的。

## 配置 github pages

1. 在本地 jekyll 目录中初始化 git 并添加自己 repo 的 remote url，push
2. 在 github 的 repo 的页面中找到 `Settings`，点左侧的 `Pages`
  * 更改 `Build and deployment` 为 Github Actions，repo 会提供一个持续部署的 workflow 脚本，直接编辑并提交该文件即可
  * 更改 Custom domain 为自己的自定义域名，不填则默认为 github 自己的 `username.github.io`
  * 如果用自己的域名，还需要点自己头像中的 settings 找到 page 根据提示给域名的 DNS 添加 txt 和 A 记录

在 Action 卡片中查看部署情况，没问题的话，访问自己的 github pages 地址，就能看到 blog 了

## 写文章

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
