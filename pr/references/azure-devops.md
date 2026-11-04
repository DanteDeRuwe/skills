# Azure DevOps (`az`)

Commands and quirks for Azure DevOps. The process lives in `SKILL.md`; this file only says how to do each step on Azure DevOps.

## Setup

- Needs the `azure-devops` extension: `az extension show --name azure-devops`. If missing: `az extension add --name azure-devops`.
- Login check: `az account show`. If not logged in, tell the user to run `az login`.
- `az` detects the organization and project from the remote. If it cannot, pass `--organization` and `--project`, read from the remote:
  - `https://dev.azure.com/<org>/<project>/_git/<repo>` (the project can be URL-encoded)
  - `https://<org>.visualstudio.com/<project>/_git/<repo>` (org is in the host)
  - `git@ssh.dev.azure.com:v3/<org>/<project>/<repo>`

## Linking work items

- Work items are linked with `--work-items`, not through the description.
- Several ids: `--work-items 123 456`.

## Create

```bash
wc -c "$DESC_FILE"
az repos pr create --source-branch <branch> --target-branch <default> \
  --title "<title>" --description "$(cat "$DESC_FILE")" \
  --work-items <id> [<id> ...] --query "{id:pullRequestId}" -o tsv
```

- No `--body-file` equivalent: pass the file content with `"$(cat "$DESC_FILE")"`. An unreadable file gives an empty description with no error, so the `wc -c` check matters.
- Draft, only when asked: add `--draft true`.

## Change an existing PR

- Title or description: `az repos pr update --id <id> --title "..." --description "$(cat "$DESC_FILE")"`. An empty value blanks the description.
- Add a linked work item: `az repos pr work-item add --id <id> --work-items <wi>`.
- Check state before adding work: `az repos pr show --id <id> --query status -o tsv` (`active`, `completed`, `abandoned`).

## Report

- URL: the repo's remote URL without credentials, plus `/pullrequest/<id>`.
