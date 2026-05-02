---
title: 博客"墨水经典"精修与全站重做纪实
date: 2026-05-02 02:00:00 +0800
categories: [项目, 前端]
tags: [Jekyll, Chirpy, 博客主题, CSS, SCSS, 响应式设计, 墨水经典]
---

> 这篇文章记录了基于 jekyll-theme-chirpy 7.5 二次开发的个人博客全站改造过程。以"加强而非推翻"为基调，保留杂志风骨架，补强对比度、加入 Hero/作品集网格/三栏页脚，引入克制的滚动揭示与悬停反馈，统一强调色，并修复多个 chirpy 主题与 Liquid 模板的隐性 bug。

## 一、项目概述

这个博客（[xizu233.github.io](https://xizu233.github.io)）基于 jekyll-theme-chirpy 7.5（gem 形式）二次开发，原 UI 已有"墨水经典"杂志风骨架：纯墨黑 `#0a0a0b` + 暖米白 `#f1efea` + 衬线/无衬线/等宽三字族。但存在 11 处对比度、响应式、信息架构与视觉层次问题。

本次工作的核心思路是**保留杂志骨架，补强薄弱环节**：

- 修复对比度问题
- 加入 Hero 区域、作品集网格、三栏页脚
- 引入克制的滚动揭示与悬停反馈
- 统一强调色（墨蓝/米黄）
- 修复 chirpy 主题与 Liquid 模板的隐性 bug

工作时间：2026-05-01 ~ 2026-05-02  
最终 commit：`b4b4c8b`  
GitHub Actions 构建成功。

---

## 二、设计决策

在项目启动时，Claude 提供了四种重做方案：

| 方案 | 描述 | 选择结果 |
|------|------|---------|
| 墨水经典精修 | 保留杂志骨架，局部优化 | ✅ **选中** |
| 推翻重写 Tufte | 完全不同的学术风格 | ❌ 排除 |
| 仅改首页 | 最小改动 | ❌ 排除 |
| 强动效方案 | 视差、3D、复杂 transition | ❌ 排除 |

最终选择：**墨水经典精修 + 全站重做 + 保留 light/dark 双模式 + 中度动效**。

排除强动效的原因是保持阅读体验优先，排除推翻重写的原因是原有骨架已经符合个人审美偏好。

---

## 三、关键架构决策

| 决策 | 选择 | 原因 |
|---|---|---|
| 主题覆盖策略 | 整体覆盖 chirpy 7.5 的 home/sidebar/footer/topbar/post 五个 layout/include | Jekyll 不支持 partial override；这五个是杂志风改造主战场 |
| 文章末尾增强 | 通过 chirpy `tail_includes` 插槽追加 | 不动模板正文，便于未来主题升级 |
| 自定义 JS 注入 | 通过 layout frontmatter 的 `script_includes` 注入 | 复用 chirpy hook，不碰 `_includes/js-selector.html` |
| 作品集数据驱动 | `_data/projects.yml` 抽离结构化数据 + `_layouts/projects.html` 渲染 | 后续维护只改 yml |
| Hero 内容 | front matter 静态 + Liquid 取最新一篇 | 不增加 `_config.yml` 复杂度 |
| 强调色 | 墨蓝 `#1f3a5f`（light）+ 米黄 `#d4b86a`（dark） | 与墨水风格协调，仅用于 link-hover / TOC active / 主按钮 |
| 暗色基调 | 从 `#0a0a0b` 调到 `#16161a`，paper 暗对应物 `#1f1f24` | 减少纯黑刺眼，仍保持深沉 |
| 动效层 | Intersection Observer + CSS keyframes，全部带 `prefers-reduced-motion: reduce` 守卫 | 中度动效，可达性兜底 |

---

## 四、技术栈

| 层 | 技术 |
|---|---|
| 静态生成 | Jekyll 4.4.1 |
| 主题 | jekyll-theme-chirpy 7.5.0（gem） |
| 样式 | SCSS（覆盖 chirpy 变量） |
| 模板 | Liquid |
| 数据 | YAML（`_data/`） |
| 脚本 | Vanilla JS + Intersection Observer |
| 部署 | GitHub Pages（Actions 构建） |
| 字体 CDN | fonts.loli.net |
| 验收工具 | Playwright（本地 4000 端口预览） |
| 本地构建 | WSL2 + Ruby/Bundler（`--force_polling`） |

**不引入新 npm/gem 依赖**，保持依赖简洁。

---

## 五、实施记录

整个改造分为 8 个 commit，按依赖顺序推进：

### Commit 1 · `fe9ee5f` style(theme): 强化墨水经典对比度与中文排版

**改动文件**：`_sass/custom/_colors.scss`、`_sass/custom/_typography.scss`

**要点**：
- 新增 `--ink-accent` 强调色变量（light=墨蓝、dark=米黄）
- `--ink-faint` 从 0.25 提到 0.4（修复链接下划线过淡）
- 暗色 `--main-bg` 从 `#0a0a0b` → `#16161a`，`--card-bg` 从 `#ffffff` → `#fbfaf6`
- 元数据 `text-transform: uppercase` 限定到 `:lang(en)`（修复中文场景错位）
- 新增 1024px、1280px 断点
- TOC active 加 `transform: translateX` 滑入动画

### Commit 2 · `831c0be` style(theme): 新增中度动效层

**新增文件**：`_sass/custom/_animations.scss`、`_includes/scroll-reveal.html`、`_includes/toc-highlight.html`

**要点**：
- 滚动揭示（fade-in + slide-up 6px）
- 链接下划线展开
- 卡片左侧 0 → 3px 竖线悬停
- TOC active 滑入
- `prefers-reduced-motion: reduce` 守卫所有动效

### Commit 3 · `a6c3a94` style(theme): Hero + 杂志风侧栏顶栏页脚

**新增文件**：`_sass/custom/_hero.scss`、`_includes/hero.html`、`_includes/post-tail-magazine.html`，整体覆盖 `_includes/sidebar.html`、`_includes/topbar.html`、`_includes/footer.html`

**要点**：
- 首页 Hero：ISSUE 号 + 大字号衬线主标题 + 副标题 + 关键词标签 + LATEST 卡片
- 侧栏插入 "Issue No.{posts.size} — {site.time}" 期刊号
- 顶栏 `#topbar-title` 旁加副标题"a quiet journal"
- 页脚改为三栏 grid：站点简介+版权 / 近期分类 / 联系方式

### Commit 4 · `bd5cd36` feat(projects): 作品集数据化与响应式网格

**新增文件**：`_data/projects.yml`、`_layouts/projects.html`、`_sass/custom/_projects-grid.scss`

**要点**：
- 2 列（≥768）/ 3 列（≥1280）卡片网格
- 含技术栈标签、外链按钮、悬停升起 1px

### Commit 5 · `943f3a4` fix(theme): 修复 Liquid 渲染与 light 模式色变量优先级

这是最重要的修复 commit，解决了 4 个 bug（详见第六节）。

### Commit 6 · `e0d0041` chore: gitignore 加入本地预览/plan/截图目录

### Commit 7 · `b4b4c8b` fix(about): title 改用英文 key 触发 chirpy locale 翻译

---

## 六、修复记录：5 个 bug

### Bug 1 · /projects/ 页 document.title 为空（"| Wenchen"）

**根因**：chirpy `_includes/head.html` 用 `page.title | downcase` 作 `site.data.locales[lang].tabs[tab_key]` 的 lookup key。中文 title `"项目"` downcase 后查 `zh-CN.yml.tabs` 字典 miss，title 退化为空。

**修复**：
1. 新建本地 `_data/locales/zh-CN.yml` 全文覆盖 chirpy vendor 内容
2. tabs 节追加 `projects: 项目`
3. `_tabs/projects.md` 的 title 改为英文 key `projects`

**关键发现**：Jekyll 的 `_data` 是文件级 override（同名 yml 完全替换 theme 提供的版本，**不合并**），所以必须把 chirpy 原文件全部抄过来再加自定义键。

### Bug 2 · light 模式 `--main-bg` 仍是 `white` 而非 `#f1efea`

**根因**：chirpy `_sass/base/_base.scss:12-33` 用 `@media (prefers-color-scheme: light) { html[data-mode='light'] { ... } }` 选择器声明 light 变量，特异性高于纯 `:root`。原来在 `_colors.scss` 用 `:root` 覆盖，被 chirpy 媒体查询里的 `html[...]` 选择器盖过。

**修复**：完全重写 `_sass/custom/_colors.scss`，使用与 chirpy 同选择器特异性的覆盖策略：

```scss
:root {
  --ink: #0a0a0b;
  --paper: #f1efea;
  --paper-tint: #e8e5de;
  --ink-accent-light: #1f3a5f;
  --ink-accent-dark: #d4b86a;
}

/* light 覆盖：与 chirpy 同选择器特异性 */
html:not([data-mode]),
html[data-mode='light'] {
  --main-bg: var(--paper);
  --card-bg: #fbfaf6;
  --sidebar-bg: var(--paper-tint);
  --text-color: var(--ink);
}

html[data-mode='dark'] {
  --main-bg: #16161a;
}
```

凭借 SCSS import 顺序在 chirpy 之后，相同特异性下后者胜出。

### Bug 3 · Liquid `slice` 在 Hash 上行为不一致

**根因**：`site.categories` 是 Hash（key=分类名、value=Array），Liquid `slice` filter 在 Hash 上的行为版本/实现不一致，可能丢键。

**修复**：改用 `sort | for ... limit`：

```liquid
{% assign top_cats = site.categories | sort %}
{% for cat in top_cats limit: 4 %}
  ...
{% endfor %}
```

### Bug 4 · Liquid filter pipeline 顺序导致 slugify 越界

**根因**：`'/categories/' | append: cat[0] | slugify` 中 slugify 把整串处理，把 `/` 变 `-`，URL 错位。

**修复**：用 capture / assign 抽中间变量：

```liquid
{% assign cat_slug = cat[0] | slugify | url_encode %}
<a href="{{ '/categories/' | append: cat_slug | append: '/' | relative_url }}">
```

### Bug 5 · /about/ 页 title 同样为空

**根因**：与 Bug 1 同源——`title: 关于我` 在 `tabs[tab_key]` lookup miss。

**修复**：`_tabs/about.md` 改 `title: about`，触发 `zh-CN.yml.tabs.about=关于` 翻译。

---

## 七、验证矩阵

| 页面 | 验证项 | 状态 |
|---|---|---|
| `/` 首页 (light) | Hero + 卡片 + 三栏页脚 | 通过 |
| `/` 首页 (dark) | 模式切换无跳变 | 通过 |
| `/posts/{slug}/` | TOC 48 锚点 + 相关文章 2 张 + post-nav | 通过 |
| `/projects/` | 作品集网格响应式 | 通过 |
| `/categories/` | 5 个分类卡片渲染 | 通过 |
| `/tags/` | 21 标签 | 通过 |
| `/archives/` | 4 篇文章按年归档 | 通过 |
| `/about/` | 关于内容 + 3 章节 | 通过 |

**响应式断点**：

| 视口 | hero-title 字号 | 页脚列数 | 侧栏状态 | 状态 |
|---|---|---|---|---|
| 768 x 1024 | 36px | 2 列 | 折叠 | 通过 |
| 1024 x 800 | 56px | 2 列 | 展开 | 通过 |
| 1280 x 800 | 68px | 3 列 (2:1:1) | 展开 + 右面板 | 通过 |

---

## 八、关键经验

### 1. Chirpy 主题 lookup 机制

`_includes/head.html` 在生成 `<title>` 时按以下顺序：

```
page.title | downcase  →  site.data.locales[lang].tabs[tab_key]  →  page.title
```

如果 `tab_key` 在 locale tabs 字典命中就用翻译值，否则回到原 title。**中文 title 在 lookup 时下钻到 tabs[中文 key]，永远 miss，最终 title 为空**。

**通用模式**：自定义 tab 的 title 必须是英文 key，对应翻译写在 `_data/locales/{lang}.yml` 的 `tabs.<key>`。

### 2. CSS 变量优先级

chirpy `_base.scss` 用 `@media (prefers-color-scheme) { html[data-mode='X'] { ... } }` 而不是纯 `:root`，特异性是 `(0,2,1)`。任何在 `_colors.scss` 用 `:root` 覆盖（特异性 `(0,1,0)`）都失效。必须用同选择器或更高特异性。

### 3. Jekyll `_data` 不合并

`_data/locales/zh-CN.yml` 在本地存在则**完全替换** theme 提供的同名文件。必须把 chirpy vendor 内容全部抄过来，再加自定义键。

### 4. Liquid filter 链顺序

`'/path/' | append: variable | slugify` 会把整个 `/path/value` 一起 slugify，**斜杠也被替换**。必须先 capture/assign 中间值再 append。

### 5. WSL2 cross-fs 文件监听

Jekyll 在 WSL2 监听 `/mnt/e/` (NTFS cross-fs) 时不收到 inotify 事件，必须加 `--force_polling`：

```bash
bundle exec jekyll serve --livereload --force_polling
```

### 6. 整体覆盖 layout 的维护成本

将 chirpy 5 个文件（home/sidebar/topbar/footer/post）整体覆盖后，未来 chirpy 升级（>= 7.6）需手工合并。已在每个覆盖文件顶部加注释 `<!-- Override of jekyll-theme-chirpy@7.5 ... -->` 标记基准版本。

---

## 九、文件变更清单

### 新增（15 个文件）

| 文件 | 作用 |
|---|---|
| `_data/projects.yml` | 作品集结构化数据 |
| `_data/locales/zh-CN.yml` | 覆盖 chirpy 同名文件，加 `projects: 项目` |
| `_layouts/projects.html` | 作品集网格渲染 |
| `_layouts/home.html` | 整体覆盖 chirpy 7.5 同名 |
| `_layouts/post.html` | 整体覆盖（仅 frontmatter） |
| `_includes/hero.html` | 首页 Hero |
| `_includes/sidebar.html` | 整体覆盖 + Issue 期刊号 |
| `_includes/topbar.html` | 整体覆盖 + 副标题 |
| `_includes/footer.html` | 整体覆盖 + 三栏 grid |
| `_includes/post-tail-magazine.html` | 文章末尾"延伸阅读" |
| `_includes/scroll-reveal.html` | 滚动揭示脚本 |
| `_includes/toc-highlight.html` | TOC active 滑入脚本 |
| `_sass/custom/_animations.scss` | 动效层 |
| `_sass/custom/_hero.scss` | Hero 样式 |
| `_sass/custom/_projects-grid.scss` | 作品集网格样式 |

### 修改（8 个文件）

| 文件 | 改动要点 |
|---|---|
| `_sass/custom/_colors.scss` | 重写：`:root` 抽象色 + `html[data-mode]` 覆盖 |
| `_sass/custom/_typography.scss` | uppercase 限 `:lang(en)`，加 1024/1280 断点 |
| `_sass/custom/_components.scss` | 卡片悬停左竖线 + reduced-motion 守卫 |
| `_sass/custom/_index.scss` | 加 3 个 import |
| `assets/css/jekyll-theme-chirpy.scss` | 同步加 3 个 import |
| `_tabs/projects.md` | layout: projects, title: projects（英文 key） |
| `_tabs/about.md` | title: about（英文 key） |
| `.gitignore` | 加入 `.claude/`、`.playwright-mcp/`、`preview-*.png` 等 |

---

## 十、写在最后

这次改造不只是换了个皮肤，而是把博客从一个"能用的主题"变成了"符合个人审美的阅读空间"。最花时间的不是写样式，而是理解 chirpy 主题的扩展机制、处理 Liquid 模板的隐性行为、以及建立可靠的验证流程。

如果你也在用 Jekyll + Chirpy 搭建博客，希望这篇文章中的踩坑记录能帮你少走一些弯路。

---

## 相关链接

- 仓库：[XiZu233/XiZu233.github.io](https://github.com/XiZu233/XiZu233.github.io)
- 在线站点：[xizu233.github.io](https://xizu233.github.io/)
- chirpy 主题：[cotes2020/jekyll-theme-chirpy](https://github.com/cotes2020/jekyll-theme-chirpy)
