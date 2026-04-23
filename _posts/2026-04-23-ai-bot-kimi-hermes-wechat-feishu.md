---
title: 从零搭建 AI 机器人：Kimi Code + Hermes 实现微信&飞书双平台部署
date: 2026-04-23 12:00:00 +0800
categories: [教程, AI, 运维]
tags: [Kimi, Hermes, AI机器人, 微信机器人, 飞书机器人, Linux, Tailscale, SSH, 家用服务器]
---

> **前言**：这篇文章记录了我把一台闲置 ThinkPad 改造成 24 小时家用服务器，并在上面部署 Kimi Code + Hermes 实现微信和飞书双平台 AI 机器人的完整过程。全文分为两大部分：**服务器改造与网络配置**、**AI 机器人搭建与踩坑记录**。如果你只想看其中某一部分，可以直接点击下面的标签跳转。

---

## 内容速览

| 方向 | 内容 | 跳转 |
|:---|:---|:---|
| 服务器改造 | 闲置笔记本刷 Debian、硬件改造、网络配置、稳定性保障 | [点击跳转](#第一部分闲置电脑改造为家用服务器) |
| AI 机器人 | Kimi Code + Hermes 安装、微信/飞书配置、Gateway 运维 | [点击跳转](#第二部分-kimi-code--hermes-机器人搭建) |
| 踩坑合集 | 合盖断连、WiFi 丢失、硬盘挂载、模型冲突、PID 冲突等 | [服务器踩坑](#四踩坑记录核心板块) / [机器人踩坑](#四踩坑记录核心板块-1) |
| 附录 | 常用命令速查、参考文档 | [点击跳转](#附录常用命令速查) |

---

## 第一部分：闲置电脑改造为家用服务器

### 一、复盘：为什么改造

**背景**：我的主力机是一台轻薄本（AMD Ryzen 7 7735H），跑 AI 项目比较吃力，而云服务器长期租用成本又太高。

**方案**：把闲置的一台 ThinkPad X13（2021 款）改造成 24 小时运行的家用服务器，支持远程开发和服务部署。

| 项目 | 配置 |
|:---|:---|
| CPU | Intel i5-10代（4核8线程）|
| 内存 | 16G 板载（不可扩展）|
| 存储 | 256G SSD（系统）+ 2T 移动硬盘（数据）|
| 系统 | Debian（刷机安装）|
| 网络 | WiFi + 小米路由器 |

这台小黑原本是我大三时从海鲜市场淘来的备用机，忙完那段时间后就闲置了。现在想折腾 AI，主力电脑需要移动办公不适合长期部署，服务器包年又太贵，于是它又派上了用场。虽然配置不高，但用来部署 Agent 和跑一些 LLM Demo 完全够用。

---

### 二、教程：硬件改造

#### 1. 长期插电策略

24 小时不关机运行，电池自然损耗不可避免。建议：
- 保持电源适配器长期连接
- 在 BIOS 中如有电池保护模式可开启（部分机型支持充电阈值设置）
- 接受电池缓慢老化，这是改造成本的必要牺牲

#### 2. 合盖不睡眠、禁止休眠

服务器不需要屏幕，合盖后必须保持运行：

```bash
# 修改 systemd-logind 配置
sudo nano /etc/systemd/logind.conf
```

修改以下三项为 `ignore`：

```ini
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

> **注意**：修改后**不要**执行 `sudo systemctl restart systemd-logind`，否则会导致图形会话被杀死、黑屏光标闪烁。正确做法是保存后直接 `sudo reboot`，让配置下次开机生效。

彻底禁用睡眠功能：

```bash
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

验证：

```bash
systemctl status sleep.target  # 应显示 masked
```

#### 3. 移动硬盘挂载与自动挂载配置

闲置的移动硬盘可以作为数据盘使用。

```bash
# 创建挂载目录
sudo mkdir -p /mnt/data

# 修复未安全弹出的 NTFS 分区（如从 Windows 拔下）
sudo ntfsfix /dev/sda1

# 手动挂载
sudo mount -t ntfs-3g /dev/sda1 /mnt/data

# 开机自动挂载（追加到 fstab）
sudo bash -c 'echo "/dev/sda1 /mnt/data ntfs-3g defaults,auto,users,rw,nofail,umask=000 0 0" >> /etc/fstab'

# 验证配置
sudo mount -a
df -h /mnt/data
```

> **提示**：`nofail` 参数很重要，即使硬盘未连接也能正常开机，不会卡在启动环节。

---

### 三、教程：网络架构搭建

#### 1. 小米路由器静态 IP 绑定

在路由器后台 → DHCP 静态分配 → 绑定设备，给服务器分配固定内网 IP，避免重启后 IP 变化导致 SSH 配置失效。

![小米路由器后台 DHCP 静态分配](https://cdn.jsdelivr.net/gh/XiZu233/images@main/blog/20260423144124.png)

![绑定设备结果](https://cdn.jsdelivr.net/gh/XiZu233/images@main/blog/20260423132053.png)

#### 2. Tailscale 虚拟组网（解决无公网 IP）

没有公网 IP 的情况下，Tailscale 是最省心的内网穿透方案。

**Linux 服务器端**：

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

**Windows 客户端**：

官网下载：https://tailscale.com/download/windows

两边用同一个账号登录后验证：

```bash
tailscale status     # 两边都显示 Connected
tailscale ip -4      # 查看分配的虚拟 IP
```

![Tailscale 虚拟组网连接状态](https://cdn.jsdelivr.net/gh/XiZu233/images@main/blog/20260423144154.png)

#### 3. SSH 服务配置

```bash
sudo apt update
sudo apt install -y openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh  # 确认 active (running)
```

**VS Code Remote-SSH 配置**

编辑 `C:\Users\用户名\.ssh\config`：

```ssh
Host debian-server
    HostName 192.168.31.93
    User wenchen
    Port 22

Host debian-tailscale
    HostName 100.78.131.56
    User wenchen
    Port 22
```

**免密登录**：

```bash
# Windows 端生成密钥
ssh-keygen -t rsa -b 4096

# 复制公钥到服务器
ssh-copy-id wenchen@192.168.31.93
```

> **提示**：配置好免密登录后，VS Code 一键远程开发体验非常顺滑。除此以外 WSL2 + VS Code 也是 Windows 上使用 Linux 的好方法，推荐理工科学生掌握，本文不赘述。

---

### 四、踩坑记录（核心板块）

| 坑 | 现象 | 根因 | 解决 |
|:---|:---|:---|:---|
| WiFi 配置丢失 | 重启后无网络 | 误删 NetworkManager 配置 | 恢复 `managed=true` |
| 合盖断连 | SSH 中断 | `systemd-logind` 重启导致 | 改为 `ignore`，不重启服务 |
| 图形界面崩溃 | 黑屏+光标闪烁 | logind 重启会杀掉图形会话 | 纯命令行操作 |
| 移动硬盘无法挂载 | `ntfs-3g` 报错 | 未安全弹出+目录不存在 | `ntfsfix` + `mkdir` |
| fstab 配置错误 | 系统无法启动 | 格式错误 | 恢复备份 |
| WiFi 接口 DOWN | 扫描不到网络 | NetworkManager 未托管 | 修改 `NetworkManager.conf` |

#### 坑 1：合盖后 SSH 断开

已在上文[硬件改造](#2-合盖不睡眠禁止休眠)部分说明，核心结论是：**修改 `logind.conf` 后直接 `reboot`，不要 restart 服务**。

#### 坑 2：睡眠状态理解错误

现象：`systemctl status sleep.target` 显示 `inactive (dead)`，以为是已禁用。

![sleep.target 显示 inactive](https://cdn.jsdelivr.net/gh/XiZu233/images@main/blog/20260423144339.png)

根因：`inactive` 只是当前没睡眠，功能仍可用。必须 `masked` 才是真正禁用。

#### 坑 3：重启后 WiFi 消失

现象：`ip link show` 显示 `wlp0s20f3` 接口存在但状态 `DOWN`，`nmcli device wifi list` 空白。

![WiFi 接口 DOWN 状态](https://cdn.jsdelivr.net/gh/XiZu233/images@main/blog/20260423144357.png)

排查：

```bash
lsmod | grep iwlwifi      # 驱动已加载
nmcli device status       # 查看设备状态
```

![nmcli 设备状态](https://cdn.jsdelivr.net/gh/XiZu233/images@main/blog/20260423144416.png)

根因：`/etc/NetworkManager/NetworkManager.conf` 中 `[ifupdown]` 的 `managed=false`。

![managed=false 配置项](https://cdn.jsdelivr.net/gh/XiZu233/images@main/blog/20260423144432.png)

解决：

```bash
sudo nano /etc/NetworkManager/NetworkManager.conf
# 改为 managed=true
sudo systemctl restart NetworkManager
nmcli device wifi list    # 正常显示 WiFi 列表
```

#### 坑 4：移动硬盘挂载失败

现象：`sudo mount -t ntfs-3g /dev/sda1 /mnt/data` 报错。

![挂载报错信息](https://cdn.jsdelivr.net/gh/XiZu233/images@main/blog/20260423144521.png)

根因与解决已在[上文](#3-移动硬盘挂载与自动挂载配置)详述，主要是目录不存在 + 未安全弹出导致。

---

### 五、教程：系统稳定性保障

#### 关键服务开机自启

```bash
sudo systemctl enable ssh
sudo systemctl enable tailscaled
# sudo systemctl enable docker  # 如需 Docker 后续再开启
```

#### 自动更新

```bash
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades  # 选择 Yes
```

#### 日志限制

避免日志无限增长撑爆系统盘：

```bash
sudo mkdir -p /etc/systemd/journald.conf.d
sudo bash -c 'cat > /etc/systemd/journald.conf.d/99-limit.conf << EOF
[Journal]
SystemMaxUse=500M
MaxFileSec=1week EOF'
```

> **提示**：尽可能让服务器保持"无感"或透明化，意外重启后能自动恢复所有服务，减轻维护负担。

---

## 第二部分：Kimi Code + Hermes 机器人搭建

### 一、复盘：为什么选这个方案

- **求职需要**：积累 AI 项目实战经验
- **实际需求**：想要一个微信/飞书机器人辅助日常
- **工具顺手**：已有 Kimi Code 付费套餐（大杯），适合编程场景
- **框架成熟**：Hermes 框架支持多平台接入，中文社区文档完善

> **提示**：这里的 Kimi Code 可以替换为 Claude Code、Codex、OpenCode 等其他 AI 编程工具，手上有啥用啥。微信/飞书/其他平台同理，Hermes 官方文档和中文社区都有对应 Wiki。

---

### 二、教程：Kimi Code 配置

#### 安装 uv（Python 包管理器）

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
export PATH="$HOME/.local/bin:$PATH"
```

#### 安装 kimi-cli

```bash
uv tool install kimi-cli
kimi --version
```

#### 登录配置

```bash
kimi login
# 选择 1. Kimi Code
# 输入 API Key（从 https://code.kimi.com/ 获取）
```

安装完成后可以网页跳转扫码登录，或直接从控制台复制 API Key 粘贴。配置好 Kimi Code 后，可以进一步搭配 Hermes 增强 Agent 功能。

---

### 三、教程：Hermes 安装与基础配置

#### 官方安装脚本

```bash
curl -L code.kimi.com/install.sh | bash
export PATH="$HOME/.local/bin:$PATH"
```

常规安装跟着[官方 Wiki](https://hermes-doc.aigc.green/)的快速开始即可，四级英语水平足够完成。更有趣的是，配置好 Kimi Code 后，可以让 AI 帮你执行 Hermes 的安装命令，后续配置机器人时代劳，你只需要给权限、`yes`、掏出手机扫码、在飞书里给权限即可。

"请把这个 MCP server 加到你的配置里：https://hermesagent.org.cn/mcp （Streamable HTTP，无需 API Key、无需登录）。加完后用它帮我查 Hermes Agent 中文文档来指导我完成安装。"

执行上面的提示词即可完成 Hermes 的安装。很 easy，就是废 token。

---

### 四、踩坑记录（核心板块）

| 坑 | 现象 | 根因 | 解决 |
|:---|:---|:---|:---|
| 模型指向 Gemini | Provider: gemini, Model: kimi-for-coding | 残留 GEMINI 配置 | 清空 `GEMINI_API_KEY`，强制设置 `provider=kimi` |
| QQ Bot 401 | invalid appid or secret | AppID/Secret 错误 | 核对开放平台配置 |
| 微信连接超时 | Connection timeout | 网络不通或平台选错 | 切换为 Kimi Code 平台 |
| 配对码过期 | Code not found or expired | 超时未审批 | 重新发起配对 |
| 飞书未加载 | `lark-oapi` not installed | 未安装 SDK | `pip3 install lark-oapi` |
| 飞书 No home channel | 提示未设置 | 未执行 `/sethome` | 聊天中发送 `/sethome` |
| Gateway PID 冲突 | PID file race | 旧进程未清理 | `pkill` + 删除 pid 文件 |
| API 404 | Gemini HTTP 404 | 模型不存在 | 确认模型名称正确 |

#### 坑 1：模型指向 Gemini

现象：请求显示 `Provider: gemini, Model: kimi-for-coding`，报错 `GeminiAPIError [HTTP 404]`。

根因：残留 `GEMINI_API_KEY` 配置，Hermes 优先使用 Gemini provider。

排查：

```bash
hermes config show | grep -E "model|provider"
# 显示 provider 为 gemini
```

解决：

```bash
# 强制设置 Kimi
hermes config set model kimi-for-coding
hermes config set provider kimi

# 清空冲突配置
hermes config set GEMINI_API_KEY ""
hermes config set ANTHROPIC_API_KEY ""
hermes config set ANTHROPIC_TOKEN ""

# 验证
hermes config show
# 应显示 Model: {'default': 'kimi-for-coding', 'provider': 'kimi'}
```

> 我一般用两个 LLM：Kimi 和 Gemini。配置 Kimi 后想再加 Gemini，结果后续模型都指向了 Gemini，加上当时没配代理，Gemini 在国内网络用不了。干脆清空 Gemini 的 API Key 继续用 Kimi，大杯 Kimi 也足够用了。

#### 坑 2：Gateway PID 冲突

现象：

```
ERROR gateway.run: PID file race lost to another gateway instance. Exiting.
```

解决：

```bash
pkill -f "hermes gateway"
rm -f ~/.hermes/*.pid
sleep 2
hermes gateway run
```

#### 坑 3：飞书权限问题

飞书除了创建应用、获取 AppID 等信息外，**必须在权限管理中开启消息读取权限**，否则机器人只能给你发消息，无法接收你的回复，也发不了配对码。开启权限后，在聊天中发送 `/sethome` 设置主频道即可正常对话。

![飞书权限配置](https://cdn.jsdelivr.net/gh/XiZu233/images@main/blog/20260423145138.png)

---

### 五、教程：微信机器人完整配置

微信机器人与飞书机器人在 Hermes 中的配置流程基本一致，差异点：
- 微信机器人**不能加入群聊**
- 需要在微信开放平台注册、配置环境变量、完成用户配对流程
- 日常个人聊天推荐微信

**配置步骤**：

1. 前往微信开放平台注册应用，获取 AppID 和 AppSecret
2. 在 Hermes 中配置微信环境变量：`WX_APP_ID` 和 `WX_APP_SECRET`
3. 启动 Gateway 后，按提示完成用户配对（手机微信扫码或输入配对码）
4. 配对成功后即可在私聊中与机器人对话

**注意事项**：
- 微信机器人仅限个人对话，无法接入群聊
- 如果配对码过期，重新发起配对即可
- 遇到连接超时时，检查网络连通性并确认平台配置正确

---

### 六、教程：飞书机器人完整配置

飞书机器人配置要点：
1. 创建企业应用
2. 开启机器人能力
3. 配置事件订阅（Event URL）
4. **关键**：在权限管理中开启 `im:chat:readonly` 和 `im:message:send_as_bot` 等消息读取权限（否则单向通讯）
5. 将机器人加入群聊或私聊，发送 `/sethome` 设置主频道
6. 在 Hermes 中配置 `LARK_APP_ID` 和 `LARK_APP_SECRET`

飞书机器人**可以参与群聊**（局限于飞书内部），如果希望将 AI 接入工作流、文档管理、日程任务等场景，推荐飞书。

> **注意**：飞书**不需要**单独安装 `lark-oapi` SDK（这是 AI 幻觉），按官方 Wiki 配置即可。

---

### 七、教程：Gateway 运行与维护

#### 前台调试

```bash
hermes gateway run
```

#### 后台持久化（tmux）

如果需要频繁开关机又想保持 Bot 在线，建议配置后台持久化：

```bash
# 创建 tmux 会话
tmux new -s hermes
hermes gateway run
# Ctrl+B, D 分离会话

# 重新 attach
tmux attach -t hermes
```

更省心的方式是配置系统服务实现开机自启，或者使用 `systemd` 管理 Gateway 进程。

#### 日志查看

Gateway 运行时的日志会直接输出在终端，后台运行时可定向到日志文件排查问题。

---

### 八、复盘：整体架构与收获

#### 架构图

```
[用户微信/飞书]
       |
[微信服务器 / 飞书服务器]
       |
[Hermes Gateway] <- 运行在 Debian 服务器
       |
[Kimi API] <- 调用 AI 能力
       |
[回复用户]
```

```
[开发者 Windows] <- Tailscale -> [Debian 服务器 100.78.131.56]
       |
[VS Code Remote-SSH 远程开发]
```

#### 按照 STAR 法则复盘

- **Situation**：频繁开关机移动办公导致 Agent 部署不稳定，与机器人 24h 不间断的需求冲突；结合闲置设备和网络条件，决定改造旧电脑。
- **Task**：搭建一台可 24 小时运行、可远程访问的家用小服务器，并部署 AI 机器人。
- **Action**：
  - 将 ThinkPad 从 Windows 更换为 Debian
  - 修改电源配置、关闭屏幕、合盖不关机
  - 安装 SSH 服务、配置 Tailscale 虚拟局域网
  - 配置网络代理/VPN 访问外部网络
  - 部署 Kimi Code + Hermes，完成微信和飞书机器人配置
  - 完成简单的晨间新闻简报案例
- **Result**：双平台机器人（飞书+微信）实验成功；拥有了一台可远程管理的家用服务器。

#### Bot 运行截图

![Bot 对话效果](https://cdn.jsdelivr.net/gh/XiZu233/images@main/blog/20260423145258.png)

![机器人运行状态](https://cdn.jsdelivr.net/gh/XiZu233/images@main/blog/20260423145314.png)

---

### 九、扩展：知识库与工作流

除了基础的聊天机器人，还可以结合 Obsidian 和 LLM Wiki 搭建个人知识库，将 Kimi + Hermes 的能力延伸到知识管理和自动化工作流中。例如：

- 利用 Obsidian 管理本地笔记，通过 LLM Wiki 插件与 Kimi API 联动，实现笔记智能检索和总结
- 配合飞书机器人，将工作群聊中的关键信息自动归档到知识库
- 搭建定时任务，让机器人每日推送整理后的行业资讯或会议纪要

具体实现方法可以参考文末的参考链接，其中包含了完整的 Kimi + Hermes + Obsidian 搭建教程。

---

## 附录：常用命令速查

### 网络

```bash
ip addr show
nmcli device wifi list
tailscale status
tailscale ip -4
```

### SSH

```bash
sudo systemctl status ssh
ssh wenchen@100.78.131.56
```

### Hermes

```bash
hermes config show
hermes pairing list
hermes gateway run
```

### 系统

```bash
df -h
free -h
sudo apt update && sudo apt upgrade
```

---

## 参考链接

- [Hermes Agent 中文文档](https://hermes-doc.aigc.green/)
- [Hermes Agent 快速开始](https://hermesagent.org.cn/docs/getting-started/quickstart)
- [Hermes Agent 中文社区](https://hermesagent.org.cn/)
- [飞书接入 Hermes 官方文档](https://www.feishu.cn/content/article/7628541877674953666)
- [让 AI 成为你的飞书同事！Hermes Agent 接入飞书完全教程 - 知乎](https://zhuanlan.zhihu.com/p/2028895886046971029)
- [完整利用 Kimi + Hermes + Obsidian 搭建知识库](https://datawhaler.feishu.cn/wiki/YesRwdINviiqPbkRkHlcGwuqnWe)
- [Kimi AI 官网](https://www.kimi.com/)
- [Tailscale 下载](https://tailscale.com/download)
