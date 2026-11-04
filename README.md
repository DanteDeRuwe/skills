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
| 🔀 [**`pr`**](pr/) | Opens a PR on GitHub or Azure DevOps from a fresh branch, linked to an issue or work item. |
| 📝 [**`write-adr`**](write-adr/) | Records an Architecture Decision Record. Runs only when you ask. |
| 🧾 [**`wrap-up`**](wrap-up/) | Checks what a `/clear` would lose and suggests where to save it. Manual only. |

---

## The skills

### 🔀 [`pr`](pr/SKILL.md)

Branches from the latest default branch, commits only the intended files, and writes a description that explains *why*. It always asks which issue to link and never merges or approves.

**Requires:** [`gh`](https://cli.github.com/) for GitHub, or [`az`](https://learn.microsoft.com/cli/azure/) with the `azure-devops` extension for Azure DevOps.

### 📝 [`write-adr`](write-adr/SKILL.md)

Writes a lean ADR (status, date, author, then the reasoning) in the repo's ADR folder, or `docs/adr/` if there isn't one. Past ADRs are superseded, never rewritten.

### 🧾 [`wrap-up`](wrap-up/SKILL.md)

Lists every decision, open question and touched file, and marks each as saved, unsaved, chat-only or stale. Saves only the items you pick.

---

## License

[MIT](LICENSE) © Dante De Ruwe
