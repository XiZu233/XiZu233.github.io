---
title: "Agentic AI 学习笔记：M5 高自主性智能体模式"
date: 2026-05-27 10:00:00 +0800
categories: [AI, 学习笔记]
tags: [Agentic AI, Andrew Ng, Planning, Multi-Agent, Code Execution]
description: "整理 Andrew Ng Agentic AI 课程 M5 核心内容：Planning 规划工作流、Code Execution 代码执行、Multi-Agent 多智能体协作。"
---

# M5: Patterns for Highly Autonomous Agents

> 来源: agentic_workflows_M5_learner.pdf
> 讲师: Andrew Ng

> 💡 **小白注释**：M4 教的是"怎么发现和改进 AI 的问题"，M5 教的是"怎么让 AI 更聪明、更能自己干活"。核心思路：别什么事都手把手教 AI，让它自己定计划、自己写代码、甚至组建一个"AI 团队"分工协作。全文用 💡 标记的部分都是给新手看的通俗解释。

---

## 目录

1. 一、Planning Workflows 规划工作流
2. 二、Planning with Code Execution 用代码执行做规划
3. 三、Multi-Agentic Workflows 多智能体工作流
4. 四、Communication Patterns 通信模式
5. 五、全课总结

---

## 一、Planning Workflows 规划工作流

### 1.1 什么是 Planning

之前的模式里，人类把任务拆成固定的步骤（Step 1 → Step 2 → Step 3），AI 按顺序执行。而 **Planning 模式** 是让 AI **自己决定**需要哪些步骤、按什么顺序执行。

> 💡 **小白注释**：以前你是老板，每一步都写好指令让 AI 照做。现在你是董事长，只告诉 AI "去搞定这件事"，AI 自己当项目经理，自己排计划、自己调资源。

### 1.2 案例：客服 Agent

**用户提问**："有没有圆形太阳镜，价格在 100 美元以下，还有库存的？"

**AI 自己规划的三步计划**：
1. 用 `get_item_descriptions` 查找圆形太阳镜
2. 用 `check_inventory` 查库存
3. 用 `get_item_price` 确认价格是否低于 $100

![page 03](/assets/img/posts/agentic-workflows/M5/page_03.png)

**执行过程**：
- 系统提示告诉 AI 有哪些工具可用
- AI 先输出第一步计划 → 执行 → 拿到结果
- AI 根据结果输出第二步计划 → 执行 → 拿到结果
- AI 输出第三步计划 → 执行 → 给出最终回答

![page 04](/assets/img/posts/agentic-workflows/M5/page_04.png)

> 💡 **小白注释**：AI 不是一次性想好所有步骤然后一口气执行完，而是"想一步、做一步、看结果、再想下一步"。就像你导航去陌生地方，不是出发前一秒就把所有转弯都背下来，而是开一段、看导航、再开一段。

**另一个客服场景——退货**：

用户说："我想退金色框的眼镜，但金属框的不退。"

AI 自己规划：
1. 查历史交易 → 找到买过哪些眼镜
2. 找金色框的眼镜描述
3. 只退金色框的

![page 05](/assets/img/posts/agentic-workflows/M5/page_05.png)

### 1.3 案例：邮件助手

用户说："回复 Bob 关于纽约晚餐的邀请，说我参加，然后把邮件归档。"

AI 自己规划：
1. 用 `search_email` 搜 Bob 发的、提到 dinner 和 New York 的邮件
2. 用 `send_email` 回复确认参加
3. 用 `move_email` 移到归档文件夹

![page 06](/assets/img/posts/agentic-workflows/M5/page_06.png)

> 💡 **小白注释**：这三个任务的共同点是——用户只说了"想要什么结果"，没说"具体怎么做"。AI 自己分析需要哪些工具、按什么顺序用。这比人类预写死流程更灵活，因为不同用户的问题需要的步骤完全不同。

### 1.4 把计划格式化为 JSON

为了让 AI 的计划能被程序可靠解析，可以要求 AI 用 JSON 格式输出计划：

```json
{
  "plan": [
    {
      "step": 1,
      "description": "Find round sunglasses",
      "tool": "get_item_descriptions",
      "args": {"query": "round sunglasses"}
    },
    {
      "step": 2,
      "description": "Check available stock",
      "tool": "check_inventory",
      "args": {"items": "results from step 1"}
    }
  ]
}
```

![page 09](/assets/img/posts/agentic-workflows/M5/page_09.png)

> 💡 **小白注释**：JSON 就是一种机器能读懂的列表格式。把计划写成 JSON，程序就能自动解析每一步该调用哪个工具、传什么参数，不用靠猜。

