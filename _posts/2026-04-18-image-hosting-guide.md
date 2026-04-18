---
title: 如何使用图床在博客中插入图片
date: 2026-04-18 12:00:00 +0800
categories: [教程, 博客技巧]
tags: [图床, 图片, Markdown, 图床工具]
---

本文介绍如何在博客文章中方便地使用图片，以及推荐几种免费好用的图床方案。

## 什么是图床？

图床就是专门存放图片的网络服务器。你可以把图片上传到图床，获得一个链接，然后在 Markdown 文章中引用这个链接来显示图片。

相比把图片直接放在博客仓库里，使用图床的好处是：
- 仓库体积不会膨胀
- 图片加载速度通常更快（尤其是使用 CDN 加速的图床）
- 写作时不需要管理本地图片文件

---

## 方案一：GitHub 仓库 + jsDelivr CDN（推荐，免费稳定）

这是最适合 GitHub Pages 博客的方案。原理是：把图片上传到一个独立的 GitHub 仓库，然后通过 jsDelivr CDN 加速访问。

### 步骤

1. **新建一个图床仓库**
   - 在 GitHub 新建一个仓库，比如叫 `images` 或 `picbed`
   - 设置为 Public（公开）

2. **上传图片**
   - 进入仓库，点击 **Add file > Upload files**
   - 选择你要上传的图片，提交到仓库
   - 假设图片路径为：`images/2026/04/screenshot.png`

3. **获取图片链接**
   - 点击仓库中的图片，复制浏览器地址栏的链接
   - 格式类似：`https://github.com/你的用户名/images/blob/main/2026/04/screenshot.png`

4. **转换为 jsDelivr CDN 链接**
   - 把链接按以下规则替换：
   ```
   原链接：https://github.com/用户名/仓库/blob/分支/路径/图片.png
   CDN链接：https://cdn.jsdelivr.net/gh/用户名/仓库@分支/路径/图片.png
   ```
   - 示例：
   ```
   https://github.com/XiZu233/images/blob/main/2026/04/screenshot.png
   ↓ 转换为 ↓
   https://cdn.jsdelivr.net/gh/XiZu233/images@main/2026/04/screenshot.png
   ```

5. **在文章中引用**
   ```markdown
   ![截图说明](https://cdn.jsdelivr.net/gh/XiZu233/images@main/2026/04/screenshot.png)
   ```

---

## 方案二：SM.MS 图床（简单快速，有免费额度）

SM.MS 是一个老牌的免费图床，国内访问速度不错。

### 使用步骤

1. 打开 https://sm.ms
2. 注册并登录账号
3. 直接拖拽或选择图片上传
4. 上传成功后，复制 **Markdown** 格式的链接
5. 直接粘贴到文章中即可

### 免费额度
- 免费用户：每张图片最大 5MB，总量有一定限制
- 如果图片较多，建议升级到付费版或使用方案一

---

## 方案三：路过图床（无需注册）

https://imgse.com （原 SM.MS 国内镜像）

- 无需注册即可上传
- 支持拖拽上传
- 自动生成 Markdown 链接
- 适合临时使用或少量图片

---

## 方案四：PicGo 工具（效率最高，推荐搭配方案一）

如果你经常需要在文章中插入图片，手动上传再复制链接太麻烦。推荐安装 **PicGo** 工具，它可以让你截图后一键上传并自动复制 Markdown 链接。

### 安装与配置

1. 下载 PicGo：https://github.com/Molunerfinn/PicGo/releases
2. 安装后打开，选择**图床设置**
3. 推荐使用 **GitHub 图床** 插件：
   - 仓库名：`XiZu233/images`
   - 分支：`main`
   - Token：在 GitHub Settings > Developer settings > Personal access tokens 中生成
   - 存储路径：`blog/`
4. 设置好后，截图 → 粘贴到 PicGo → 自动上传 → 复制 Markdown 链接

### 配合 Typora/VSCode 使用

如果你用 Typora 或 VSCode 写 Markdown，可以配置**粘贴图片时自动上传到图床**：
- **Typora**：偏好设置 → 图像 → 上传图片 → 选择 PicGo
- **VSCode**：安装 `PicGo` 插件，配置好后粘贴图片自动上传

---

## 在 Markdown 中引用图片的语法

无论使用哪种图床，Markdown 语法都是一样的：

```markdown
![图片描述](图片链接)
```

### 带链接的图片（点击图片可放大）

```markdown
[![图片描述](图片链接)](图片链接)
```

### 控制图片大小（用 HTML）

```html
<img src="图片链接" alt="描述" width="600">
```

### 图片居中显示

```html
<p align="center">
  <img src="图片链接" alt="描述" width="600">
</p>
```

---

## 我的推荐组合

| 场景 | 推荐方案 |
|---|---|
| 偶尔用几张图 | 路过图床 / SM.MS |
| 长期维护博客，图片较多 | GitHub 仓库 + jsDelivr CDN |
| 追求效率，经常写作 | PicGo + GitHub 图床 + Typora/VSCode |

---

## 注意事项

1. **图片不要太大**：建议单张图片不超过 2MB，否则加载慢
2. **给图片起好名字**：不要用 `IMG_20240418_123456.jpg` 这种无意义的名字，改成 `hexo-deploy-screenshot.png` 更方便管理
3. **备份重要图片**：即使是免费图床，也建议保留本地备份
