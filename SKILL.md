---
name: pre-workout
description: Warm up a task before running it: optimize the draft prompt, grill the user one question at a time on its assumptions, then output a final ready-to-paste prompt. Advisory only.
---

# Pre-Workout: Prompt Optimizer + Grill

Turn a rough draft prompt into a final, ready-to-paste prompt in three stages:

1. **Optimize**: analyze the draft and produce an optimized **v1 prompt** in which every gap is filled with an explicit, labeled assumption.
2. **Grill**: interview the user **one question at a time** about those assumptions until every decision that matters is confirmed.
3. **Finalize**: output the final prompt, built on confirmed decisions.

The order matters. The v1 prompt makes every hidden decision visible, so the grill targets the real decisions in this task instead of asking generic questions.

## When to Use

- User invokes `/pre-workout`, or says "pre-workout this"
- User says "optimize this prompt", "improve my prompt", "rewrite this prompt", "help me write a better prompt for..."
- User says "优化prompt", "改进prompt", "怎么写prompt", "帮我优化这个指令"
- User says "grill me on this", "问清楚再写", "拷问我"
- User pastes a draft prompt and asks for feedback or enhancement
- User says "I don't know how to prompt for this"
- User says "what's the best way to ask Claude Code to..." or "how should I use ECC for..."

### Do Not Use When

- User wants the task done directly ("just do it", "直接做")
- User says "优化代码", "优化性能", "optimize this code", "optimize performance". These are refactoring tasks, not prompt work.
- User is asking about ECC configuration (use `configure-ecc`) or wants a skill inventory (use `skill-stocktake`)

## Ground Rules