---

## 二、Planning with Code Execution 用代码执行做规划

### 2.1 工具规划的局限

假设用户问："哪个月的热巧克力销量最高？"

如果用传统 Planning 模式，AI 可能生成这样的计划：
1. 用 `filter_rows` 筛选 1 月的热巧克力交易
2. 用 `get_column_mean` 算平均金额
3. 对 2 月重复一遍
4. 对 3 月重复一遍
5. ……对 12 月都重复一遍
6. 比较哪个月最高

![page 11](/assets/img/posts/agentic-workflows/M5/page_11.png)

**问题**：
- **脆弱（Brittle）**：如果数据格式变了，每一步都可能崩
- **低效（Inefficient）**：12 个月要重复 24 次工具调用
- **边缘 case 无穷无尽**：按周统计怎么办？按季度呢？前 10 名呢？

![page 12](/assets/img/posts/agentic-workflows/M5/page_12.png)

> 💡 **小白注释**：让 AI 一步步调用工具做数据分析，就像让一个人用计算器做Excel报表——按一下键看一眼结果再按一下。慢、容易错、遇到新需求就得重写流程。

### 2.2 解法：让 AI 写 Python 代码

与其让 AI 规划"调用什么工具"，不如让 AI 直接写 **Python 代码** 来解决问题。

**示例 1：查最近 5 笔交易的金额**

```python
<execute_python>
import pandas as pd
# Load CSV
df = pd.read_csv("transactions.csv")
# Ensure date column is parsed as datetime
df["date"] = pd.to_datetime(df["date"])
# Sort by date to get most recent transactions
df_sorted = df.sort_values(by="date", ascending=False)
# Select the last 5 transactions
last_5 = df_sorted.head(5)
# Show just the price column (amounts)
print("Last 5 transaction amounts:")
print(last_5["price"].to_list())
</execute_python>
```

![page 13](/assets/img/posts/agentic-workflows/M5/page_13.png)

**示例 2：上周有多少笔独立交易**

```python
<execute_python>
import pandas as pd
# Read CSV and parse the "date" column as datetime
df = pd.read_csv("transactions.csv", parse_dates=["date"])
# Define time window
today = pd.Timestamp.today()
week_ago = today - pd.Timedelta(days=7)
# Filter rows where date is within last week
last_week = df[df["date"].between(week_ago, today)]
# Drop duplicate rows and count
print(last_week.drop_duplicates().shape[0])
</execute_python>
```

![page 14](/assets/img/posts/agentic-workflows/M5/page_14.png)

> 💡 **小白注释**：让 AI 写代码就像雇了一个会编程的分析师。你问"上周卖了多少钱？"，他直接写一段 Python 代码跑一遍，出结果。不用你一步步告诉他"先打开文件、再找日期列、再筛选、再求和"。代码一次性把事情做完，又快又准。

### 2.3 代码规划 vs 工具规划的性能对比

研究表明，**用代码执行的 Agent 成功率显著高于只用工具调用的 Agent**。

![page 15](/assets/img/posts/agentic-workflows/M5/page_15.png)

> 💡 **小白注释**：论文出处《Executable Code Actions Elicit Better LLM Agents》(Wang et al. 2024)。简单说：让 AI 写代码自己跑，比让 AI 一步步按你定的规矩调用工具，效果好得多。代码是通用的，工具调用是局限的。

---

## 三、Multi-Agentic Workflows 多智能体工作流

### 3.1 为什么需要多个 Agent

有些任务太复杂，一个人搞不定，需要一个团队协作：

| 任务 | 团队角色 |
|------|---------|
| 制作营销素材 | 研究员、平面设计师、文案 |
| 写研究论文 | 研究员、统计师、主笔、编辑 |
| 准备法律案件 | 助理律师、律师助理、调查员 |

![page 17](/assets/img/posts/agentic-workflows/M5/page_17.png)

> 💡 **小白注释**：现实世界里，做一本宣传册需要好几个人：有人去调研市场、有人做图、有人写文案。AI 也一样——让一个 AI 又做研究又做设计又写作文，容易样样通样样松。不如让三个 AI 各专一项，像一个小团队。

### 3.2 案例：营销团队

三个 Agent 分工：

| Agent | 任务 | 工具 |
|-------|------|------|
| **研究员** | 分析市场趋势、研究竞争对手 | 网页搜索 |
| **平面设计师** | 做数据可视化、做配图 | 图片生成、代码生成图表 |
| **文案** | 把研究转成报告文本和营销文案 | 无 |

