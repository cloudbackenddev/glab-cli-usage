---
name: glab-cli-usage
description: >
  Use the glab command-line tool to interact with GitLab from the terminal.
  Use when asked to create, list, view, close, reopen, or comment on GitLab
  issues and merge requests. Also covers CI/CD pipeline management, repository
  operations, release creation, label and milestone management, direct GitLab
  API calls, custom aliases, snippets, and todos. Use this skill when the user
  mentions glab, GitLab CLI, merge requests, GitLab issues, GitLab pipelines,
  or GitLab API from the command line. Works with GitLab.com, GitLab
  Self-Managed, and GitLab Dedicated instances.
metadata:
  revision: 1
  updated-on: "2026-04-24"
  source: community
  tags: "gitlab,glab,cli,merge-request,ci-cd,pipeline,devops,git"
---

# Skill: Use glab to Manage GitLab from the Terminal

## What This Skill Does

When asked to interact with GitLab from the command line, use the `glab` CLI tool (gitlab.com/gitlab-org/cli). This skill covers issues, merge requests, CI/CD pipelines, releases, direct API access, aliases, and more.

**Prerequisite**: Confirm `glab` is installed and authenticated:
```bash
glab --version
glab auth status
```
If not configured, see `references/setup-configuration.md`.

---

## Task 1: Issues

Use `glab issue` subcommands to manage GitLab issues.

```bash
# List open issues in the current project
glab issue list

# Filter by label, assignee, milestone
glab issue list --label bug --assignee @me --milestone "v1.0"

# List closed issues
glab issue list --closed

# List all issues (open + closed)
glab issue list --all

# Search by title
glab issue list --search "login error"

# Create an issue (interactive)
glab issue create

# Create non-interactively
glab issue create --title "Fix auth bug" --description "Steps to reproduce..." \
  --label bug --label urgent --assignee @me --milestone "v1.0"

# View an issue
glab issue view 42

# View in browser
glab issue view 42 --web

# Close / reopen
glab issue close 42
glab issue reopen 42

# Add a comment
glab issue note 42 --message "Working on this now"

# Subscribe / unsubscribe
glab issue subscribe 42
glab issue unsubscribe 42

# Delete an issue
glab issue delete 42
```

---

## Task 2: Merge Requests

Use `glab mr` for merge request workflows.

```bash
# List open merge requests
glab mr list

# Filter by label, assignee, reviewer
glab mr list --label frontend --assignee @me --reviewer "jdoe"

# Create an MR from the current branch (interactive)
glab mr create

# Create non-interactively
glab mr create --title "Add search feature" --description "Implements #42" \
  --label feature --assignee @me --target-branch main --remove-source-branch

# Create as draft
glab mr create --draft --title "WIP: Refactor auth"

# View an MR
glab mr view 15
glab mr view 15 --web

# Checkout an MR locally
glab mr checkout 15

# Approve / revoke approval
glab mr approve 15
glab mr revoke 15

# Merge an MR
glab mr merge 15
glab mr merge 15 --squash --remove-source-branch --when-pipeline-succeeds

# Close / reopen
glab mr close 15
glab mr reopen 15

# Add a comment / note
glab mr note 15 --message "LGTM, approved"

# Diff
glab mr diff 15

# Mark as ready (remove draft status)
glab mr update 15 --ready

# Update MR metadata
glab mr update 15 --title "New title" --add-label priority::high --milestone "v2.0"
```

---

## Task 3: CI/CD Pipelines & Jobs

Use `glab ci` and `glab job` for pipeline management.

