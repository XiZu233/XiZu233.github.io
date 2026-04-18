---
title: 如何在博客文章中插入图片与开启评论
date: 2026-04-18 11:00:00 +0800
categories: [教程, 博客技巧]
tags: [图片, 评论, Giscus, Markdown]
---

本文介绍如何在 Chirpy 主题博客中插入图片，以及如何为文章添加评论功能。

## 一、在文章中插入图片

### 方式 1：引用外部图片链接（最简单）

如果你的图片已经托管在图床或其他网站上，直接用 Markdown 语法引用即可：

```markdown
![图片描述](https://example.com/image.png)
```

例如引用一张占位图：

![示例占位图](https://via.placeholder.com/800x400?text=Sample+Image)

### 方式 2：使用本地图片（推荐）

把图片文件放在博客项目的 `assets/img/` 目录下。推荐按文章分文件夹存放：

```
assets/
└── img/
    └── posts/
        └── 你的文章名/
            ├── screenshot-1.png
            └── diagram-2.jpg
```

然后在文章中这样引用：

```markdown
![截图说明](/assets/img/posts/你的文章名/screenshot-1.png)
```

> **注意**：路径要以 `/` 开头，表示从网站根目录开始。

### 图片尺寸与对齐

Chirpy 主题对图片有自动样式处理。如果你想控制图片大小，可以用 HTML 标签：

```html
<img src="/assets/img/posts/sample/demo.png" alt="描述" width="600">
```

## 二、为博客添加评论功能

本博客已配置 **Giscus** 评论系统，它基于 GitHub Discussions，完全免费且无广告。

### 读者如何评论？

访问任何一篇文章的底部，如果你看到评论区，可以直接用 GitHub 账号登录并发表评论。所有评论数据都存储在你的 GitHub Discussions 中，不会丢失。

### 管理员需要完成的配置

如果你发现文章底部没有评论区，请按以下步骤完成最后配置：

#### 步骤 1：开启仓库的 Discussions 功能

1. 打开你的 GitHub 仓库：`https://github.com/XiZu233/XiZu233.github.io`
2. 点击上方的 **Settings**（设置）
3. 在 General 页面往下滚动，找到 **Features** 区域
4. 勾选 **Discussions** 选项

#### 步骤 2：获取 category_id

1. 访问 https://giscus.app/zh-CN
2. 在页面上填写：
   - 仓库：`XiZu233/XiZu233.github.io`
   - 页面 ↔️ Discussion 映射关系：选择 **路径**
   - Discussion 分类：选择 **Announcements**
3. 页面会自动生成一段代码，从中找到 `data-category-id` 的值
4. 打开博客项目的 `_config.yml` 文件
5. 找到 `giscus:` 配置下的 `category_id:`，把刚才复制的值填进去

#### 步骤 3：重新部署

```bash
cd e:\00Project\Blog-chen
git add _config.yml
git commit -m "chore: 配置评论系统 category_id"
git push
```

等待 2-5 分钟，刷新网站，评论区就会出现了。

## 三、发布带图片的文章的完整流程

假设你写了一篇新文章，里面用到了本地图片：

```
1. 新建文件夹：assets/img/posts/2026-04-20-我的新文章/
2. 把图片复制进去
3. 在 _posts/ 下创建文章 Markdown 文件
4. 文章中使用 /assets/img/posts/2026-04-20-我的新文章/xxx.png 引用
5. git add . && git commit -m "新增文章" && git push
6. 等待 Actions 构建完成
```

## 四、参考链接

- [Giscus 官方配置工具](https://giscus.app/zh-CN)
- [Chirpy 主题文档 - 写作](https://chirpy.cotes.page/posts/write-a-new-post/)
- [Markdown 图片语法](https://www.markdownguide.org/basic-syntax/#images)
