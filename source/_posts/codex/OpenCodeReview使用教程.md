---
title: OpenCodeReview使用教程
date: 2026-09-21 12:01:22
tags: [Codex]
categories: [Codex]
---

OpenCodeReview是一个专门用于代码审查的命令行工具，命令简写为`ocr`。这里的OCR指OpenCodeReview，不是图片文字识别。

可以把它理解成一位“审查助理”：它先找出需要审查的文件，排除无关内容，匹配项目规则，再交给模型或当前编程Agent分析。

## 一、OpenCodeReview能做什么？

常见用法有两类：

- `ocr review`：审查Git工作区、分支差异或某个Commit。
- `ocr scan`：审查完整文件或目录，不依赖Git历史。

直接向通用AI说“帮我看下代码”，容易遇到这些问题：

- 改动较多时，部分文件没有被检查。
- 每次审查关注点不一致。
- 问题描述正确，但文件或行号对不上。
- 生成文件、依赖目录等噪声消耗大量上下文。

OpenCodeReview的作用不是保证“AI一定能找到所有Bug”，而是先把审查范围、文件过滤和规则匹配做得更稳定。

## 二、普通模式和委托模式有什么区别？

OpenCodeReview有两种审查方式：

| 对比项 | 普通模式 | 委托模式 |
| --- | --- | --- |
| 谁完成审查 | OCR配置的模型 | 当前Codex等编程Agent |
| OCR是否需要API Key | 需要 | 不需要 |
| 适合场景 | 命令行、CI、独立模型复查 | 在Codex、Cursor等Agent中交互式审查 |
| 模型费用 | 使用对应Provider的API额度 | 使用当前Agent的订阅或额度 |

### 1、普通模式

OCR自己调用配置好的模型，完成文件分组、代码分析和评论输出。这种方式适合放到脚本或CI中长期运行。

### 2、委托模式

OCR只负责找文件、排除无关内容和解析规则，真正的代码分析交给当前Codex。

> 委托模式不等于“不使用AI”，也不等于“完全本地运行”。代码仍会像普通Codex任务一样由Codex处理，只是OCR不再调用另一个外部模型。

如果日常已经在使用Codex，且没有单独的模型API Key，可以默认选择委托模式。

## 三、安装OCR CLI

OpenCodeReview要求Git版本不低于`2.41`：

```bash
git --version
```

使用NPM安装：

```bash
npm install -g @alibaba-group/open-code-review
```

macOS也可以通过Homebrew安装：

```bash
brew install open-code-review
```

验证安装：

```bash
ocr version
ocr --help
```

## 四、普通模式：让OCR自己调用模型

### 1、配置Provider和模型

```bash
ocr config provider
ocr config model
ocr llm test
```

`ocr config provider`会打开交互界面，选择OpenAI、DashScope、DeepSeek或自定义兼容端点等Provider，再填入对应的API Key。

选择时不用过度纠结，优先使用已经拥有、团队允许且网络稳定的Provider。如果没有独立API Key，就不必为了OCR额外申请，直接使用委托模式即可。

OCR默认把配置保存在：

```text
~/.opencodereview/config.json
```

这个文件可能包含API Key，不要复制到项目、提交到Git或放入文档。团队项目优先使用环境变量、密钥管理器或内部网关。

### 2、审查当前工作区

```bash
cd your-project
ocr review
```

工作区模式会检查当前仓库的已暂存、未暂存和未跟踪文件。

先预览哪些文件会被审查：

```bash
ocr review --preview
```

`--preview`不调用模型，适合在正式审查前确认范围和排除原因。

### 3、审查分支差异

```bash
ocr review --from main --to feature-branch
```

这会从`main`和`feature-branch`的合并基准开始，审查功能分支新增的改动。

### 4、审查单个Commit

```bash
ocr review --commit abc123
```

默认审查该Commit相对父提交的改动。Commit必须存在于当前Git仓库，否则会出现`not a valid commit ref`。

### 5、全量文件扫描

```bash
ocr scan
ocr scan --path internal/agent
```

