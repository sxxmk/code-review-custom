# code-review-custom

A self-contained, two-axis code review skill — no other skill, setup command, or issue-tracker config required.

**[English](#english) · [中文](#中文)**

## English

**What it is.** Reviews the changes since a fixed point (commit SHA, branch, tag, or merge-base) on two independent axes:

- **Standards** — does the code follow this repo's documented conventions? Includes a built-in Fowler smell baseline for repos that document nothing.
- **Spec** — does the code faithfully implement the originating issue / PRD? When none exists, it falls back to the intent stated in the commit messages.

Both axes run as parallel sub-agents and are reported side by side — never merged or re-ranked.

**Why "custom".** The upstream `code-review` skill assumes the Matt Pocock skill suite (a `/setup-...` step and `docs/agents/issue-tracker.md`). This variant inlines issue/PR fetching and spec discovery, so it works standalone in any repo.

**Install.** Copy this folder into a skill directory:

- user level: `~/.agents/skills/code-review-custom/`
- project level: `<repo>/.agents/skills/code-review-custom/`

**Use.**

> Use code-review-custom to review the changes since `abc1234`.
>
> Use code-review-custom to review `main...HEAD`.

**Output.** Two sections — `## Standards` and `## Spec` — followed by a one-line summary per axis (finding count + worst issue).

**Attribution.** Adapted from the `code-review` skill in [mattpocock/skills](https://github.com/mattpocock/skills) (MIT, Copyright (c) 2026 Matt Pocock). This variant removes the cross-skill dependencies and adds commit-message-based spec fallback.

## 中文

**这是什么.** 一个自包含的双轴代码审查 skill，不依赖任何其他 skill、初始化命令或 issue tracker 配置文件。

- **Standards 轴** — 改动是否符合仓库文档化的编码规范；仓库没有规范时，使用内置的 Fowler 代码味道基线（12 条）。
- **Spec 轴** — 改动是否忠实实现了来源 issue / PRD；没有时，退化为用 commit message 中声明的意图做审查，并在报告中注明这是弱证据。
- 两个轴由并行子代理分别执行，结果并排展示，不合并、不重排。

**为什么叫 custom.** 上游 `code-review` 依赖 Matt Pocock 系列 skill 的初始化配置（`/setup-...` 和 `docs/agents/issue-tracker.md`）。本版本把 issue/PR 拉取和 spec 查找内联，因此可在任意仓库独立使用。

**安装.** 把整个目录复制到技能目录：

- 用户级：`~/.agents/skills/code-review-custom/`
- 项目级：`<repo>/.agents/skills/code-review-custom/`

**使用.**

> 用 code-review-custom 审查 `abc1234` 之后的提交。
>
> 用 code-review-custom 审查从 `main` 到 `HEAD` 的改动。

**输出.** 分 `## Standards` 和 `## Spec` 两节，最后每个轴一行汇总（发现数 + 最严重问题）。

**署名.** 改编自 [mattpocock/skills](https://github.com/mattpocock/skills) 的 `code-review` skill（MIT，Copyright (c) 2026 Matt Pocock）。本版本去掉了跨 skill 依赖，并补充了基于 commit message 的 spec 兜底。

## License

MIT — see [LICENSE](LICENSE).
