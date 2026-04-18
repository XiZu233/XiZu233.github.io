---
title: 为博客配置评论系统与图床自动上传
date: 2026-04-18 13:00:00 +0800
categories: [教程, 建站]
tags: [Giscus, 评论, 图床, PicGo, VSCode]
---

这篇文章记录了我为博客配置评论区和图床自动上传的过程。作为一个长期维护的个人博客，评论互动和图片展示是提升阅读体验的重要部分。

## 为什么要做这两件事？

### 评论系统的意义

博客如果只是单向输出，作者和读者之间就缺少连接。引入评论系统后：

- 读者可以对文章内容提出问题或补充
- 作者能收到真实的反馈，知道哪些内容对别人有帮助
- 评论区本身也能沉淀出有价值的讨论
- 相比第三方平台，数据存储在自己可控的地方，不会随平台关闭而丢失

### 图床的意义

写技术博客时，截图、流程图、架构图几乎不可避免。如果直接把图片放在博客仓库里：

- 仓库体积会越来越大
- Git 对二进制文件的管理效率低
- 国内访问 GitHub 原始资源的速度不稳定
- 写作时需要频繁手动管理图片路径

使用专门的图床 + CDN 加速，再配合编辑器自动上传，写作体验会好得多。

---

## 第一部分：配置 Giscus 评论系统

### 为什么选择 Giscus？

市面上可选的评论系统很多，比如 Disqus、Valine、Utterances、Giscus 等。我最终选择了 **Giscus**，原因如下：

| 方案 | 优点 | 缺点 |
|---|---|---|
| Disqus | 功能全面 | 有广告、国内访问慢 |
| Valine | 轻量、无后端 | 需要 LeanCloud，配置稍复杂 |
| Utterances | 基于 GitHub Issues | 界面相对简单 |
| **Giscus** | 基于 GitHub Discussions、支持中文、无广告 | 需要开启仓库 Discussions |

Giscus 基于 GitHub Discussions，这意味着：
- 评论数据存储在你的 GitHub 仓库里，完全可控
- 读者用 GitHub 账号登录即可评论，无需额外注册
- 支持 Markdown 格式的评论
- 支持表情回应（Reactions）

### 配置步骤

#### 1. 安装 Giscus GitHub App