```bash
# View current pipeline status
glab ci status

# List recent pipelines
glab ci list

# View pipeline in browser
glab ci view --web

# Trace/follow a running pipeline's logs
glab ci trace

# Retry a failed pipeline
glab ci retry

# Cancel a running pipeline
glab ci cancel

# Trigger a pipeline run
glab ci run

# Trigger on a specific branch with variables
glab ci run --branch develop --variables "DEPLOY_ENV=staging"

# Delete a pipeline
glab ci delete <pipeline-id>

# View a specific job's trace/logs
glab ci trace <job-id>

# List jobs in a pipeline
glab job list --pipeline <pipeline-id>

# Download job artifacts
glab ci artifact <job-id>

# Lint CI config
glab ci lint
glab ci lint .gitlab-ci.yml
```

---

## Task 4: Repositories

```bash
# Clone a repository
glab repo clone owner/repo

# Clone into a specific directory
glab repo clone owner/repo ./my-dir

# Fork a repository
glab repo fork owner/repo

# View repo in browser
glab repo view --web

# Archive a project
glab repo archive owner/repo

# Search for repositories
glab repo search --search "my-project"
```

---

## Task 5: Releases

```bash
# List releases
glab release list

# Create a release
glab release create v1.0.0

# Create with notes and assets
glab release create v1.0.0 --name "Version 1.0" \
  --notes "Release notes here" ./build/app.zip

# Create from a tag with auto-generated notes
glab release create v1.0.0 --notes ""

# View a release
glab release view v1.0.0

# Delete a release
glab release delete v1.0.0

# Upload assets to existing release
glab release upload v1.0.0 ./dist/artifact.tar.gz
```

---

## Task 6: Direct API Calls

Use `glab api` for authenticated requests to any GitLab API endpoint. See `references/api-usage.md` for full details.

```bash
# GET request (default method)
glab api projects/:fullpath/releases

# POST with fields (--field for auto-typed, --raw-field for strings)
glab api projects/:fullpath/issues --method POST \
  --field title="New issue" --raw-field description="Details here"

# Using URL-encoded project path
glab api projects/gitlab-com%2Fwww-gitlab-com/issues

# Upload a file via multipart form
glab api projects/:fullpath/wikis/attachments --method POST \
  --form "file=@./image.png" --form "branch=main"

# Paginate through all results
glab api issues --paginate

# Paginate with NDJSON output (memory-efficient for large datasets)
glab api issues --paginate --output ndjson

# Filter paginated results with jq
glab api issues --paginate --output ndjson | jq 'select(.state == "opened")'

# GraphQL query
glab api graphql -f query="query { currentUser { username } }"

# Complex GraphQL query
glab api graphql -f query='
  query {
    project(fullPath: "gitlab-org/gitlab-docs") {
      name
      forksCount
      statistics { wikiSize }
      boards { nodes { id name } }
    }
  }
'

# Paginated GraphQL
glab api graphql --paginate -f query='
  query($endCursor: String) {
    project(fullPath: "gitlab-org/graphql-sandbox") {
      issues(first: 10, after: $endCursor) {
        edges { node { title } }
        pageInfo { endCursor hasNextPage }
      }
    }
  }
'

# Include response headers
glab api projects/:fullpath --include

# Override hostname for different GitLab instance
glab api projects/:fullpath --hostname gitlab.example.com
```

### Placeholder Variables

These are auto-replaced from the current Git repo context:
`:id`, `:fullpath`, `:namespace`, `:repo`, `:group`, `:branch`, `:user`, `:username`

---

## Task 7: Aliases, Labels, Milestones & Other Commands

```bash
# --- Labels ---
glab label list
glab label create "priority::high" --color "#FF0000" --description "High priority"

# --- Milestones ---
glab milestone list

# --- Snippets ---
glab snippet create --title "My snippet" --filename snippet.py
glab snippet list

# --- Todos ---
glab todo list
glab todo done <todo-id>

# --- Config ---
glab config set editor vim
glab config get editor

# --- Completions ---
glab completion bash > /etc/bash_completion.d/glab
glab completion zsh > "${fpath[1]}/_glab"
glab completion fish > ~/.config/fish/completions/glab.fish
glab completion powershell > glab.ps1

# --- Misc ---
glab changelog generate
glab check-update
glab version
```

