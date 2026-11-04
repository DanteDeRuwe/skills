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

## 0. Detect the platform

Read `git remote get-url origin` (or the remote the branch tracks) and pick the platform from the host:

| Remote host | Platform | CLI |
|---|---|---|
| `github.com`, or a GitHub Enterprise host that `gh auth status` knows | GitHub | `gh` |
| `dev.azure.com`, `*.visualstudio.com`, `ssh.dev.azure.com` | Azure DevOps | `az` with the `azure-devops` extension |

If the host matches neither, or the repo has several remotes on different platforms, ask the user. Never assume an organization, owner or project.

Check that the CLI is installed and logged in (`gh auth status`, or `az account show`). If it is not, tell the user the login command to run (`gh auth login`, or `az login`) rather than working around it.

Platform notes:

- **GitHub**: `gh` reads the owner and repo from the remote. For a fork, PRs go to the upstream repo by default; confirm the target repo when `gh repo view --json isFork,parent` says it is a fork.
- **Azure DevOps**: `az` detects the organization and project from the remote. If it cannot, pass `--organization` and `--project`, read from the remote: `https://dev.azure.com/<org>/<project>/_git/<repo>` (the project can be URL-encoded), or `https://<org>.visualstudio.com/<project>/_git/<repo>`.

## 1. Ask before you create

- **Ask which issue(s) or work item(s) to link, every time.** One PR can link several, and different PRs in the same session often need different ones. If an id is already in the conversation (named by the user, or in the branch name or task), offer it as the default instead of asking blind: "Link #1234, mentioned earlier? Or a different one, more than one, or none?" Never link it without that confirmation. "None" is a valid answer.
- Ask when anything is unclear: target branch, a new PR versus adding to an open one, pre-existing uncommitted changes, whether unrelated edits belong in this PR. The user would rather answer a question than undo a surprise.

## 2. Pick the base and the branch

- Target the repo's default branch. Find it with `git symbolic-ref --short refs/remotes/origin/HEAD` after `git fetch origin` (on GitHub, `gh repo view --json defaultBranchRef -q .defaultBranchRef.name` also works). It differs per repo (`master` or `main`), so never assume.
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

Write the description to a file first, check it is not empty, then pass it in. An empty file or a path the shell cannot read turns into an empty description with no error. Keep temp files in a location the same program that writes them also reads (Python on Windows does not see Git Bash's `/tmp`).

**GitHub**

```bash
wc -c "$DESC_FILE"
gh pr create --base <default> --head <branch> --title "<title>" --body-file "$DESC_FILE"
```

GitHub links issues through the description: add a line such as `Closes #123`, or `Refs #123` when merging should not close it. For an issue in another repo, use `owner/repo#123`. If the repo has a PR template (`.github/pull_request_template.md` or `.github/PULL_REQUEST_TEMPLATE/`), fill in its sections instead of the headings below.

**Azure DevOps**

```bash
wc -c "$DESC_FILE"
az repos pr create --source-branch <branch> --target-branch <default> \
  --title "<title>" --description "$(cat "$DESC_FILE")" \
  --work-items <id> [<id> ...] --query "{id:pullRequestId}" -o tsv
```

Azure DevOps links work items through `--work-items`, not through the description.

**The description**

A good description is short and factual. Skip preambles and keep the prose brief. Where it fits, write in the spirit of ASD-STE100 (Simplified Technical English): short sentences, one idea per sentence, active voice, plain common words, and the same word for the same thing. Use these headings as guidance, not a fixed template: drop a section a small PR has nothing to say in, rather than filling it with filler.

- **Summary**: what changed and why, in a few lines. Add the smallest visual that makes the point when it clarifies, next to the text it supports. Plain text is the safe default: a shallow file tree, a call or component tree, pseudocode, or a diff-style sketch. A fenced `mermaid` block (sequence diagram, flow) is also allowed when a picture beats text; GitHub and Azure DevOps both render it in PR descriptions, and if it does not render, replace it with a plain-text tree or diff. Show a whole code block only when most of it is new or the reader needs a copyable target shape. Skip visuals for dependency bumps and other changes that read fine as text.
- **Notable decisions**: anything a reviewer might question (a temporary pin, a skipped option, a forced bump), with the reason.
- **Evidence**: before and after. For a dependency bump, the resolved versions before and after. For a fix, the test that failed and now passes, in pseudocode. For a visual change, screenshots. Say plainly what you did not run (tests that need Docker, environments you cannot reach) and name failures that also occur on the untouched base branch. Never imply more was checked than was.
- **Merge danger** (omit for trivial changes such as docs, or write "none"): the *door*, one-way or two-way, meaning whether it can be rolled back; and the *blast radius*, one word plus any ramifications, such as consumers that could break or services that must be redeployed together.
- End with any attribution line the environment asks for.

Do not add reviewers, set auto-merge or auto-complete, approve, merge, close or abandon the PR unless the user asks. Open it as a draft (`gh pr create --draft`, or `az repos pr create --draft true`) only when asked.

## 5. Change an existing PR

- **Open PR, more work**: commit to the same branch and push; the PR updates itself. Update the title or description if the scope grew. Check the file first, because an empty value blanks the description.
  - GitHub: `gh pr edit <id> --title "..." --body-file "$DESC_FILE"`
  - Azure DevOps: `az repos pr update --id <id> --title "..." --description "$(cat "$DESC_FILE")"`
- **Merged, closed or abandoned PR**: it will not take new commits. Branch again from the latest default branch and open a new PR.
- **Wrong description**: fix it right away with the edit command above; reviewers read it.
- **Add or change a linked issue or work item**:
  - GitHub: edit the description to add or change the `Closes #<id>` line.
  - Azure DevOps: `az repos pr work-item add --id <id> --work-items <wi>`.

## 6. Several repos at once

Open one PR per repo with a consistent title. Ask for the issue or work item once, but confirm it applies to all; repos on different platforms or projects usually need different ids. If the first repo reveals a problem (a failing test, a wrong base), fix the approach before repeating it everywhere. Finish with a table: repo, platform, PR id and link, target branch, linked issue or work item, and what was or was not verified.

## 7. Report back

Give the PR id and URL (both CLIs return it; for Azure DevOps it is the repo's remote URL without credentials, plus `/pullrequest/<id>`), the linked issue(s) or work item(s), a one-line summary of the change, and what was not verified. Mention anything you left alone on purpose, such as pre-existing uncommitted files.