`scan`审查完整文件，适合刚接手一个项目，或者需要检查没有Git diff的旧代码。它的范围通常更大，运行前最好先用`--path`限定目录。

### 6、恢复中断审查和导出结果

```bash
ocr session list
ocr review --from main --to feature-branch --resume <session-id>
ocr review --format json --output result.json
```

`review --resume`适用于分支区间或单Commit审查，不适用于随时可能变化的工作区。

## 五、委托模式：让当前Codex完成审查

### 1、安装Codex插件

先确保`ocr`CLI已经安装，再添加插件市场：

```bash
codex plugin marketplace add alibaba/open-code-review
```

打开Codex后，输入`/plugins`，安装并启用**Open Code Review**，然后新建一个任务。

新版Codex CLI也可以直接安装插件：

```bash
codex plugin add open-code-review-codex@open-code-review
```

插件刚安装完时，macOS上建议使用`Command + Q`完全退出Codex，重新打开后再新建任务。

### 2、用通俗的话调用

审查当前改动：

```text
@Open Code Review 用当前Codex直接审查当前改动，OCR只负责找改动，不调用外部模型，只报告问题。
```

审查当前分支相对`main`的改动：

```text
@Open Code Review 用当前Codex审查当前分支相对main的改动，不调用外部模型，只报告问题。
```

审查单个Commit：

```text
@Open Code Review 用当前Codex审查Commit abc123，不调用外部模型，只报告问题。
```

`Delegation Mode`只是官方模式名称，提示词不必强制使用这个英文。关键是说清楚：**由当前Codex审查，不调用外部模型。**

如果只说“审查当前改动”，插件可能选中普通模式并执行`ocr review`。没有配置Provider时，就会提示`no valid LLM endpoint configured`。

### 3、委托模式内部做了什么？

OCR主要提供两个底层命令：

```bash
ocr delegate preview --format json
ocr delegate rule --format json src/main.go src/handler.go
```

- `preview`：确定工作区、分支或Commit范围，并列出可审查文件。
- `rule`：找出每个文件应使用的审查规则。

这两个命令本身不会完成AI审查。安装插件后，应由Codex按顺序调用它们、读取diff并输出结论，日常使用时不需要手工拼这套流程。

## 六、一个完整的入门案例

假设商城项目正在开发`feature/cancel-order`分支，需求是：

> 用户取消已支付订单后，系统自动发起退款并释放库存。重复请求不能重复退款或重复释放库存。

### 使用普通模式

```bash
ocr review \
  --from main \
  --to feature/cancel-order \
  --background "取消已支付订单后自动退款并释放库存；必须防止重复退款和重复释放。"
```

### 使用委托模式

```text
@Open Code Review 用当前Codex审查feature/cancel-order相对main的改动，不调用外部模型。需求是取消已支付订单后自动退款并释放库存，重点检查并发、幂等、事务、重试和状态兼容性，只报告问题。
```

同一个审查目标，两种模式的区别只在于“谁负责真正分析”。业务背景、验收标准和重点风险都应说清楚。

## 七、配置项目审查规则

稳定、长期适用的项目规则，可以放在：

```text
<repo>/.opencodereview/rule.json
```

例如，给Hexo博客项目增加规则：

```json
{
  "exclude": ["public/**", "themes/matery/source/libs/**"],
  "rules": [
    {
      "path": "source/_posts/**/*.md",
      "rule": "检查Front Matter是否包含title、date、tags和categories；站内文章优先使用post_link；不编造引用和外部资料。",
      "merge_system_rule": true
    }
  ]
}
```

`merge_system_rule: true`表示在项目规则之外，仍保留OCR针对该文件类型的内置检查。

确认某个文件会命中哪条规则：

```bash
ocr rules check source/_posts/example.md
```

规则按声明顺序匹配，第一条命中的规则生效，因此更具体的路径应放在前面。规则只应记录稳定要求，一次性需求仍然放在本次审查的提示词中。

## 八、实际使用建议

### 1、Codex日常审查优先用委托模式