![page 18](/assets/img/posts/agentic-workflows/M5/page_18.png)

**线性执行流程**：
1. 用户："做一个太阳镜的夏季营销方案"
2. 研究员调研 → 输出趋势和竞品分析
3. 设计师基于研究结果 → 输出 5 张数据图 + 5 张配图
4. 文案基于研究 + 配图 → 输出最终报告

![page 19](/assets/img/posts/agentic-workflows/M5/page_19.png)

### 3.3 多 Agent 的规划模式

**模式一：单 Agent 规划 + 多 Agent 执行**

一个 LLM 充当"项目经理"，看完全局后制定计划，然后分配给不同 Agent 执行：
1. 让研究员调研太阳镜趋势
2. 让设计师做广告图
3. 让文案写报告
4. 项目经理审核报告

![page 23](/assets/img/posts/agentic-workflows/M5/page_23.png)

> 💡 **小白注释**：就像创业公司，CEO（项目经理 AI）想清楚战略，然后告诉 CTO（研究员）去调研、设计师去做图、文案去写稿。CEO 不亲自画图，但负责统筹。

**模式二：Manager Agent 模式**

系统提示把 Agent 定义为团队成员：

```
You are a marketing manager and have the following 
team of agents to work with:
{description of agents}
Return a step-by-step plan to carry out the user's request.
```

![page 22](/assets/img/posts/agentic-workflows/M5/page_22.png)

---

## 四、Communication Patterns 通信模式

### 4.1 线性模式（Linear）

信息单向流动：研究员 → 设计师 → 文案

每个 Agent 只接收上一个 Agent 的输出，完成后传给下一个。

![page 25](/assets/img/posts/agentic-workflows/M5/page_25.png)

> 💡 **小白注释**：像工厂流水线——A 做完交给 B，B 做完交给 C。简单、好管理，但缺点是设计师不能直接向研究员追问"这个数据我没看懂"。

### 4.2 全员互联（All-to-all）

每个 Agent 都可以和其他所有 Agent 直接通信。

![page 27](/assets/img/posts/agentic-workflows/M5/page_27.png)

> 💡 **小白注释**：像一个小办公室，谁都可以直接找谁聊天。研究员发现个好点子，可以直接告诉文案；文案写的时候缺张图，可以直接找设计师。沟通灵活，但也容易乱。

### 4.3 层级模式（Deeper Hierarchy）

多层管理结构，比如：
- 顶层：项目经理
- 中层：研究员、写手
- 底层：事实核查员、风格校对员、引用检查员

![page 27](/assets/img/posts/agentic-workflows/M5/page_27.png)

> 💡 **小白注释**：像大公司架构——写手写完后，不是直接交稿，而是先给事实核查员验真假、给风格校对员改语气、给引用检查员确认来源。每层都有专人把关，出错的概率更低。

---

## 五、全课总结

M5 的核心内容：**三种让 AI 更自主的模式**

| 模式 | 核心思想 | 适用场景 |
|------|---------|---------|
| **Planning** | AI 自己制定执行步骤 | 步骤不固定、需要灵活应变的任务 |
| **Code Execution** | AI 写代码解决问题 | 数据分析、需要循环/条件判断的复杂计算 |
| **Multi-Agent** | 多个 AI 分工协作 | 任务复杂、需要不同专长的场景 |

> 💡 **小白注释**：M5 的进化路线很清楚——> **从"人类写好每一步"到"AI 自己定计划"**> **从"一步步调用工具"到"直接写代码搞定"**> **从"一个 AI 干所有活"到"AI 团队分工协作"**

Andrew Ng 在课程最后给出的 Agentic AI 全貌：
- Why Agentic AI（为什么要做）
- Reflection design pattern（反思模式——M1）
- Tool use / Function calling（工具使用——M2）
- Evals, error analysis（评估与错误分析——M4）
- Planning, multi-agent systems（规划与多智能体——M5）

![page 29](/assets/img/posts/agentic-workflows/M5/page_29.png)

---

## 附录：全部幻灯片

| 页码 | 内容 |
|------|------|
| 1 | 封面: M5 Patterns for highly autonomous agents |
| 2 | 目录: Planning workflows |
| 3-6 | Planning 案例：客服 Agent、邮件助手 |
| 7-9 | Creating and executing LLM plans / JSON 格式化 |
| 10-15 | Planning with code execution / 性能对比 |
| 16-23 | Multi-agentic workflows / 营销团队案例 |
| 24-27 | Communication patterns（线性、全员互联、层级） |
| 28-29 | 总结: Agentic AI 全貌 |
| 30 | End of M5 |
