---
title: "本站新特性介绍"
date: 2020-07-29T21:05:09+08:00
author: Alvin
keywords: idealvin,Alvin,blog,about,hugo,hugo-ivy,latex,gitalk,plantuml,sidebar
plantuml: true
toc: true
---


近期对本站进行了更新，增加了不少新特性，同时微调了页面布局，使得网站页面看起来更加和谐、优雅。下面奉上一份不太详细的介绍:wave:。


## 简介

本站用 markdown 进行写作，然后用 [Hugo](https://gohugo.io) 生成静态页面，同时部署到 [github.io](https://idealvin.github.io/) 与 [gitee.io](https://idealvin.gitee.io/) 上。在作者的 github 上可以找到 [markdown 源文件](https://github.com/idealvin/blog)、[主题文件](https://github.com/idealvin/hugo-ivy) 与 [静态页面文件](https://github.com/idealvin/idealvin.github.io)。


## 新特性

### 侧边栏 (sidebar)

侧边栏能提升阅读效率，带来极致的阅读体验。只需在 markdown 开头加上 `toc: true`，就会自动根据 markdown 中的标题生成 sidebar:

```yaml
---
title: "xxx"
toc: true
---
```

### 时序图 (plantuml)

[plantuml](https://plantuml.com/) 是一个极佳的画图工具，特别地，能极大的提高画时序图的效率。在 markdown 中输入下面的代码看效果：

> \`\`\`plantuml  
> Alvin -> Bob : hello plantuml  
> \`\`\`

```plantuml
Alvin -> Bob : hello plantuml
```

有了 sidebar 与 plantuml，就可以方便的完成详细设计文档之类的长篇大论了:heart:

### 评论系统 (gitalk)

[gitalk](https://github.com/gitalk/gitalk) 是基于 github issues 实现的评论系统，用户需要修改 [comments.html](https://github.com/idealvin/blog/blob/master/themes/hugo-ivy/layouts/partials/comments.html) 模板文件，填上自己的 gitalk 信息。

本站支持在全局配置 [config.toml: Params](https://github.com/idealvin/blog/blob/master/config.toml#L52) 中加上 `comments = false`，禁用评论系统。也可以针对单个页面禁用评论，只需在 markdown 开头加上 `no_comment: true` 或 `nocom: true`:

```yaml
---
title: "xxx"
nocom: true
---
```

### 数学公式 (LaTex)

Markdown 中支持 [LaTex](https://www.latex-project.org/) 数学公式，此功能依赖于 [Mathjax](https://docs.mathjax.org/en/latest/configuration.html)。

Markdown 与 LaTeX 有一个冲突：`_` 在 markdown 中表示强调，在 LaTeX 中表示下标，可能导致 mathjax 渲染公式出错，[Hugo 简记](/coding/2018/08/hugo/) 一文提供了详细的解决方法。

由于并非所有页面都需要包含 LaTex 公式，mathjax 默认是不开启的，用户需要在 markdown 开头加上 `mathjax: true` 启用此功能：

```yaml
---
title: "xxx"
mathjax: true
---
```


## 细节优化

### 隐藏的 Next 按钮

鼠标放到文章右侧边缘黄金分隔点处，会出现一个不太起眼的 `>>` 符号，点击它可以跳转到下一篇文章，主要是为了方便用户一路 `Next` 点到底。

### 隐藏 scroll bar

文章不是很长时，没必要显示滚动条(影响美观:unamused:)。本站支持根据页面长度自动隐藏滚动条，用户可以修改全局配置 [config.toml: Params](https://github.com/idealvin/blog/blob/master/config.toml#L52) 中的参数：

```toml
hidsbn = 49    # list   页面文章数小于 49，  隐藏滚动条  (len(.Pages) < hidsbn)
hidsbc = 4000  # single 页面字符数小于 4000，隐藏滚动条  (len(.Content) < hidsbc)
```

另外，也可以在 markdown 开头加上 `hide_scroll_bar: true` 或 `hidsb: true`，显示隐藏页面中的滚动条：

```yaml
title: "xxx"
hidsb: true
```