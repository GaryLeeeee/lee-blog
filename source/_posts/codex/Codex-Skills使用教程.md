---
title: Codex Skills使用教程：把重复工作变成可复用流程
date: 2026-09-18 15:39:00
tags: [Codex, Skills]
categories: [Codex]
---

如果说`AGENTS.md`是项目里的规章制度，那么Skill更像一份可以反复使用的“标准菜谱”。

例如，每次审查订单改动时，都要检查库存、支付、优惠券、消息和退款。把这套步骤做成Skill，以后只要说“按订单风险流程审查”，Codex就知道该怎么做。

## 一、哪些场景适合封装成Skill？

适合封装的任务通常有三个特点：**经常重复、步骤相对稳定、结果格式明确**。

常见场景包括：

- 按团队标准做代码审查。
- 根据固定结构整理接口文档。
- 把会议记录整理成结论、负责人和待办事项。
- 按模板生成周报、测试报告或发布说明。
- 固定顺序查询日志、数据库和监控数据。
- 批量处理PDF、表格、图片等文件。

> **商城案例**
>
> 团队每次上线订单功能前，都要检查状态流转、库存释放、支付幂等、优惠券返还和消息重复消费。这套检查顺序稳定，适合封装成`order-risk-review` Skill。

> **实际案例：开发排障文档**
>
> 每次排查后都要整理接口、Redis键、数据库表、计算公式、curl命令和风险点。如果只靠临时提示词，文档结构很容易变化；封装成Skill后，同类排障记录可以稳定使用一套目录和检查清单。

下面这些情况通常不值得封装：

- 只会执行一次的临时任务。
- 一句话就能说清楚的简单要求。
- 流程还在频繁变化，团队自己也没有统一做法。
- 只属于某个项目、每次都必须遵守的规则，这类内容更适合`AGENTS.md`。

## 二、Skill怎么调用？

### 1、显式调用

在Codex CLI或IDE中，可以输入`/skills`选择，也可以在提示词中使用`$技能名`：

```text
$order-risk-review 检查当前分支的订单改动，重点关注库存和退款。
```

在ChatGPT中通常使用`@`选择对应Skill。

显式调用适合这些情况：

- 你明确知道要用哪套流程。
- 多个Skill都可能匹配，想指定其中一个。
- Skill关闭了自动调用，只允许手动选择。

### 2、自动调用

如果用户请求与Skill的`description`匹配，Codex可以自动选择它。例如：

```text
帮我审查这次订单改动，检查可能的资金和库存风险。
```

能否正确自动调用，主要取决于`description`是否把“做什么”和“什么时候用”写清楚。

## 三、怎么创建Skill？

### 1、使用skill-creator

最省事的方式是在Codex中调用：

```text
$skill-creator

创建一个order-risk-review Skill。
用于审查商城订单相关改动，检查订单状态、库存、支付、退款、优惠券和消息消费。
输入是当前分支diff和相关代码，输出分为确定问题、潜在风险、验证建议。
发现业务规则不明确时先提问，不要自行猜测。
```

创建后，用一个真实任务试跑，再根据遗漏项调整。

> **官方案例：修复GitHub CI**
>
> OpenAI公开的`gh-fix-ci` Skill把“读取失败检查、定位日志、制定修复方案、修改并验证”做成固定流程。它解决的不是某一次CI错误，而是以后遇到同类失败都按相同步骤处理。

### 2、手工创建

项目内共享的Skill放在：

```text
项目根目录/.agents/skills/<skill-name>/SKILL.md
```

个人通用Skill可以放在：

```text
~/.agents/skills/<skill-name>/SKILL.md
```

如果不确定放哪里，直接让`$skill-creator`按当前环境创建即可。

一个完整目录可能是：

```text
order-risk-review/
├── SKILL.md
├── agents/
│   └── openai.yaml
├── references/
│   └── order-rules.md
├── scripts/
│   └── check-order-config.sh
└── assets/
    └── review-template.md
```

除`SKILL.md`外，其余目录都不是必需的：

- `references/`：放业务规则、接口说明等长资料。
- `scripts/`：放需要稳定重复执行的检查脚本。
- `assets/`：放输出时要使用的模板或素材。
- `agents/openai.yaml`：配置列表中的名称、简介、默认提示词和是否允许自动调用。

简单Skill只有一个`SKILL.md`就够了。

## 四、SKILL.md怎么写最好？

### 1、名称简单明确

文件夹名和`name`保持一致，只使用小写字母、数字和短横线：

```text
order-risk-review
```

### 2、description写清触发条件

不推荐：

```yaml
description: 帮助检查代码。
```

推荐：

```yaml
description: 审查商城订单、库存、支付和退款相关代码改动，并输出问题、风险和验证建议。用户要求检查订单链路、资金安全或上线风险时使用。
```

Codex平时只先看到Skill的名称和简介，因此简介含糊，自动调用就容易失准。

### 3、步骤写动作，不写长篇理论

```md
1. 读取当前分支diff，列出行为变化。
2. 从入口搜索调用方和下游消费者。
3. 检查并发、幂等、事务、重试和兼容性。
4. 运行相关测试并记录结果。
```

### 4、写清输入和输出

Skill要说明开始时需要什么，结束时交付什么。缺少必要输入时，应先提问，而不是编造。

### 5、长资料拆出去

