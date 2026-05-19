Always respond in Chinese-simplified

# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

## 跨会话上下文记录

每个项目的上下文文件 `context.md` 就在该项目的根目录下。

- **对话开始时：** 必须先读取项目根目录下的 `context.md`，获取之前的对话内容和工作状态
- **如果不存在：** 自动创建 `context.md`，记录本次对话内容
- **对话结束前：** 必须更新项目根目录下的 `context.md`，以"用户说了什么 → 我做了什么"的格式逐条记录对话过程，体现来回交流，而非只写结果摘要
- **路径规则：** 永远是 `{项目根目录}/context.md`，不要放到其他地方

## Git 提交规范

**每次修改代码后必须自动提交。**

提交信息格式：
```
claude：/类型：描述
```

类型包括：
- `/修改` - 代码变更、重构
- `/修复` - Bug 修复
- `/增加` - 新增功能
- `/删除` - 移除代码/功能
- `/优化` - 性能优化
- `/样式` - UI/CSS 调整

示例：
```
claude：/修复：登录按钮点击无响应问题
claude：/增加：用户头像上传功能
claude：/样式：调整卡片悬浮动画效果
```
