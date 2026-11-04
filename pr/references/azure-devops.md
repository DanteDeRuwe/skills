# Azure DevOps (`az`)

Commands and quirks for Azure DevOps. The process lives in `SKILL.md`; this file only says how to do each step on Azure DevOps.

## Setup

- Needs the `azure-devops` extension: `az extension show --name azure-devops`. If missing: `az extension add --name azure-devops`.
- Login check: `az account show`. If not logged in, tell the user to run `az login`.
- `az` detects the organization and project from the remote. If it cannot, pass `--organization` and `--project`, read from the remote:
  - `https://dev.azure.com/<org>/<project>/_git/<repo>` (the project can be URL-encoded)
  - `https://<org>.visualstudio.com/<project>/_git/<repo>` (org is in the host)
  - `git@ssh.dev.azure.com:v3/<org>/<project>/<repo>`
  - `<org>@vs-ssh.visualstudio.com:v3/<org>/<project>/<repo>`

## Linking work items

- Work items are linked with `--work-items`, not through the description.
- Several ids: `--work-items 123 456`. With no work item, leave the flag out.

## PR template

The CLI does not apply the repo's PR template. If the repo has a `pull_request_template.md` in `.azuredevops/`, `.vsts/`, `docs/` or the root (or `pull_request_template/branches/<target>.md` for the target branch), fill in its sections instead of the headings in `SKILL.md`. The style rules in `SKILL.md` still apply.

## Create

```bash
test -s "$DESC_FILE" && \
  az repos pr create --source-branch <branch> --target-branch <default> \
  --title "<title>" --description "@$DESC_FILE" \
  --work-items <id> [<id> ...] --query "{id:pullRequestId,url:repository.webUrl}" -o tsv
```

- `@<path>` makes `az` read the value from the file. This avoids shell quoting, newline and command-length problems (`az.cmd` from PowerShell cuts a multi-line argument). `az` runs on Python, so on Windows use a Windows or relative path, not Git Bash's `/tmp`.
- Draft, only when asked: add `--draft true`.

## Change an existing PR

- Title or description: `az repos pr update --id <id> --title "..." --description "@$DESC_FILE"`. An empty value blanks the description.
- Add a linked work item: `az repos pr work-item add --id <id> --work-items <wi>`.
- Check state before adding work: `az repos pr show --id <id> --query status -o tsv` (`active`, `completed`, `abandoned`).

## Report

- URL: `<webUrl>/pullrequest/<id>`, with `webUrl` from the create output or `az repos pr show --id <id> --query repository.webUrl -o tsv`. Do not build it from an SSH remote, which is not a web address.
