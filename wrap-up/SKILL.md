---
name: wrap-up
description: Audit what this session would lose on /clear and propose where to save it.
disable-model-invocation: true
---

Audit the session as a **ledger**: every decision, open question and touched file gets a row saying where it lives now. Run it before `/clear`.

## 1. Gather

Verify, never recall:

- `git status -sb`, `git log @{u}..` and `git branch -vv`: uncommitted, unpushed, untracked and ignored files.
- Every file the session wrote or referenced, including scratch and gitignored ones.
- Every outward action (issue comments, PRs, messages) and what each now contradicts.
- The conversation: decisions with their reasons, rejected options, the user's judgment calls, open questions, next steps, preferences the user stated about how to work.

Changes that predate the session get one line under the table and stay out of the ledger.

## 2. Classify

| Mark | Meaning |
|---|---|
| ✅ | Saved: durable and pushed or posted |
| ❌ | Written but not durable: uncommitted, unpushed, or local-only |
| ⚠️ | Chat only |
| 🔁 | Stale: saved somewhere, but another doc or issue now contradicts it |

## 3. Place

Best guess per ⚠️, ❌ and 🔁 row. Read the repo's own conventions first (`AGENTS.md`, `CLAUDE.md`, its docs layout); they beat the defaults below.

| Item | Default destination |
|---|---|
| Decision with reasons or a rejected option | ADR (the repo's decision log) |
| New or changed term | Glossary |
| Value that is configurable | Settings catalog, asking the user for its level |
| Follow-up work | Issue |
| Stale doc, issue or comment | Edit, or a correction comment pointing at the source |
| How the user likes to work | Memory, `feedback` |
| Fact about the user or the project that the repo can't show | Memory, `user` or `project` |
| Local-only file worth keeping | Commit, or move out of the ignored folder |
| Trivia or derivable from the repo | Drop |

Confidence is **high** when one destination fits and **low** when two do; name the runner-up for low.

## 4. Report

Print exactly this shape, short cells, no prose around it:

```
## Verdict
🟢 Safe to /clear          or          🟡 N items would be lost

## Ledger
| | Item | Lives in |
|---|---|---|
| ✅ | one-line item | `path` or commit |
| ⚠️ | one-line item | chat only |

## Suggested saves
| # | Item | Destination | Sure? |
|---|---|---|---|
| 1 | one-line item | ADR 0009, Considered Options | high |

Reply `all`, numbers to save (`1 3`), or `drop 2`.
```

Omit "Suggested saves" on 🟢.

## 5. Save

Write nothing before the user replies; commits, pushes and posts are outward-facing. Then apply the chosen saves, re-run the gathering commands, and print the Ledger again.

Done when every row is ✅ or explicitly dropped. Only then say **🟢 Safe to /clear**.