已经在Codex里开发时，委托模式的配置最少，而且Codex可以继续读取上下文、运行测试和追查调用链。

### 2、自动化和独立复查使用普通模式

需要在CI中自动运行，或希望用另一个模型作为独立审查者时，再配置Provider。

### 3、先报告问题，再决定是否修复

审查和修改同时进行，容易在还没看完所有文件时就改变diff。默认加上“只报告问题，不修改代码”，确认结论后再处理更稳妥。

### 4、重要改动单独开审查任务

文案、样式等小改动可以在原任务内检查。登录、权限、支付、数据库、部署或跨服务改动，建议新建一个Codex任务审查，减少原实现思路的影响。

### 5、大改动要拆分

审查几十个不相关文件，无论哪种模式都容易降低质量。按可验证的功能结果拆分分支或Commit，比在提示词里反复强调“仔细看”更有效。

### 6、审查不代替测试

OCR可以找风险，但不能证明程序已经正确运行。审查后仍要执行项目的构建、测试和关键业务验证。

### 7、注意代码发送范围

普通模式会把审查内容发给所配置的Provider；委托模式会由当前Codex处理。公司私有代码应先确认团队对外部模型、内部网关和代码出域的规定。

## 九、常见问题

### 1、提示没有配置LLM

说明当前执行的是普通模式。可以执行：

```bash
ocr config provider
ocr config model
ocr llm test
```

如果原本就想使用委托模式，在新任务中明确说“用当前Codex审查，不调用外部模型”。

### 2、Commit无法识别

OpenCodeReview只在当前Git仓库中查找Commit，不会扫描电脑上其他项目。可以依次检查：

```bash
git rev-parse --verify abc123^{commit}
git fetch origin --prune
git branch --all --contains abc123
```

仍然找不到时，通常是Commit属于另一个项目、未推送的本地分支，或Commit ID输入错误。

### 3、审查结果显示0个文件

先确认当前目录和Git状态：

```bash
git status --short
ocr review --preview
```

工作区干净，或文件被生成目录、二进制、用户规则等过滤时，都可能得到0个文件。

### 4、Codex设置中看不到插件

完全退出Codex后重新打开，然后新建任务。只关闭窗口不一定会结束后台进程。

### 5、macOS执行插件命令后显示`Killed`

先检查Codex CLI本身：

```bash
codex --version
```

如果这条命令也被系统终止，问题不在OpenCodeReview仓库，可先更新Codex CLI：

```bash
npm install -g @openai/codex@latest
```

更新后重新确认`codex --version`，再执行插件市场命令。

### 6、Git版本警告

如果OCR提示Git低于`2.41`，部分命令可能仍然能执行，但不在官方支持范围内。建议先升级Git，再把它放进稳定的审查流程。

## 十、总结

OpenCodeReview最有价值的部分，不是又多了一个“帮我看代码”的入口，而是把审查范围、文件过滤和项目规则固定下来。

实际使用时可以记住两句话：

- 已经在Codex中工作，优先使用委托模式。
- 需要命令行、CI或独立模型复查，使用普通模式。

还不熟悉Skill和Plugin的区别，可继续阅读{% post_link codex/Codex-Skills使用教程 Codex Skills使用教程 %}。

## 参考资料

- [OpenCodeReview官方仓库](https://github.com/alibaba/open-code-review)
- [OpenCodeReview中文说明](https://github.com/alibaba/open-code-review/blob/main/docs/i18n/README.zh-CN.md)
- [OpenCodeReview编程Agent插件说明](https://github.com/alibaba/open-code-review/blob/main/plugins/open-code-review/README.md)
- [OpenCodeReview委托模式](https://github.com/alibaba/open-code-review/blob/main/pages/src/content/docs/zh/integrations/delegate.md)
- [OpenCodeReview配置说明](https://github.com/alibaba/open-code-review/blob/main/pages/src/content/docs/zh/configuration.md)
- [OpenCodeReview评审规则](https://github.com/alibaba/open-code-review/blob/main/pages/src/content/docs/zh/review-rules.md)
