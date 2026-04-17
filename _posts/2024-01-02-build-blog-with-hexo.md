---
title: 如何使用 Hexo + GitHub Pages 搭建个人博客
date: 2024-01-02 10:00:00 +0800
categories: [教程, 技术]
tags: [Hexo, GitHub Pages, 教程]
---

本文简要介绍如何利用闲置电脑和 GitHub Pages 免费搭建一个个人博客/简历/作品集网站。

## 技术选型

- **[Hexo](https://hexo.io/)**：基于 Node.js 的快速、简洁且高效的博客框架
- **[Butterfly](https://butterfly.js.org/)**：美观、功能丰富的 Hexo 主题
- **[GitHub Pages](https://pages.github.com/)**：免费的静态网站托管服务
- **[GitHub Actions](https://github.com/features/actions)**：自动化构建与部署

## 核心步骤

### 1. 环境准备

确保电脑已安装：
- [Node.js](https://nodejs.org/)（推荐 LTS 版本）
- [Git](https://git-scm.com/)

### 2. 初始化 Hexo 项目

```bash
npx hexo-cli init Blog-chen
cd Blog-chen
npm install
npm install hexo-theme-butterfly hexo-renderer-pug hexo-renderer-stylus --save
```

### 3. 配置主题与站点信息

修改 `_config.yml` 和 `_config.butterfly.yml`，设置站点标题、菜单、社交链接等。

### 4. 编写内容

在 `source/_posts/` 目录下用 Markdown 写文章；在 `source/about/`、`source/projects/` 等目录下创建独立页面。

### 5. 配置自动部署

添加 `.github/workflows/pages.yml` 工作流文件， push 到 GitHub 后自动构建并发布。

### 6. 开启 GitHub Pages

进入仓库 **Settings > Pages**，Source 选择 **GitHub Actions**。

## 本地预览

```bash
npm run server
```

访问 `http://localhost:4000/` 即可实时预览。

## 总结

整个方案零服务器成本，写作体验极佳（纯 Markdown），且主题美观、扩展性强。非常适合作为个人品牌与作品集展示入口。
