---
title: "hugo 实现写完一键发布"
date: 2022-03-12T23:17:55+08:00
draft: false
tags: ["hugo", "pages", "obsidian"]
---

# 使用 Hugo 创建站点
首先打开 [Hugo](https://gohugo.io/)  官网 [Quick start](https://gohugo.io/getting-started/quick-start/)

hugo 的安装就不说了，如果这一步觉得很困难，就可以放弃了。 

## 新建站点new
```shell
hugo new site quickstart
cd quickstart
git init
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
echo theme = \"ananke\" >> config.toml
```

上述做完之后，可以先别急着玩主题，咱们可以后面慢慢探索

## 添加内容

```bash
hugo new posts/my-first-post.md
```

这时，我们可以打开 `posts/my-first-post.md` ，在里面使用 `markdown` 语法写一写文字。

比如我[这里](https://wudanyang.top/post/hello-world/)
![](Pasted%20image%2020220313000425.png)

使用 `hugo server` 命令可以在本地预览站点效果，如下所示：

![](Pasted%20image%2020220313001939.png)

# 发布网站
我们使用 `hugo` 命令生成静态文件，将静态文件上传到`托管服务器`

常见的托管服务器可以使用阿里云、百度云、腾讯云等多种云平台。

不过我们这里介绍的是一个免费的服务 GitHub pages

## 创建 Github pages

我这里将文章、配置等元数据和网站数据分了两个仓库
1. `元数据仓库`设置成了私有，因为里面有可能有一些敏感信息
2. `网站仓库`设置为公开，开启 GitHub pages 服务

当然，你也可以设置成公开并且都使用一个仓库。

`网站仓库`打开 `Settings` 选择 `Pages` 选项，选择数据源分支

### 示例
如下图所示，我这里分支设置成了 `gh-pages` 分支的根目录，并且使用了一个`自定义域名`，而且开启了强制 `https` 访问。自定义域名和 https 第一次使用完全可以不理会，咱们接着往下看。

![](Pasted%20image%2020220313001358.png)

有了 GitHub pages 之后，咱们可以在生成的 `public` 文件夹下创建一个 git 仓库，或者直接在上面的路径 /root 配置成 `public` 目录。将生成的站点数据 push 到 github 即可。

但是这样一通操作还是很复杂，而且我并不想让在私有仓库里面开启 pages 服务（要收费）

github 提供了另外一项服务：`workflows`。

这里 hugo 也提供了一个 `workflow` [hugo-setup](https://github.com/marketplace/actions/hugo-setup)，可以通过 github 仓库数据生成静态文件，并自动部署。

发布的仓库可以修改成刚才创建的 `网站仓库` 

![](Pasted%20image%2020220313003340.png)

`workflow` 的具体配置就不细说了，可以参考 github 的教程。

上面一套下来，`git push` 之后就会跟着一次自动部署，非常方便

# markdown 编辑器 Obsidian
市面上有很多 markdown 编辑器，并且各有优劣，这里介绍一个比较中规中矩，基本能满足要求的编辑器：[Obsidian](https://obsidian.md/)。

Obsidian 最近更新之后可以支持所见即所得的编辑，并且体验还不错。不过如果你对编辑器要求比较高，也可以使用 typora 。

这款编辑器下限较低，开箱即用，用完即走。但是上限很高，主打双链知识图谱功能，并且有非常丰富的插件，活跃的社区，无限的可能。

咱们就主要用这款编辑器，后面介绍图床的时候，会介绍到 Obsidian 的一个好处。

# 图床
到这里，hugo 创建站点了，网站也能发布了，基本的文字编辑也解决了。

还有一个比较大的问题，图片怎么办？

主流的方案是使用第三方图床，上传到第三方图床之后，将图片地址放到 markdown 中。

hugo 给出了另外一个方案，将图片放到 static （可配置）目录下，hugo 会将 static 目录下的内容放到站点根目录。

也就是说，我在 static 目录下的 a.png 可以通过 /a.png 的方式访问到。

这样的话，图片的问题也解决了。

但是不完美的是， /a.png 在编辑器无法识别，无法显示这张图片。

![](Pasted%20image%2020220313005855.png)

我这里给出一个折中的方案，在 obsidian 中配置一下选项：
![](Pasted%20image%2020220313010147.png)

配置好之后，就可以在文件中去掉 `/` 前缀，并且obsidian也能识别图片并显示。

然后在 static/js 目录下创建一个 img_trick.js 文件，放入以下内容

```javascript
$(".post .post-content img").each(function () {
    $(this).attr('src', "/img/" + $(this).attr('src'))
})
```

这段代码会将 `content` 中的所有图片地址前面拼上 `/img/` 路径，页面就回去找站点根目录下面的 `img/图片` ，最终用户就可以看到图片。

# 总结
自此，一键发布全部结束。

如果有兴趣的话，可以继续探索 hugo 的更多主题，我用的是 [even](https://github.com/olOwOlo/hugo-theme-even) 主题。

也可以继续探索 obsidian 的更多插件（比如 [obsidian-git](https://github.com/denolehov/obsidian-git) 插件可以定时 push 修改）。