`SKILL.md`只保留核心流程。订单状态说明放进`references/order-rules.md`，固定检查程序放进`scripts/`，不要把所有内容塞进一个文件。

## 五、可直接使用的Skill模板

在`.agents/skills/order-risk-review/SKILL.md`中写入：

```md
---
name: order-risk-review
description: 审查商城订单、库存、优惠、支付和退款相关代码改动，并输出确定问题、潜在风险和验证建议。用户要求检查订单链路、资金安全、库存一致性或上线风险时使用。
---

# 订单改动风险审查

## 需要的输入

- 当前分支diff或待审查文件。
- 需求说明和预期行为。
- 可用的测试命令。

缺少会影响判断的信息时，先列出问题，不要猜测业务规则。

## 执行步骤

1. 阅读需求和diff，列出本次行为变化。
2. 从接口入口搜索调用方、消息生产者和消费者。
3. 检查订单状态是否存在跳转遗漏或重复处理。
4. 检查库存锁定、释放和补偿是否成对出现。
5. 检查支付与退款的并发、幂等、事务和重试。
6. 检查优惠券返还、历史数据和旧客户端兼容性。
7. 运行相关测试；无法运行时说明原因。

## 输出格式

### 确定问题

- 只写能够从代码或测试确认的问题，并附文件和位置。

### 潜在风险

- 写明触发条件、影响范围和需要补充确认的内容。

### 验证结果

- 列出执行过的命令、结果和未覆盖范围。

## 边界

- 默认只审查，不修改代码。
- 不把个人偏好当成Bug。
- 不确定的业务规则交给用户确认。
- 不为了顺手优化而扩大审查范围。
```

调用示例：

```text
$order-risk-review 审查当前分支，需求是“取消已支付订单后自动退款并释放库存”。
```

## 六、agents/openai.yaml有什么用？

它主要控制Skill在界面里的展示和调用方式。通常让`$skill-creator`生成即可。

```yaml
interface:
  display_name: "订单风险审查"
  short_description: "检查订单、库存、支付和退款改动"
  default_prompt: "审查当前分支的订单链路风险"

policy:
  allow_implicit_invocation: true
```

如果设置为`false`，Codex不会自动调用，只能通过`$order-risk-review`显式使用。

## 七、Skill、Plugin和AGENTS.md有什么区别？

| 类型 | 通俗比喻 | 适合做什么 |
| --- | --- | --- |
| `AGENTS.md` | 项目规章制度 | 每次在项目中工作都要遵守的规则 |
| Skill | 一份标准菜谱 | 遇到某类任务时执行固定流程 |
| Plugin | 装好工具的工具箱 | 安装、分享一组Skill，并可连接外部服务 |

### Skill和AGENTS.md

`AGENTS.md`会在任务开始时随项目规则一起加载；Skill只在任务匹配或被明确调用时加载。

例如：

- “金额使用整数分”是长期项目规则，写进`AGENTS.md`。
- “审查订单改动的七个步骤”是一套任务流程，做成Skill。

详细配置可查看{% post_link codex/Codex-AGENTS使用教程 AGENTS.md使用教程 %}。

### Skill和Plugin

Skill主要解决“这件事应该按什么步骤做”。Plugin主要解决“如何把一组能力安装、连接并分享给别人”。

当一份Skill只给自己或当前项目使用时，直接保留Skill即可；需要发给团队安装、组合多个Skill，或者连接GitHub、Slack、Google Drive等外部服务时，再考虑Plugin。

> **实际案例：从Skill升级为Plugin**
>
> “按固定清单审查订单代码”只需要Skill；如果还要读取GitHub PR、查询线上监控并把结论发到Slack，就更适合做成Plugin，把审查Skill和外部工具连接一起打包。

## 八、创建Skill时要注意什么？

- **一个Skill只解决一类任务**：触发条件不同的流程应拆开。
- **简介比标题更重要**：简介决定Codex什么时候想到它。
- **先用文字，必要时再加脚本**：能可靠说明的流程，不必急着写程序。
- **脚本必须实际运行验证**：不能只看代码觉得可用。
- **不要放密码和令牌**：外部服务应使用正式连接和授权方式。
- **不要复制大段项目文档**：使用`references/`按需读取。
- **同时测试“应该触发”和“不该触发”**：避免什么任务都抢着执行。
- **更新后没有出现就重启Codex**：新任务通常会自动发现，异常时重启最直接。

建议至少测试四种请求：

```text
# 应该触发
帮我检查订单退款改动的风险。

# 换一种说法也应该触发
这次取消订单会不会导致库存或资金不一致？

# 信息不足，应先提问
帮我审查一下。

# 不应该触发
把商品详情页按钮改成“立即购买”。
```

## 九、总结

不要为了“看起来高级”而创建Skill。只有当一件事会重复发生，而且步骤和结果都比较稳定时，封装才真正省时间。

先从一个最常重复的任务开始，只写清触发条件、执行步骤、输出格式和边界；真实使用几次后，再补充资料或脚本。

## 参考资料

- [OpenAI：Build skills](https://learn.chatgpt.com/docs/build-skills)
- [OpenAI：Skills & Plugins](https://learn.chatgpt.com/docs/skills-and-plugins)
- [OpenAI Developers：Build skills](https://developers.openai.com/plugins/build/skills)
- [OpenAI Skills：gh-fix-ci案例](https://github.com/openai/skills/tree/main/skills/.curated/gh-fix-ci)