- **Advisory only.** Do NOT write code, create files, run commands that change anything, or carry out the user's task. Read-only checks (reading CLAUDE.md or project files) are allowed. If the user says "just do it" or "don't optimize, just execute", tell them this skill only produces prompts and that they should make a normal task request if they want it executed.
- Respond in the same language as the user's input.
- **Recommend only what exists.** Before you name an ECC command, skill or agent, check that it is available in this environment (use the session's skill and command list, or `skill-stocktake` if installed). If it is missing, describe the step in plain words instead, e.g. "write failing tests first" instead of `/tdd`.
- Ask questions about the **task**, not about the prompt's wording.

---

## Stage A: Optimize

### Phase 0: Project Detection

1. Read `CLAUDE.md` in the working directory if it exists.
2. Detect the tech stack from project files:
   - `package.json` → Node.js / TypeScript / React / Next.js
   - `go.mod` → Go
   - `pyproject.toml` / `requirements.txt` → Python
   - `Cargo.toml` → Rust
   - `build.gradle` / `pom.xml` → Java / Kotlin (look for `quarkus` → Quarkus, or `spring-boot` → Spring Boot)
   - `Package.swift` → Swift
   - `Gemfile` → Ruby
   - `composer.json` → PHP
   - `*.csproj` / `*.sln` → .NET
   - `Makefile` / `CMakeLists.txt` → C / C++
   - `cpanfile` / `Makefile.PL` → Perl
3. If nothing is found and the task is about an existing project, record "tech stack unknown" as a **critical assumption**. Stage B will ask for the stack, or ask the user to connect the project folder. For an abstract prompt or a new project, note that there is no project context.

### Phase 1: Intent Detection

Classify the task into one or more categories:

| Category | Signal Words | Example |
|----------|-------------|---------|
| New Feature | build, create, add, implement, 创建, 实现, 添加 | "Build a login page" |
| Bug Fix | fix, broken, not working, error, 修复, 报错 | "Fix the auth flow" |
| Refactor | refactor, clean up, restructure, 重构, 整理 | "Refactor the API layer" |
| Research | how to, what is, explore, investigate, 怎么, 如何 | "How to add SSO" |
| Testing | test, coverage, verify, 测试, 覆盖率 | "Add tests for the cart" |
| Review | review, audit, check, 审查, 检查 | "Review my PR" |
| Documentation | document, update docs, 文档 | "Update the API docs" |
| Infrastructure | deploy, CI, docker, database, 部署, 数据库 | "Set up CI/CD pipeline" |
| Design | design, architecture, plan, 设计, 架构 | "Design the data model" |

### Phase 2: Scope Assessment

If Phase 0 detected a project, use codebase size as a signal. Otherwise estimate from the prompt and mark the estimate as uncertain.

| Scope | Heuristic | Orchestration |
|-------|-----------|---------------|
| TRIVIAL | Single file, < 50 lines | Direct execution |
| LOW | Single component or module | Single command or skill |
| MEDIUM | Multiple components, same domain | Command chain + /verify |
| HIGH | Cross-domain, 5+ files | /plan first, then phased execution |
| EPIC | Multi-session, multi-PR, architectural shift | blueprint skill for a multi-session plan |

### Phase 3: Component Matching

Match intent, scope and tech stack to components, and keep only the ones that are available (see Ground Rules).

#### By Intent Type

| Intent | Commands | Skills | Agents |
|--------|----------|--------|--------|
| New Feature | /plan, /tdd, /code-review, /verify | tdd-workflow, verification-loop | planner, tdd-guide, code-reviewer |
| Bug Fix | /tdd, /build-fix, /verify | tdd-workflow | tdd-guide, build-error-resolver |
| Refactor | /refactor-clean, /code-review, /verify | verification-loop | refactor-cleaner, code-reviewer |
| Research | /plan | search-first, iterative-retrieval | — |
| Testing | /tdd, /e2e, /test-coverage | tdd-workflow, e2e-testing | tdd-guide, e2e-runner |
| Review | /code-review | security-review | code-reviewer, security-reviewer |
| Documentation | /update-docs, /update-codemaps | — | doc-updater |
| Infrastructure | /plan, /verify | docker-patterns, deployment-patterns, database-migrations | architect |
| Design (MEDIUM-HIGH) | /plan | — | planner, architect |
| Design (EPIC) | — | blueprint (invoke as skill) | planner, architect |

#### By Tech Stack

| Tech Stack | Skills to Add | Agent |
|------------|--------------|-------|
| Python / Django | django-patterns, django-tdd, django-security, django-verification, python-patterns, python-testing | python-reviewer |
| Go | golang-patterns, golang-testing | go-reviewer, go-build-resolver |
| Spring Boot / Java | springboot-patterns, springboot-tdd, springboot-security, springboot-verification, java-coding-standards, jpa-patterns | java-reviewer |
| Quarkus / Java | quarkus-patterns, quarkus-tdd, quarkus-security, quarkus-verification, java-coding-standards, jpa-patterns | java-reviewer |
| Kotlin / Android | kotlin-coroutines-flows, compose-multiplatform-patterns, android-clean-architecture | kotlin-reviewer |
| TypeScript / React | frontend-patterns, backend-patterns, coding-standards | code-reviewer |
| Swift / iOS | swiftui-patterns, swift-concurrency-6-2, swift-actor-persistence, swift-protocol-di-testing | code-reviewer |
| PostgreSQL | postgres-patterns, database-migrations | database-reviewer |
| Perl | perl-patterns, perl-testing, perl-security | code-reviewer |
| C++ | cpp-coding-standards, cpp-testing | code-reviewer |
| Other / Unlisted | coding-standards (universal) | code-reviewer |

### Phase 4: Gaps → Assumptions (do NOT ask questions yet)

Check the draft against this list, and mark what Phase 0 already detected:

- [ ] **Tech stack**
- [ ] **Target scope**: files, directories or modules
- [ ] **Acceptance criteria**: how to know the task is done
- [ ] **Error handling**: edge cases and failure modes
- [ ] **Security requirements**: auth, input validation, secrets
- [ ] **Testing expectations**: unit, integration, E2E
- [ ] **Performance constraints**: load, latency, resource limits
- [ ] **UI/UX requirements**: design specs, responsive, a11y (if frontend)
- [ ] **Database changes**: schema, migrations, indexes (if data layer)
- [ ] **Existing patterns**: reference files or conventions to follow
- [ ] **Scope boundaries**: what NOT to do

For every missing item, **pick the most reasonable default** and record it as an assumption with these fields:

- **ID**: A1, A2, …
- **Decision**: what has to be decided
- **Default**: the value you chose
- **Why**: one line
- **Alternatives**: 1–3 options
- **Depends on**: IDs of upstream decisions
- **Weight**: critical (changes the result substantially) or minor

Order the list by dependency, upstream decisions first. Example: auth method → token storage → CSRF handling.

### Phase 5: Workflow & Model

Place the task in the development lifecycle:

```
Research → Plan → Implement (TDD) → Review → Verify → Commit
```

For MEDIUM and above, start with /plan. For EPIC, use the blueprint skill.

| Scope | Recommended Model | Rationale |
|-------|------------------|-----------|
| TRIVIAL–MEDIUM | Current Sonnet-tier model | Fast and cost-efficient for standard coding |
| HIGH | Opus-tier for planning, Sonnet-tier for implementation | Deep reasoning for architecture |
| EPIC | Opus-tier for the blueprint, Sonnet-tier for phase execution | Multi-session planning |

Use the latest model names available to the user; do not hardcode version numbers.

For HIGH/EPIC, split the work into sequential prompts:
- Prompt 1: research and plan (search-first skill, then /plan)
- Prompts 2–N: implement one phase each, ending with /verify
- Final prompt: integration tests and /code-review across all phases
- Use /save-session and /resume-session between sessions

### Stage A Output

**A1. Diagnosis**: strengths of the draft, then an issue table:

| Issue | Impact | Suggested Fix |
|-------|--------|---------------|

**A2. Optimized Prompt v1**: one fenced code block, complete and runnable as is. Tag each assumption inline, e.g. `认证方式：JWT [A1]`.

**A3. Assumptions to Confirm**:

| ID | Decision | Default | Why | Alternatives | Depends on | Weight |
|----|----------|---------|-----|--------------|------------|--------|

Then tell the user what happens next, e.g.: "Next I'll go through these one at a time. Reply 'ok' to accept my recommendation, give your own answer, 'skip' to leave it as an assumption, or 'enough / 够了' to finish now."

**Skip Stage B** and go straight to Stage C when the scope is TRIVIAL or LOW and there are no critical assumptions, or when the user says not to ask questions ("别问了", "直接给"). Unconfirmed assumptions stay labeled in the final prompt.

---

## Stage B: Grill

Interview the user about the assumption list until you share the same understanding of the task.

1. **One question per message.** Never batch questions.
2. **Always recommend.** Each question carries a recommended answer (normally the v1 default) with a one-line reason, plus the alternatives. When AskUserQuestion is available, use it and put the recommended option first, labeled "(Recommended)".
3. **Upstream first.** Follow the depends-on order. Never ask a question whose answer depends on an unresolved one.
4. **Look before asking.** If the codebase, CLAUDE.md or attached files can answer a question, check them, record the answer as *detected*, and skip the question.
5. **Start with the goal.** If success criteria are not explicit, confirm them first. If the draft looks like it is solving the wrong problem, say so and ask before going deeper.
6. **Re-plan after every answer.** Prune branches the answer made irrelevant, add branches it opened, and revise defaults that depended on it.
7. **Critical before minor.** At the end, offer the remaining minor assumptions as one question: "accept all remaining defaults?"
8. **Show progress**, e.g. `Q3 / ~8`.
9. **Stop** when all critical decisions are resolved, when the user says enough, or after 12 questions (then offer to continue).

Question format:

```
Q3 / ~8 · [A3] 登录成功后 token 存哪？
推荐：httpOnly cookie（项目已有 Next.js middleware，且 JS 读不到，可防 XSS 窃取）
其他：localStorage · 内存 + refresh token
回复 ok 采纳推荐 / 直接说你的选择 / skip / 够了
```

---

## Stage C: Finalize

**C1. Decision Log**:

| ID | Decision | Result | Source (detected / confirmed / assumed) |
|----|----------|--------|------------------------------------------|

**C2. Recommended Components**: only available ones.

| Type | Component | Purpose |
|------|-----------|---------|
| Command | /plan | Plan architecture before coding |
| Skill | tdd-workflow | TDD methodology guidance |
| Agent | code-reviewer | Post-implementation review |
| Model | Sonnet-tier | Recommended for this scope |

**C3. Final Prompt (Full Version)**: one fenced code block, self-contained and ready to paste. Include:
- Task description and context, and the tech stack
- **Confirmed decisions**, stated as requirements without hedging
- **Remaining assumptions**, each followed by "if this turns out wrong, stop and ask before continuing"
- Workflow steps, with commands at the right stages
- Acceptance criteria and verification steps
- Scope boundaries (what NOT to do)

For blueprint, write "Use the blueprint skill to…" (not `/blueprint`; it is a skill, not a command).

**C4. Quick Version**: a compact version for experienced users.

| Intent | Quick Pattern |
|--------|--------------|
| New Feature | `/plan [feature]. /tdd to implement. /code-review. /verify.` |
| Bug Fix | `/tdd — write failing test for [bug]. Fix to green. /verify.` |
| Refactor | `/refactor-clean [scope]. /code-review. /verify.` |
| Research | `Use search-first skill for [topic]. /plan based on findings.` |
| Testing | `/tdd [module]. /e2e for critical flows. /test-coverage.` |
| Review | `/code-review. Then use security-reviewer agent.` |
| Docs | `/update-docs. /update-codemaps.` |
| EPIC | `Use blueprint skill for "[objective]". Execute phases with /verify gates.` |

Append the key confirmed decisions to the quick version in one line.

**C5. Changes vs. Draft**:

| Change | Reason / Decision ID |
|--------|----------------------|

**Footer**

> Not what you need? Tell me what to adjust, say "grill more" to keep going on the remaining assumptions, or make a normal task request if you want it executed.

---

## Examples

### Trigger Examples

- "/pre-workout 帮我写一个用户登录页面"
- "Optimize this prompt, then grill me on it"
- "Rewrite this prompt so Claude Code uses the right commands"
- "帮我优化这个指令"

### Example 1: 帮我写一个用户登录页面

**Phase 0 detects:** `package.json` → Next.js 15 + TypeScript + Tailwind CSS. Scope: MEDIUM.

**A2. v1 (excerpt):**
```
使用项目现有技术栈（Next.js 15 + TypeScript + Tailwind CSS）实现用户登录页面。
- 登录方式：邮箱 + 密码 [A1]
- 认证：JWT [A2]，存于 httpOnly cookie [A3]
- 防暴力破解：同 IP 5 次失败后限流 15 分钟 [A4]
- 成功跳转 /dashboard [A5]
- 测试：表单单元测试 + 登录流程集成测试 [A6]
不要做：注册、忘记密码、修改现有路由结构 [A7]
```

**A3. Assumptions (excerpt):**

| ID | Decision | Default | Depends on | Weight |
|----|----------|---------|------------|--------|
| A1 | 登录方式 | 邮箱+密码 | — | critical |
| A2 | 认证机制 | JWT | A1 | critical |
| A3 | token 存储 | httpOnly cookie | A2 | critical |
| A4 | 限流策略 | 5 次 / 15 分钟 | A1 | minor |
| A5 | 成功跳转 | /dashboard | — | minor |

**Stage B:**
```
Q1 / ~5 · [A1] 登录方式用哪种？
推荐：邮箱 + 密码（草稿没提第三方，且依赖最少）
其他：GitHub/Google OAuth · 手机验证码
```
User: "用 Google OAuth"
→ Re-plan: A2 becomes "use NextAuth / Auth.js?" A4 (brute-force throttling) no longer applies and is pruned. A new decision is added: "which Google accounts may sign in?"

**C3. Final prompt (excerpt):**
```
已确认的决策：
- 登录方式：仅 Google OAuth（通过 Auth.js）
- 允许登录：仅 @company.com 域名账号
- 成功跳转 /dashboard
假设（如执行中发现不成立，先停下询问）：
- 会话存储沿用 Auth.js 默认 cookie 方案
工作流：
1. /plan 规划组件结构和认证流程，参考现有页面的模式
2. /tdd 测试先行
3. 实现
4. /code-review
5. /verify
不要做：注册、修改现有路由结构
```

### Example 2: Add a REST API endpoint for user profile updates with validation

**Phase 0 detects:** `go.mod` → Go 1.22 + Chi router. Scope: LOW–MEDIUM.

Typical assumptions: method and path (`PATCH /api/users/:id`) · which fields are editable (name, email, avatar_url, bio) · auth rule (users may update only their own profile) · error codes (400 validation, 401/403 auth, 404 not found) · whether a schema change is allowed (default: no, use the existing user table) · new dependencies (default: none without checking existing ones first, via the search-first skill).

The grill usually starts with the editable fields and the auth rule, because the validation and test cases depend on them. The final prompt ends with: `/tdd` table-driven tests for success, validation failure, auth failure and not-found → implement following existing handler patterns → `/go-review` → `/verify` with the full test suite.

### Example 3: Migrate our monolith to microservices (EPIC)

The v1 prompt is "Use the blueprint skill to plan: 'Migrate monolith to microservices architecture'". The grill covers the foundational decisions in dependency order:

1. Domain boundaries in the current monolith
2. First service to extract (lowest coupling)
3. Communication pattern: REST, gRPC or event-driven (Kafka/RabbitMQ)
4. Database strategy: shared DB at first, or database-per-service from the start
5. Deployment target: Kubernetes, Docker Compose or serverless

The final prompt asks the blueprint to produce phases such as: map service boundaries → set up infrastructure (API gateway, CI/CD per service) → extract the first service with the strangler fig pattern → verify with integration tests, then extract the next → decommission the monolith. Each phase is one PR with a /verify gate, with /save-session and /resume-session between phases, and git worktrees for parallel extraction when dependencies allow. Recommend an Opus-tier model for the blueprint and a Sonnet-tier model for execution.

---

## Related Components

| Component | When to Reference |
|-----------|------------------|
| `configure-ecc` | User hasn't set up ECC yet |
| `skill-stocktake` | Check which components are installed before recommending them |
| `search-first` | Research phase in optimized prompts |
| `blueprint` | EPIC-scope prompts (invoke as skill, not command) |
| `strategic-compact` | Long session context management |
| `cost-aware-llm-pipeline` | Token optimization recommendations |