1. 访问 [https://github.com/apps/giscus](https://github.com/apps/giscus)
2. 点击 **Install**，选择你的博客仓库
3. 授权安装即可

这一步的作用是：让 Giscus 有权限在你的仓库中读取和创建 Discussions。

#### 2. 开启仓库的 Discussions 功能

1. 打开你的博客仓库页面
2. 进入 **Settings > General**
3. 在 **Features** 区域找到 **Discussions**，勾选开启

> 如果没有开启 Discussions，Giscus 无法创建讨论帖，评论区就无法正常工作。

#### 3. 在 Giscus 官网获取配置信息

1. 访问 [https://giscus.app/zh-CN](https://giscus.app/zh-CN)
2. 填写仓库信息：
   - 仓库：`你的用户名/你的博客仓库名`
   - 页面 ↔️ Discussion 映射关系：**路径**（pathname）
   - Discussion 分类：**Announcements**
3. 页面会自动生成一段代码，其中包含两个关键值：
   - `data-repo-id`
   - `data-category-id`

#### 4. 填入博客配置文件

打开博客项目的 `_config.yml`，找到 `comments` 和 `giscus` 配置段：

```yaml
comments:
  provider: giscus

giscus:
  repo: 你的用户名/你的博客仓库名
  repo_id: "从 Giscus 官网获取的 repo_id"
  category: Announcements
  category_id: "从 Giscus 官网获取的 category_id"
  mapping: pathname
  strict: 0
  input_position: bottom
  lang: zh-CN
  reactions_enabled: 1
```

#### 5. 推送并等待生效

```bash
git add _config.yml
git commit -m "feat: 配置 Giscus 评论系统"
git push
```

等待 GitHub Actions 构建完成（约 2~5 分钟），刷新任意一篇文章，底部就会出现评论区。

---

## 第二部分：配置图床自动上传

### 整体思路

要实现"写博客时只管粘贴图片，其他什么都不管"，需要几个组件配合：

```
截图（剪贴板）
  ↓
VSCode PicGo 插件（触发上传）
  ↓
GitHub 图床仓库（存储图片）
  ↓
jsDelivr CDN（加速访问）
  ↓
Markdown 中自动插入 CDN 链接
```

### 为什么选择 GitHub + jsDelivr？

- **GitHub 仓库**：免费、稳定、版本管理方便
- **jsDelivr CDN**：专门加速 GitHub 资源，国内访问速度快
- **两者结合**：完全免费，且和博客托管在同一平台，维护成本低

### 步骤 1：创建图床仓库

1. 在 GitHub 新建一个公开仓库，比如叫 `images` 或 `picbed`
2. 设置为 **Public**（否则图片无法被外部访问）
3. 勾选添加 README

### 步骤 2：生成 GitHub Personal Access Token

PicGo 需要通过 Token 才有权限向你的仓库上传图片。

1. 访问 [https://github.com/settings/tokens](https://github.com/settings/tokens)
2. 点击 **Generate new token (classic)**
3. Note 填：`PicGo Image Upload`
4. Expiration 选择 `No expiration` 或定期更换
5. Scopes 勾选 `repo`（完整控制仓库）
6. 点击 **Generate token**
7. **立即复制保存**（页面刷新后无法再次查看）

> ⚠️ **安全提醒**：Token 相当于你的仓库密码，不要泄露给他人，也不要提交到公开仓库。

### 步骤 3：安装 PicGo 桌面版（可选）

如果你需要独立的上传工具，可以安装 PicGo 桌面版：

1. 下载：[https://github.com/Molunerfinn/PicGo/releases](https://github.com/Molunerfinn/PicGo/releases)
2. 安装后配置 GitHub 图床：
   - 仓库名：`你的用户名/images`
   - 分支：`main`
   - Token：`ghp_你的Token`
   - 存储路径：`blog/`
   - 自定义域名：`https://cdn.jsdelivr.net/gh/你的用户名/images@main`

不过如果只是配合 VSCode 写作，也可以只安装 VSCode 插件，不装桌面版。

### 步骤 4：安装并配置 VSCode PicGo 插件

这是实现"粘贴即上传"的关键。

1. 打开 VSCode，进入扩展商店
2. 搜索并安装 **PicGo** 插件（作者：Spades）
3. 打开 VSCode 设置（`Ctrl + ,`），搜索 `picgo`
4. 填写以下配置：

```json
{
  "picgo.picBed.current": "github",
  "picgo.picBed.github.repo": "你的用户名/images",
  "picgo.picBed.github.branch": "main",
  "picgo.picBed.github.token": "ghp_你的Token",
  "picgo.picBed.github.path": "blog/",
  "picgo.picBed.github.customUrl": "https://cdn.jsdelivr.net/gh/你的用户名/images@main"
}
```

### 步骤 5：使用方式

配置完成后，在 VSCode 中编写 Markdown 时：

1. 截图（`Win + Shift + S`），图片进入剪贴板
2. 在 Markdown 文件中按 **`Ctrl + Alt + U`**
3. 插件自动读取剪贴板图片 → 上传到 GitHub 图床 → 在光标处插入 Markdown 链接

插入的链接格式如下：

```markdown
![image.png](https://cdn.jsdelivr.net/gh/你的用户名/images@main/blog/xxx.png)
```

如果你希望控制图片大小或居中，可以手动改成 HTML 格式：

```html
<p align="center">
  <img src="https://cdn.jsdelivr.net/gh/你的用户名/images@main/blog/xxx.png" alt="描述" width="600">
</p>
```

---

## 常见问题与排查

### 评论没有显示

| 可能原因 | 解决方法 |
|---|---|
| 仓库 Discussions 未开启 | 到 Settings > Features 勾选 Discussions |
| Giscus App 未安装 | 访问 github.com/apps/giscus 安装 |
| category_id 填错 | 到 giscus.app/zh-CN 重新获取正确值 |
| 构建未完成 | 等待 GitHub Actions 跑完（看仓库 Actions 标签页）|

### 图片上传失败

| 可能原因 | 解决方法 |
|---|---|
| Token 错误或过期 | 到 GitHub 重新生成 Token |
| 仓库名填错 | 确认格式是 `用户名/仓库名`，没有多余空格 |
| 网络问题 | 尝试切换网络或稍后再试 |
| 图片太大 | 压缩图片后再上传（建议单张不超过 2MB）|

---

## 总结

配置完评论和图床后，博客的写作体验提升了很多：

- **评论**：读者可以在文章底部直接留言讨论，数据保存在自己的 GitHub 仓库中
- **图床**：截图后一键上传，自动插入 CDN 链接，不用手动管理图片文件

这两件事的核心价值都在于：**把基础设施配置好，后面就能专注于内容本身，而不是反复处理技术细节。**
 
 