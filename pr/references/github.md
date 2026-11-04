# GitHub (`gh`)

Commands and quirks for GitHub. The process lives in `SKILL.md`; this file only says how to do each step on GitHub.

## Setup

- Login check: `gh auth status`. If not logged in, tell the user to run `gh auth login`.
- `gh` reads the owner and repo from the remote. GitHub Enterprise hosts work when `gh auth status` lists them.
- **Forks**: `gh repo view --json isFork,parent`. If it is a fork, PRs go to the upstream repo by default. Confirm the target repo with the user; pass `--repo <owner>/<repo>` to choose it.
- Default branch: `gh repo view --json defaultBranchRef -q .defaultBranchRef.name` (or the `git symbolic-ref` command in `SKILL.md`).

## Linking issues

- Issues are linked through the description, not a flag.
- `Closes #123` closes the issue on merge. `Refs #123` links without closing.
- Issue in another repo: `owner/repo#123`.

## PR template

If the repo has `.github/pull_request_template.md` or a `.github/PULL_REQUEST_TEMPLATE/` folder, fill in its sections instead of the headings in `SKILL.md`. The style rules in `SKILL.md` still apply.

## Create

```bash
wc -c "$DESC_FILE"
gh pr create --base <default> --head <branch> --title "<title>" --body-file "$DESC_FILE"
```

- Draft, only when asked: add `--draft`.
- `gh pr create` prints the PR URL.

## Change an existing PR

- Title or description: `gh pr edit <id> --title "..." --body-file "$DESC_FILE"`
- Add or change a linked issue: edit the description and change the `Closes #<id>` line.
- Check state before adding work: `gh pr view <id> --json state -q .state` (`OPEN`, `MERGED`, `CLOSED`).

## Report

- URL: printed by `gh pr create`, or `gh pr view <id> --json url -q .url`.
