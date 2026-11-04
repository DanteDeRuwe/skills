---
name: pr
argument-hint: '[issue or work item id(s)]'
description: >-
  Create a pull request on GitHub (gh CLI) or Azure DevOps (az CLI), detected from the git remote:
  branch from the latest default branch, commit only the intended changes, push, and open the PR
  linked to an issue or work item. Use this skill whenever the user asks to make, create, open or
  raise a PR or pull request, says "/pr", "push this and make a PR", "PR this to master/main", or
  wants changes in one or more repos turned into pull requests. Also use it when adding a commit to
  an already open PR or correcting a PR description. Not for reading or answering review comments.
---

# Create a pull request

Turn a finished change into a reviewable PR without surprises: right base, only the intended files, a linked issue or work item, and an honest description.

## 0. Detect the platform and load its reference

Read `git remote get-url origin` (or the remote the branch tracks) and pick the platform from the host:

| Remote host | Platform | Read now |
|---|---|---|
| `github.com`, or a GitHub Enterprise host that `gh auth status` knows | GitHub | `references/github.md` |
| `dev.azure.com`, `*.visualstudio.com`, `ssh.dev.azure.com` | Azure DevOps | `references/azure-devops.md` |

**Read the matching reference file now, before step 1.** It has the login check, how to link issues or work items, and the commands for steps 4, 5 and 7. Do not read the other one.

If the host matches neither, or the repo has remotes on different platforms, ask the user. Never assume an organization, owner or project. If the CLI is not logged in, tell the user the login command from the reference rather than working around it.

## 1. Ask before you create

- **Ask which issue(s) or work item(s) to link, every time.** One PR can link several, and different PRs in the same session often need different ones. If an id is already in the conversation (named by the user, or in the branch name or task), offer it as the default instead of asking blind: "Link #1234, mentioned earlier? Or a different one, more than one, or none?" Never link it without that confirmation. "None" is a valid answer.
- Ask when anything is unclear: target branch, a new PR versus adding to an open one, pre-existing uncommitted changes, whether unrelated edits belong in this PR. The user would rather answer a question than undo a surprise.

## 2. Pick the base and the branch

- Target the repo's default branch. Find it with `git symbolic-ref --short refs/remotes/origin/HEAD` after `git fetch origin`. It differs per repo (`master` or `main`), so never assume.
- Branch from the **latest** `origin/<default>`, not from whatever the checkout is on. A stale local branch drags old commits into the PR.
- If the checkout is dirty or on someone's feature branch, do not stash or switch it. Make a sibling worktree and work there:

  ```bash
  git worktree add -b <branch> ../<repo>_wt origin/<default>
  ```

  When done, remove it from a directory outside it: `git worktree remove --force ../<repo>_wt`. Windows refuses while the shell is inside it, so `cd` out first and delete the folder if git leaves it behind.
- Branch names: follow the repo's convention if it has one (check recent branches or the contributing guide). Otherwise use `chore/...`, `fix/<id>-...` or `feature/...`, short and descriptive.

## 3. Commit only what belongs

- Run `git status --short` before committing. A build can modify tracked generated files (API docs, lock files, snapshots); revert those unless they belong to the change. Files that were already modified before you started are not yours: leave them and mention them.
- Check the diff size. A one-line change that shows as a whole-file diff means line endings were rewritten; redo the edit with a tool that preserves them.
- Stage explicit paths, not `git add -A` in a repo with pre-existing changes. Commit with a message that says what and why, following the repo's commit convention if it has one (for example Conventional Commits). Add the attribution lines the environment asks for.
- Push with `git push -u origin <branch>`. Never force-push a branch someone may have reviewed.

## 4. Open the PR

Write the description to a file first, check it is not empty (`wc -c "$DESC_FILE"`), then create the PR with the command from the platform reference. An empty file or a path the shell cannot read turns into an empty description with no error. Keep temp files in a location the same program that writes them also reads (Python on Windows does not see Git Bash's `/tmp`).

Link the confirmed issues or work items the way the platform reference says. If it names a PR template in the repo, fill in its sections instead of the headings below.

**The description**

The reviewer can read the diff for the *what*. The description exists for the *why*.

- **Lead with the why.** Every change gets its reason: the problem, the risk, the requirement. A bullet with only a what is incomplete.
- **Bullets, not paragraphs.** One idea per bullet. Fragments are fine when they are clear: "Pin X to 2.1: 2.2 breaks Y" beats a full sentence.
- **Clear over complete.** Cut preambles, hedges and restating the title. If a word adds nothing, drop it.
- **ASD-STE100 (Simplified Technical English) as a guide:** active voice, plain common words, one word for one thing, no idioms.

Use these headings as guidance, not a fixed template: drop a section a small PR has nothing to say in, rather than filling it with filler.

- **Summary**: why the change is needed, then what changed, in a few bullets. Add the smallest visual that makes the point when it clarifies, next to the text it supports. Plain text is the safe default: a shallow file tree, a call or component tree, pseudocode, or a diff-style sketch. A fenced `mermaid` block (sequence diagram, flow) is also allowed when a picture beats text; if it does not render in the PR, replace it with a plain-text tree or diff. Show a whole code block only when most of it is new or the reader needs a copyable target shape. Skip visuals for dependency bumps and other changes that read fine as text.
- **Notable decisions**: anything a reviewer might question (a temporary pin, a skipped option, a forced bump), with the reason.
- **Evidence**: before and after. For a dependency bump, the resolved versions before and after. For a fix, the test that failed and now passes, in pseudocode. For a visual change, screenshots. Say plainly what you did not run (tests that need Docker, environments you cannot reach) and name failures that also occur on the untouched base branch. Never imply more was checked than was.
- **Merge danger** (omit for trivial changes such as docs, or write "none"): the *door*, one-way or two-way, meaning whether it can be rolled back; and the *blast radius*, one word plus any ramifications, such as consumers that could break or services that must be redeployed together.
- End with any attribution line the environment asks for.

Do not add reviewers, set auto-merge or auto-complete, approve, merge, close or abandon the PR unless the user asks. Open it as a draft only when asked.

## 5. Change an existing PR

The commands are in the platform reference.

- **Open PR, more work**: commit to the same branch and push; the PR updates itself. Update the title or description if the scope grew. Check the description file first, because an empty value blanks the description.
- **Merged, closed or abandoned PR**: it will not take new commits. Branch again from the latest default branch and open a new PR.
- **Wrong description**: fix it right away; reviewers read it.
- **Add or change a linked issue or work item**: as the platform reference says.

## 6. Several repos at once

Open one PR per repo with a consistent title. Ask for the issue or work item once, but confirm it applies to all; repos on different platforms or projects usually need different ids. If a repo is on another platform, read that platform's reference before handling it. If the first repo reveals a problem (a failing test, a wrong base), fix the approach before repeating it everywhere. Finish with a table: repo, platform, PR id and link, target branch, linked issue or work item, and what was or was not verified.

## 7. Report back

Give the PR id and URL (the platform reference says where to get it), the linked issue(s) or work item(s), a one-line summary of the change, and what was not verified. Mention anything you left alone on purpose, such as pre-existing uncommitted files.
