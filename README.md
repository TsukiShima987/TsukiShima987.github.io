# TsukiShima987.github.io

个人博客源码。使用 Jekyll 构建，minima 主题，通过 GitHub Actions 自动发布到 GitHub Pages。

- **线上地址**：https://tsukishima987.github.io
- **技术栈**：Jekyll 4.4.1 + minima 2.5.2（Ruby 3.x）
- **部署方式**：推送 `main` 分支即自动构建发布，无需手动操作

---

## 目录

- [环境要求](#环境要求)
- [本地开发](#本地开发)
- [目录结构](#目录结构)
- [写文章](#写文章)
- [修改界面](#修改界面)
- [配置说明](#配置说明)
- [部署流程](#部署流程)
- [命令速查](#命令速查)
- [排错](#排错)
- [本项目约定](#本项目约定)

---

## 环境要求

| 依赖 | 版本 | 说明 |
|---|---|---|
| Ruby | 3.0+ | 本地为 3.0.2 |
| Bundler | 2.x | 本地为 2.5.23 |
| Jekyll | 4.4.1 | 由 `Gemfile` 锁定，不需要单独装 |

首次克隆仓库后执行：

```bash
bundle install
```

---

## 本地开发

```bash
# 启动本地服务，浏览器打开 http://127.0.0.1:4000
bundle exec jekyll serve

# 加上改完自动刷新
bundle exec jekyll serve --livereload

# 连 _drafts/ 里的草稿一起预览
bundle exec jekyll serve --drafts
```

**务必使用 `bundle exec` 前缀。** 它保证用的是 `Gemfile.lock` 锁定的那套 gem，和 GitHub Actions 上构建时完全同版本。直接跑 `jekyll serve` 会用系统里装的版本，可能不一致。

---

## 目录结构

```
.
├── _config.yml              全局配置（改动需重启 serve）
├── Gemfile / Gemfile.lock   依赖声明与版本锁定（锁文件必须提交）
├── index.md                 首页（layout: home）
├── about.md                 关于页
├── 404.html                 404 页面
├── _posts/                  文章（唯一的发布入口）
├── _drafts/                 草稿（默认不构建）
├── _layouts/                页面骨架模板（覆盖主题同名文件）
│   ├── home.html            自定义首页列表布局
│   └── post.html            文章页布局（在正文后插入音乐卡片）
├── _includes/               可复用片段
│   └── music.html           文末音乐条（贴一个链接即可）
├── _sass/                   SCSS 变量与样式片段
│   └── _custom.scss         自定义样式（页面背景图、音乐卡片）
├── assets/                  样式入口与静态资源
│   ├── main.scss            样式入口（引入 minima + custom）
│   └── images/              图片资源
├── favicon.ico              网站图标（多尺寸）
├── apple-touch-icon.png     iOS 添加到主屏时的图标
├── _data/                   结构化数据（YAML/JSON，模板可读取）
├── .github/workflows/       GitHub Actions 构建发布流程
└── _site/                   构建产物，自动生成，不要手动修改
```

**命名约定**：以下划线开头的目录不会直接输出到 `_site/`。这就是 `_posts/` 里的 markdown 不会原样出现在站点上，而是被渲染成 HTML 的原因。以 `.` 开头的文件（如 `.gitignore`、`.github/`）会被自动忽略。

---

## 写文章

### 新建文章

在 `_posts/` 下创建文件，**文件名格式是硬性要求**：`YYYY-MM-DD-slug.md`

```
_posts/2026-09-25-git-notes.md
```

文件内容：

```markdown
---
title: "Git 常用命令笔记"
date: 2026-09-25 20:30:00 +0800
categories: [技术, 工具]
tags: [git]
---

正文用 Markdown 写即可。

## 二级标题

- 支持列表
- 支持 `行内代码`

```bash
git log --oneline -10
```
```

### 三条必知规则

1. **`slug`（文件名里日期后的部分）决定 URL，`title` 不影响 URL。**
   所以文件名用英文小写加连字符，标题可以写中文。

   | 文件名 | 生成的 URL |
   |---|---|
   | `2026-09-25-git-notes.md` | `/2026/09/25/git-notes/` |

2. **不需要写 `layout: post`。** `_config.yml` 的 `defaults` 已为 `_posts/` 下所有文件预设 `layout: post`，为页面预设 `layout: page`。

3. **日期不能是未来时间**，否则 Jekyll 默认跳过不构建。确需如此可在 `_config.yml` 设 `future: true`。

### 控制摘要

`_config.yml` 中 `show_excerpts: true` 已开启，首页列表显示摘要。默认取正文**第一段**。

想要手动截断，在 `_config.yml` 添加分隔符：

```yaml
excerpt_separator: "<!--more-->"
```

然后在正文里插入 `<!--more-->`，它上方的内容即为摘要。也可以在单篇文章的 front matter 里直接指定 `excerpt: "自定义摘要"`。

### 草稿

写到一半的内容放 `_drafts/`，文件名**不带日期**：

```
_drafts/jekyll-tips.md
```

只有预览时才包含草稿：

```bash
bundle exec jekyll serve --drafts
```

发布即把文件移入 `_posts/` 并补上日期前缀。

### 插入图片

1. 图片放在 `assets/images/`
2. markdown 中引用：

```markdown
![配置截图]({{ '/assets/images/setup.png' | relative_url }})
```

**要用 `relative_url` 过滤器**，不要写死 `/assets/...`，这样将来变更 `baseurl` 时路径不会全部失效。

> GitHub Pages 的服务器**区分大小写**。`Setup.PNG` 与 `setup.png` 是两个不同文件，本地可能看不出问题，线上会 404。

### 分类与标签

`categories` / `tags` 目前仅作为元数据（首页列表会显示分类）。**minima 不会自动生成分类汇总页。** 如需 `/categories/技术/` 这类页面，需要使用 `jekyll-archives` 插件，或自行编写页面遍历 `site.categories`。

### 文章配乐（文末音乐条）

在文章 front matter 里贴一个音乐链接，正文之后就会出现一条音乐条：

```yaml
---
title: "文章标题"
date: 2026-09-25 20:00:00 +0800
music: "https://music.163.com/#/song?id=1997192690"
---
```

链接从哪来：网易云网页播放器或 App 里「分享 → 复制链接」，**直接粘贴即可，不需要自己做任何转换**——模板会从地址里自动取出歌曲 ID。歌名、歌手、封面都由播放器自己显示，不用手写。

**不写 `music` 的文章不会有任何变化**，它是按文章可选的。

支持三种链接：

| 链接类型 | 效果 |
|---|---|
| 网易云单曲（含 `music.163.com` 与 `id=`） | 直接显示播放器 + 跳转链接 |
| 音频直链（`.mp3` / `.m4a` / `.ogg` / `.wav` / `.flac`） | 浏览器原生播放器 |
| 其他任意链接 | 只显示一个「收听本期配乐」跳转链接 |

**播放器直接显示，没有折叠，也不依赖 JavaScript。** 样式完全透明，只有一条顶部分隔线，直接透出页面背景。想要一点底色，取消 `_sass/_custom.scss` 里 `.music-card` 中那三行注释即可。

**关于加载开销**：iframe 标了 `loading="lazy"`，浏览器会在滚动到附近时才加载它。但这个阈值相当宽松（Chrome 大约提前 1250px 就开始加载）：

- **长文章**（音乐条在几千像素之外）→ 首屏没有任何第三方请求，滚动到下部才加载
- **短文章**（整页高度不到两屏）→ 音乐条一开始就在阈值内，**打开页面就加载**。实测一篇 1500px 高的文章会因此产生 **22 个**发往 `music.163.com` 的请求

也就是说，让播放器直接可见，代价是**每个访客都会被网易云加载一次、并可能被种下 Cookie**。如果将来想省掉这个开销，把 `_includes/music.html` 换回 `<details>` 折叠方案即可（需要少量 JS）。

**控制台里的报错是正常的**：网易云播放器加载后会出现它自己脚本的 `SecurityError: Blocked a frame...` 和 `Permissions policy violation` 警告。这些来自 `music.163.com` 的页面本身（它试图访问父页面、申请传感器权限，被浏览器按跨域策略拦截），**不影响播放**。

相关文件：

- `_includes/music.html` —— 音乐条结构
- `_layouts/post.html` —— 在正文后调用该 include
- `_sass/_custom.scss` 中 `.music-card` 一节 —— 样式

---

## 修改界面

### 核心机制：站点文件覆盖主题文件

主题作为 gem 安装，**不要修改 gem 内的文件**（升级会丢失）。Jekyll 的机制是：在站点中创建与主题相同路径的文件，即可**完全替换**主题的对应文件。

注意是**整个文件替换**，不支持只覆盖其中一部分。想改哪个文件，先从主题目录复制一份出来：

```bash
cp "$(bundle info minima --path)/_includes/footer.html" _includes/footer.html
```

| 想修改的内容 | 需要创建的文件 |
|---|---|
| 颜色、字体、间距 | `assets/main.scss` + `_sass/_custom.scss` |
| 顶部导航栏结构 | `_includes/header.html` |
| 页脚 | `_includes/footer.html` |
| `<head>`（图标、字体、统计） | `_includes/head.html` |
| 全站骨架 | `_layouts/default.html` |
| 文章页排版 | `_layouts/post.html` |
| 普通页面排版 | `_layouts/page.html` |
| 首页文章列表 | `_layouts/home.html`（**本仓库已自定义**） |

### 修改样式

主题的样式入口是 `assets/main.scss`，内容仅为 `@import "minima"`。创建站点级的同名文件接管它：

`assets/main.scss`

```scss
---
---

@import "minima";
@import "custom";
```

`_sass/_custom.scss`（新建）

```scss
.site-header {
  border-top: 4px solid #d94f4f;
}

.post-title {
  letter-spacing: -0.02em;
}
```

前两行的 `---` 是**必须的**：Jekyll 通过 front matter 判断该文件需要交给 Sass 编译，空 front matter 即可。

想改主题配色，更干净的做法是覆盖主题变量。先查看可用变量：

```bash
grep -n "^\$" "$(bundle info minima --path)/_sass/minima.scss"
```

再把需要调整的变量复制到自己的文件里重新赋值。

> minima **2.5.2 没有** skin / 主题切换功能（`minima: skin: dark` 是 3.0 开发版才有的，在本版本上无效）。

### 修改导航栏

当前 `_config.yml` 配置为：

```yaml
header_pages:
  - about.md
```

所以导航只显示「关于」，站点标题本身点击即回首页。希望「首页」也出现在导航中：

```yaml
header_pages:
  - index.md
  - about.md
```

验证配置是否生效：

```bash
bundle exec jekyll build
grep -o 'page-link[^>]*>[^<]*' _site/index.html
```

### 添加归档页

新建 `archive.md`：

```markdown
---
title: 归档
permalink: /archive/
---

{% for post in site.posts %}
- {{ post.date | date: "%Y-%m-%d" }} — [{{ post.title }}]({{ post.url | relative_url }})
{% endfor %}
```

然后把 `archive.md` 加入 `header_pages`。

### 添加网站图标

最简单的做法：把 `favicon.ico` 放到仓库根目录。浏览器默认请求 `/favicon.ico`，Jekyll 会把它作为静态文件复制到 `_site/`。

如果需要 PNG/SVG 图标并显式声明，则需覆盖 `_includes/head.html` 并添加 `<link rel="icon" ...>`。

### 添加访问统计

minima 内置 Google Analytics 支持，**仅在 production 环境注入**：

```yaml
google_analytics: UA-你的ID
```

本地 `serve` 时 `jekyll.environment` 为 `development`，不会加载。GitHub Actions 中工作流设置了 `JEKYLL_ENV: production`，因此线上会生效。

---

## 配置说明

`_config.yml` 主要配置项：

| 配置项 | 当前值 | 说明 |
|---|---|---|
| `url` | `https://tsukishima987.github.io` | 站点完整地址 |
| `baseurl` | `""` | **用户站点必须为空字符串** |
| `permalink` | `/:year/:month/:day/:title/` | 文章 URL 格式 |
| `timezone` | `Asia/Shanghai` | 影响文章日期解析 |
| `theme` | `minima` | 主题 |
| `show_excerpts` | `true` | 列表显示摘要 |
| `header_pages` | `[about.md]` | 导航栏页面及顺序 |
| `defaults` | 见文件 | 为文章/页面预设 layout |

完整可用配置项参见 [Jekyll 官方配置文档](https://jekyllrb.com/docs/configuration/)。

---

## 部署流程

```
写 markdown → 本地预览 → git commit → git push → Actions 自动构建 → 上线
```

### 首次启用（仅需一次）

仓库 **Settings → Pages → Build and deployment → Source** 选择 **`GitHub Actions`**。

> 不要选择 "Deploy from a branch"。GitHub 内置构建器使用的是 Jekyll 3.9.x 且插件受限，与本地 4.4.1 不一致。

### 工作流

`.github/workflows/jekyll.yml`：

```
push 到 main
  → actions/checkout
  → ruby/setup-ruby（bundler-cache，按 Gemfile.lock 安装）
  → actions/configure-pages
  → bundle exec jekyll build
  → actions/upload-pages-artifact
  → actions/deploy-pages
```

推送后可在仓库 **Actions** 标签页查看进度，成功后约一分钟内线上更新。

**不要在 GitHub 网页上直接编辑文件**，会导致本地与远端分叉，下次推送产生冲突。

---

## 命令速查

```bash
# 本地预览
bundle exec jekyll serve --host 127.0.0.1 --port 4000

# 预览并自动刷新浏览器
bundle exec jekyll serve --livereload

# 连草稿一起预览
bundle exec jekyll serve --drafts

# 只构建不启服务
bundle exec jekyll build

# 模拟生产环境构建（可验证 GA 等生产专属逻辑）
JEKYLL_ENV=production bundle exec jekyll build

# 清除构建产物与缓存（遇到陈旧内容时使用）
bundle exec jekyll clean

# 查看某个 gem 的安装位置（复制主题文件时用）
bundle info minima --path

# 发布
git add -A && git commit -m "说明" && git push
```

---

## 排错

| 现象 | 原因与处理 |
|---|---|
| 改了 `_config.yml` 没反应 | `serve` 不会热加载配置，需 `Ctrl+C` 重启 |
| 新文章不出现 | 文件名不符合 `YYYY-MM-DD-xxx.md`，或日期是未来时间 |
| 首页显示旧内容 | `bundle exec jekyll clean` 后重新构建 |
| 线上正常、本地异常 | 本地 gem 版本漂移，执行 `bundle install` |
| 本地正常、线上异常 | 查看 Actions 构建日志；同时检查文件名大小写 |
| 构建成功但行为异常 | **重新读一遍配置文件核对**。YAML 键值写错时 Jekyll 往往不报错，只是静默使用默认值 |
| 构建失败 | 仓库 Actions 标签页点进失败的那次运行查看日志 |

---

## 本项目约定

以下几条是刻意为之，修改前请先理解原因。

### 1. 文章 URL 不包含分类

`permalink` 设为 `/:year/:month/:day/:title/`，**刻意不包含 `:categories`**。

minima 默认格式含分类，而分类通常写中文，会导致 URL 出现百分号转义（如 `/随笔/2026/09/20/welcome/`），分享链接和 CDN 处理都容易出问题。

### 2. 文章 slug 保持纯 ASCII

因为 slug 取自**文件名**而非 `title`，所以只要文件名用英文，URL 就一定是 ASCII——标题依然可以任意写中文。

### 3. 不使用 `github-pages` gem

引入它会把 Jekyll 降级到 3.9 并锁定插件白名单。本项目通过 GitHub Actions 自行构建，可以自由使用任意插件。

### 4. `Gemfile.lock` 必须提交

不提交会导致本地与 CI 各自解析出不同版本，出现「本地正常、线上样式错乱」这类难以排查的问题。

### 5. `_layouts/home.html` 是自定义覆盖版本

不是主题原版。原因：主题原版的 RSS 文案 `subscribe via RSS` 是硬编码英文，无法通过配置修改。本版本做了四件事：

- 文案改为中文
- 文章列表额外显示分类
- RSS 链接可通过 `minima: hide_rss_link: true` 关闭
- 无文章时显示提示文案

想回到主题原版，删除该文件即可。

### 6. 新增插件需同时改两处

`Gemfile` 里声明依赖，`_config.yml` 的 `plugins:` 里注册。改完执行 `bundle install` 并提交 `Gemfile.lock`。

---

## 参考文档

- [Jekyll 官方文档](https://jekyllrb.com/docs/)
- [Liquid 模板语法](https://shopify.github.io/liquid/)
- [minima 主题](https://github.com/jekyll/minima)
- [GitHub Pages 文档](https://docs.github.com/pages)