---

## Task 8: Searching (Repositories & Code)

### Search for Repositories
Use `glab repo search` to find projects across your GitLab instance by name.
Reference: [glab repo search](https://docs.gitlab.com/cli/repo/search/)

```bash
# Search for repositories matching a string
glab repo search --search "my-project"

# Get JSON output for parsing
glab repo search --search "my-project" --output json
```

### Search Code Inside a Repository
To search the actual content (blobs) of a repository remotely without cloning, use the GitLab Search API:

```bash
# Search codebase remotely (replace :id with project path or ID)
glab api "projects/:id/search?scope=blobs&search=YOUR_QUERY" --output json
```

---

## Workflow Patterns

These sequences help achieve larger multi-step goals:

### Summarize Feature/Epic Status
1. Find related issues: `glab issue list --label=<feature-label> --output json`
2. Find related MRs: `glab mr list --label=<feature-label> --output json`
3. For each open MR, check pipeline: `glab ci status --branch <source-branch>`
4. Optionally review code: `glab mr diff <id>` or checkout and read files

### Review an MR Thoroughly
1. Read discussion: `glab mr view <id> --comments`
2. See changes: `glab mr diff <id>`
3. Check CI: `glab ci status`
4. Add feedback: `glab mr note <id> --message "..."`
5. (Optional) Get code locally: `glab mr checkout <id>`

### Debug a Failed Pipeline
1. Find failed pipeline: `glab ci list --output json`
2. Identify failed job: `glab ci view`
3. Read full logs: `glab ci trace <job-id>`
4. Suggest fix or create issue

### Prepare Release Summary
1. Find previous tag: `glab release list`
2. Compare commits: `glab api projects/:id/repository/compare?from=<old>&to=<new>`
3. Get merged MRs: `glab mr list --state=merged --updated-after=<date> --output json`
4. Get closed issues: `glab issue list --state=closed --updated-after=<date> --output json`

### Triage My Work
1. Assigned issues: `glab issue list --assignee=@me --state=opened --output json`
2. Assigned MRs: `glab mr list --assignee=@me --state=opened --output json`
3. MRs to review: `glab mr list --reviewer=@me --state=opened --output json`

---

## Gotchas

- **JSON Parsing**: Always use `--output json` when you need to parse output programmatically in scripts or automated workflows.
- **`@me`** is the shortcut for the current authenticated user — use in `--assignee` and filter flags.
- **`--web` / `-w`** flag opens the resource in the default browser from most view commands.
- **Auto-detection**: `glab` reads Git remotes from the current directory to determine the project and GitLab host. Run commands from inside a cloned repo, or use `--repo owner/repo` to override.
- **Multiple instances**: Use `glab auth login --hostname gitlab.example.com` to add additional hosts. Use `--hostname` on individual commands to target a specific host.
- **NO_PROMPT**: Set `NO_PROMPT=1` to skip interactive prompts in scripts and automation.
- **API placeholders**: `:fullpath`, `:repo`, `:namespace`, `:branch` etc. are replaced from the current repo context only when inside a Git directory.
- **`--output ndjson`**: Use with `--paginate` on `glab api` to get one JSON object per line, which is more memory-efficient for large datasets and works well with `jq`.
- **CI pipeline context**: `glab ci` commands operate on the current branch's pipeline by default. Use `--branch` to target a different branch.

---

## Reference Files

Load on demand for deeper context:

- `references/setup-configuration.md` — Install, auth, multi-host config, environment variables
- `references/api-usage.md` — Full `glab api` reference, placeholders, GraphQL, pagination, forms
- `references/commands-detailed.md` — Comprehensive command reference covering all flags and specific use-cases
- `references/troubleshooting.md` — Common errors, authentication issues, and connection problems

---

## Official Sources

- https://gitlab.com/gitlab-org/cli
- https://docs.gitlab.com/cli/
- https://docs.gitlab.com/api/
