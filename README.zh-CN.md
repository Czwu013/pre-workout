# pre-workout

[English](README.md) | 简体中文

一个在正式执行任务前"热身"的 Claude Code skill：先优化你的 prompt 草稿，再针对它不得不做的假设一次一个问题地拷问你，最后给出一份基于你确认过的决策、可以直接粘贴的 prompt。

## 解决什么问题

"帮我写一个登录页面"这样的草稿背后藏着十几个决定：认证方式、token 存哪、跳转到哪、要不要限流、怎么测、哪些不能动。普通的 prompt 优化器会悄悄填上默认值；访谈式的 skill 又没有草稿做锚点，只能问泛泛的问题。结果都是任务做到一半卡住，或者方向跑偏。

pre-workout 把两者按固定顺序结合：**先优化，再拷问**。优化后的草稿让每个隐藏的决定都摆到台面上，拷问就能只问真正影响这个任务的问题。

## 工作流程

| 阶段 | 做什么 |
|---|---|
| A. 优化 | 检测项目和技术栈，判断意图和规模，匹配 ECC 的命令、skill 和 agent，写出 **v1 prompt**。每个缺口都填上带标签的默认值（`[A1]`、`[A2]`…），并按依赖关系列成假设表。 |
| B. 拷问 | 逐条过假设表，**每条消息只问一个问题**，附推荐答案和备选项。上游的决定先问；能从代码库查到的自己查，不问你；每个回答都会剪掉或新增分支。可以回 `ok`、你自己的答案、`skip` 或 `够了`。默认最多 12 题。 |
| C. 定稿 | 决策记录（已检测 / 已确认 / 仍是假设）、包含「已确认的决策」和「假设」两部分的完整 prompt、精简版，以及相对草稿的改动清单。 |

没有关键假设的小任务会跳过拷问；说"别问了"也会跳过。

这个 skill **只给建议**，不会写代码，也不会替你执行任务。

## 安装

```bash
git clone https://github.com/Czwu013/pre-workout ~/.claude/skills/pre-workout
```

文件夹名要和 `SKILL.md` 里的 `name`（`pre-workout`）一致。如果只想在某个项目里用，可以放到该项目的 `.claude/skills/pre-workout/`。

## 使用

```
/pre-workout 帮我写一个用户登录页面
```

说"优化prompt"、"问清楚再写"、"optimize this prompt"、"grill me on this"之类的话也会触发。

## 文件说明

| 文件 | 作用 |
|---|---|
| `SKILL.md` | skill 本体，触发时加载 |
| `README.md` / `README.zh-CN.md` | 给人看的说明，Claude 不会自动加载 |

## 已知限制

- 组件表基于 [ECC](https://github.com/affaan-m/everything-claude-code) 生态。skill 只会推荐已安装的组件，没装的会改用文字描述步骤，但装了 ECC 效果最好。
- 意图分类和缺口清单偏向软件开发，写作、调研报告这类非编码 prompt 还覆盖得不好。
- "最多 12 题"和"TRIVIAL/LOW 跳过拷问"是经验值，可按需调整。
- 触发准确率和输出质量还没有用真实 prompt 测过。

## 致谢

- A 阶段改编自 YannJY02 的 `prompt-optimizer` skill（ECC 社区）。
- B 阶段沿用 Matt Pocock 的 [`grill-me`](https://github.com/mattpocock/skills) skill 的做法：一次一个问题、每题都给推荐答案、能查代码库就不问。

## License

[MIT](LICENSE)
