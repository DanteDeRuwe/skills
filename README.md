<div align="center">

# 🧰 DanteDeRuwe/skills

**Agent skills for agentic engineers**
[dantederuwe.com](https://dantederuwe.com)

</div>

---

## Overview

| Skill | What it does | Invoke | Notes |
|---|---|---|---|
| 🔀 [**`pr`**](pr/) | Branches from the latest default branch, commits only the intended changes, pushes, and opens a PR linked to an issue or work item. | `/pr [id…]` or ask for a PR | Supports **GitHub** (`gh`) and **Azure DevOps** (`az`), picked from the git remote. Always asks which issue to link. Never merges, approves or adds reviewers on its own. |
| 📝 [**`write-adr`**](write-adr/) | Records an Architecture Decision Record: the decision, **why** it was made, and its consequences. | `/write-adr [summary]` | Only runs when you ask for an ADR. Uses the repo's existing ADR folder and numbering, otherwise `docs/adr/NNNN-*.md`. |
| 🧾 [**`wrap-up`**](wrap-up/) | Audits what the session would lose on `/clear` and suggests where to save each item. | `/wrap-up` | Manual only (`disable-model-invocation`). Writes nothing until you pick which saves to apply. |

---

## The skills

### 🔀 [`pr`](pr/SKILL.md): create a pull request

Turns a finished change into a reviewable PR: the right base, only the intended files, a linked issue, and an honest description.

- **Platform detection.** Reads the remote and loads only the matching reference: [`github.md`](pr/references/github.md) or [`azure-devops.md`](pr/references/azure-devops.md).
- **Fresh base.** Fetches and branches from the latest `origin/<default>`. A dirty checkout gets a sibling **worktree** rather than a stash.
- **Clean commits.** Stages explicit paths, reverts build noise, and catches line-ending rewrites.
- **Descriptions that explain why.** Short bullets covering *Summary*, *Notable decisions*, *Evidence* and *Merge danger* (one-way or two-way door, blast radius). Fills in the repo's PR template when there is one.
- **Multi-repo.** One PR per repo, then a summary table of links, targets and what was verified.

### 📝 [`write-adr`](write-adr/SKILL.md): record a decision

A lean ADR template: Status, Date and Author, then prose. *Considered options* and *Consequences* are optional. One paragraph is a complete ADR.

```text
docs/adr/0007-use-outbox-pattern-for-event-publishing.md
```

Past ADRs are never rewritten. A change of mind gets a new ADR, and the old one is marked `Superseded by ADR NNNN`.

### 🧾 [`wrap-up`](wrap-up/SKILL.md): audit the session before `/clear`

Builds a **ledger** of every decision, open question and touched file, and checks git rather than relying on memory.

| Mark | Meaning |
|:-:|---|
| ✅ | Saved: durable and pushed or posted |
| ❌ | Written but not durable: uncommitted, unpushed or local-only |
| ⚠️ | Chat only |
| 🔁 | Stale: saved, but something newer contradicts it |

Each gap gets a suggested home (ADR, glossary, issue, memory, commit) with a confidence level. You reply `all`, `1 3` or `drop 2`, and it ends with 🟢 **Safe to /clear**.

---

## Installation

Each top-level folder is a self-contained skill (`SKILL.md` plus optional `references/`).

```bash
npx skills add DanteDeRuwe/skills
```

### Manually

Copy or symlink the folders you want into your skills directory:

```bash
git clone https://github.com/DanteDeRuwe/skills.git

# Personal (all projects)
cp -r skills/pr skills/write-adr skills/wrap-up ~/.claude/skills/

# Or per project
cp -r skills/pr .claude/skills/
```

> [!TIP]
> Symlink instead of copying (`ln -s "$PWD/skills/pr" ~/.claude/skills/pr`) so a `git pull` updates them.

### Requirements

| Skill | Needs |
|---|---|
| `pr` | `git`, plus [`gh`](https://cli.github.com/) for GitHub or [`az`](https://learn.microsoft.com/cli/azure/) with the `azure-devops` extension for Azure DevOps |
| `write-adr` | `git` (used for the author name if the session doesn't provide it) |
| `wrap-up` | `git` |

---

## Repository layout

```text
.
├── pr/
│   ├── SKILL.md
│   └── references/
│       ├── azure-devops.md
│       └── github.md
├── write-adr/
│   └── SKILL.md
└── wrap-up/
    └── SKILL.md
```

## License

[MIT](LICENSE) © Dante De Ruwe
