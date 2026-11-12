# pre-workout

English | [简体中文](README.zh-CN.md)

A Claude Code skill to warm up a task before you run it. It optimizes your draft prompt first, then grills you one question at a time on the assumptions it had to make, and finally gives you a ready-to-paste prompt built on decisions you confirmed.

## The problem

A rough prompt like "build a login page" hides a dozen decisions: auth method, token storage, redirects, rate limiting, tests, what not to touch. Prompt optimizers usually fill these with silent defaults; interview-style skills ask generic questions with no draft to anchor them. Either way, the task stalls halfway through or goes in the wrong direction.

pre-workout combines the two in a fixed order: **optimize first, then grill**. The optimized draft makes every hidden decision visible, so the questions target the decisions that matter for this task.

## How it works

| Stage | What happens |
|---|---|
| A. Optimize | Detects the project and tech stack, classifies intent and scope, matches ECC commands, skills and agents, and writes a **v1 prompt**. Every gap is filled with a labeled default (`[A1]`, `[A2]`…) and listed in an assumption table ordered by dependency. |
| B. Grill | Goes through the assumptions **one question per message**, each with a recommended answer and alternatives. Upstream decisions first; anything the codebase can answer is looked up, not asked; each answer prunes or adds branches. Reply `ok`, your own answer, `skip`, or `enough` / `够了`. Stops at 12 questions by default. |
| C. Finalize | A decision log (detected / confirmed / assumed), the final full prompt with "confirmed decisions" and "remaining assumptions" sections, a quick version, and a change list against your draft. |

Small tasks with no critical assumptions skip the grill. Saying "no questions" / "别问了" skips it too.

The skill is **advisory only**: it never writes code or carries out the task itself.

## Install

```bash
git clone https://github.com/Czwu013/pre-workout ~/.claude/skills/pre-workout
```

The folder name must match the `name` field in `SKILL.md` (`pre-workout`). To use it in a single project only, put it under that project's `.claude/skills/pre-workout/` instead.

## Usage

```
/pre-workout 帮我写一个用户登录页面
```

It also triggers on phrases like "optimize this prompt", "grill me on this", "优化prompt", "问清楚再写".

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | The skill; loaded when it triggers |
| `README.md` / `README.zh-CN.md` | For humans; not loaded by Claude automatically |

## Known limitations

- The component tables assume the [ECC](https://github.com/affaan-m/everything-claude-code) ecosystem. The skill only recommends components that are installed and falls back to plain-language steps otherwise, but the output is most useful with ECC set up.
- Intents and the gap checklist are software-development oriented; non-coding prompts (writing, research reports) are not well covered yet.
- The 12-question cap and the "skip grill for TRIVIAL/LOW" rule are rule-of-thumb defaults.
- Trigger accuracy and output quality have not been measured against real prompts.

## Credits

- Stage A is adapted from the `prompt-optimizer` skill by YannJY02 (ECC community).
- Stage B follows the approach of Matt Pocock's [`grill-me`](https://github.com/mattpocock/skills) skill: one question at a time, always recommend an answer, explore the codebase instead of asking.

## License

[MIT](LICENSE)
