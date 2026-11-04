<div align="center">

# 🧰 DanteDeRuwe/skills

**Agent skills for agentic engineers**
[dantederuwe.com](https://dantederuwe.com)

</div>

---

## Installation

```bash
npx skills add DanteDeRuwe/skills
```

---

## Overview

| Skill | Description |
|---|---|
| 🔀 [**`/pr`**](pr/SKILL.md) | Opens a PR on GitHub or Azure DevOps from a fresh branch, linked to an issue or work item. |
| 📝 [**`/write-adr`**](write-adr/SKILL.md) | Records an Architecture Decision Record when you ask for one. |
| 🧾 [**`/wrap-up`**](wrap-up/SKILL.md) | Checks what a `/clear` would lose and proposes where to save it. You start it by hand. |

---

## The skills

### 🔀 [`/pr`](pr/SKILL.md)

Turns a finished change into a PR a reviewer can trust. It reads your git remote to pick GitHub or Azure DevOps, branches from the latest default branch, and stages only the files that belong to the change. If your checkout holds unrelated work, it opens a separate worktree and leaves yours alone.

Before it creates anything, it asks which issue or work item to link. The description leads with why, in short bullets: summary, notable decisions, evidence, and merge danger (can you roll it back, and what breaks if you can't). Your repo's PR template takes over when one exists. It won't merge, approve or add reviewers unless you ask.

**Requires:** [`gh`](https://cli.github.com/) for GitHub, or [`az`](https://learn.microsoft.com/cli/azure/) with the `azure-devops` extension for Azure DevOps.

### 📝 [`/write-adr`](write-adr/SKILL.md)

Records a technical decision and the reasoning behind it. It writes to your repo's ADR folder and follows its numbering, or creates `docs/adr/0001-<summary>.md` if you have none.

The template stays lean: status, date and author, then prose. "Considered options" and "Consequences" go in only when they add information, and one paragraph makes a complete ADR. To change a past decision, it writes a new ADR and marks the old one superseded.

It runs only when you ask for an ADR, so design discussions won't trigger it.

### 🧾 [`/wrap-up`](wrap-up/SKILL.md)

Run it before `/clear`. It checks git and the conversation, then lists each decision, open question and touched file with where it lives: saved, unsaved, chat-only or stale.

For each gap it proposes a home (an ADR, an issue, a memory, a commit) and says how sure it is. You reply `all`, pick numbers, or drop items. It saves nothing until you answer, then tells you when the session is safe to clear.

---

## License

[MIT](LICENSE) © Dante De Ruwe
