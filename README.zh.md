# Visual Reporting Skill

一个 `SKILL.md`，让 agent 的报告——状态更新、测试结果、根因分析、架构说明——
变成结论行、表格、要点块和内联 ASCII 图，而不是大段文字。为查找而写，
不为叙事而写：读者直接跳到与自己相关的那一行。

[English](./README.md) | [繁體中文](./README.zh-TW.md) | 简体中文

提炼自真实的多仓库数据管线开发实践：同一个 agent 每天汇报部署、IAM 审计、
一致性校验和线上故障。

## 使用前后对比

同一模型（Haiku 4.5）、同样的事实输入，唯一区别是有没有这个技能。

**没有技能**：五段叙述文字，"全部通过"埋在第三段
（[完整样本](benchmarks/samples/status-report.baseline.md)）。

**有技能**（[完整样本](benchmarks/samples/status-report.skilled.md)）：

> **结论：** 全部测试通过 (180/180)。Schema 不匹配已用向后兼容读取修复。
>
> | 项目 | 详情 |
> |------|------|
> | 报错 | `KeyError: 'article_slug'` at `derive.py:84` |
> | 根因 | Manifest schema v2 把 `article_slug` 改名为 `slug` |
> | 修复 | 读 `slug`，回退 `article_slug` |

第三组对比是一条线上冒烟测试链路：六个事实写成段落约 122 token，画成一条
时序线只要约 70 token（**−43%**），每一跳、每个时间戳、退出码全在三行里
（[样本](benchmarks/samples/trigger-trace.timeline.md)）。

## Token 消耗——基本持平

关键数字：在上面的 A/B 实测中，**整轮 token 总量**（系统提示、输入、思考、
输出全算）差异只有 **+1.1% 和 +1.4%**。会话里 token 主要花在读文件、diff
和工具输出上，报告的形状是零头。

| 报告类型 | 无技能 | 有技能 | 差异 |
|---|--:|--:|--:|
| 链路时序线 vs 段落 | ~122 | ~70 | **−43%** |
| 状态/测试报告 | ~266 | ~229 | **−14%** |
| 架构讲解（大幅流程图） | ~514 | ~911 | **+77%** |
| 技能文本加载（仅报告触发时） | — | ~1.2k，每会话一次 | 200k 窗口的 0.6% |
| **整轮全量**（Haiku 4.5） | ~20.3k | ~20.6k | **+1.1% / +1.4%** |

规律：**结构删掉连接性废话，空白把它加回来。** 表格和紧凑时序线比段落省；
唯一更贵的是整页流程图——每次架构讨论只画一次，多花约 400 token，换来
读者再也不用重读。常用的省钱项和偶尔的花钱项在一个工作会话里互相抵消。

## 安装

**Claude Code 插件：**

```
/plugin marketplace add Edricklam123/visual-reporting
/plugin install visual-reporting@visual-reporting
```

**其他方式：** 直接让你的 AI 把技能文件复制到本地——比如
"把 Edricklam123/visual-reporting 的 SKILL.md 复制到我的 skills 目录"
（Claude Code 是 `~/.claude/skills/`，Cursor 是 `.cursor/rules/`，
其他 agent 放进它的规则文件）。就一个 markdown 文件，AI 知道放哪。

---

## 许可证

MIT
