# glab API Usage Reference

The `glab api` command makes authenticated HTTP requests to any GitLab API endpoint — both REST and GraphQL.

## Synopsis

```bash
glab api <endpoint> [flags]
```

---

## HTTP Methods

| Method | When Used |
|--------|-----------|
| **GET** | Default when no body parameters are provided |
| **POST** | Default when `--field`, `--raw-field`, or `--form` parameters are added |
| **Custom** | Override with `--method` / `-X` (PUT, PATCH, DELETE, etc.) |

```bash
# Explicit method
glab api projects/:fullpath/issues --method POST --field title="Bug report"
glab api projects/:fullpath/issues/42 --method PUT --field state_event="close"
glab api projects/:fullpath/issues/42 --method DELETE
```

---

## Placeholder Variables

When inside a Git directory, these placeholders are auto-replaced from the repo context:

| Placeholder | Resolves To |
|-------------|-------------|
| `:id` | Project ID |
| `:fullpath` | URL-encoded full project path (e.g., `group%2Fproject`) |
| `:namespace` | Project namespace/group |
| `:repo` | Repository name |
| `:group` | Group name |
| `:branch` | Current Git branch |
| `:user` | Authenticated user |
| `:username` | Authenticated username |

```bash
# These are equivalent when inside the project repo:
glab api projects/:fullpath/issues
glab api projects/my-group%2Fmy-project/issues
```

---

## Sending Data

### `--field` / `-F` (Auto-Typed)

Infers JSON types from the value format:
- `true`, `false`, `null` → JSON boolean/null
- Integer values → JSON numbers
- Placeholder values (`:namespace`, `:repo`, `:branch`) → auto-resolved strings
- `@filename` → reads value from file
- `@-` → reads from stdin

```bash
glab api projects/:fullpath/issues --method POST \
  -F title="New issue" \
  -F confidential=true \
  -F weight=5
```

### `--raw-field` / `-f` (Always String)

All values are treated as JSON strings, no type conversion.

```bash
glab api projects/:fullpath/issues --method POST \
  -f title="New issue" \
  -f description="This is always a string"
```

### `--form` (Multipart Form Data)

Required for file upload endpoints. Cannot be combined with `--field`, `--raw-field`, or `--input`.

```bash
# Upload a file
glab api projects/:fullpath/wikis/attachments --method POST \
  --form "file=@./image.png" \
  --form "branch=main"

# Read from stdin
echo "content" | glab api projects/:fullpath/wikis/attachments --method POST \
  --form "file=@-" --form "branch=main"
```

### `--input` (Raw Body)

Send a raw request body from a file. When used, `--field` flags become URL query parameters.

```bash
glab api projects/:fullpath/issues --method POST \
  --input request-body.json
```

---

## Pagination

### REST Pagination

```bash
# Fetch all pages automatically
glab api projects/:fullpath/issues --paginate

# NDJSON output (one object per line, memory-efficient)
glab api projects/:fullpath/issues --paginate --output ndjson

# Pipe to jq for filtering
glab api projects/:fullpath/issues --paginate --output ndjson | \
  jq 'select(.state == "opened")'
```

### GraphQL Pagination

For GraphQL, the query must:
1. Accept `$endCursor: String` as a variable
2. Fetch `pageInfo { hasNextPage, endCursor }` from the collection

```bash
glab api graphql --paginate -f query='
  query($endCursor: String) {
    project(fullPath: "gitlab-org/graphql-sandbox") {
      issues(first: 10, after: $endCursor) {
        edges {
          node { title state }
        }
        pageInfo {
          endCursor
          hasNextPage
        }
      }
    }
  }
'
```

---

## Output Formats

| Flag | Format | Description |
|------|--------|-------------|
| `--output json` | JSON (default) | Pretty-printed. Arrays combined into single JSON array |
| `--output ndjson` | NDJSON/JSONL | One JSON object per line. Better for large datasets |

```bash
# Default JSON
glab api projects/:fullpath/issues

# NDJSON for large datasets
glab api projects/:fullpath/issues --paginate --output ndjson
```

---

## Additional Options

| Flag | Description |
|------|-------------|
| `-H, --header` | Add custom HTTP request header |
| `-i, --include` | Include HTTP response headers in output |
| `--hostname` | Override the GitLab hostname |
| `--silent` | Do not print the response body |

```bash
# Custom header
glab api projects/:fullpath -H "Accept: application/json"

# Include response headers (useful for debugging pagination)
glab api projects/:fullpath/issues --include

# Target a different GitLab instance
glab api projects/123 --hostname gitlab.internal.company.com

# Silent mode (check status code only)
glab api projects/:fullpath/issues --silent
```

---

## GraphQL Queries

All fields other than `query` and `operationName` are treated as GraphQL variables.

```bash
# Simple query
glab api graphql -f query="query { currentUser { username } }"

# Query with variables
glab api graphql \
  -f query='query($path: ID!) { project(fullPath: $path) { name } }' \
  -f path="gitlab-org/gitlab"

# Complex query
glab api graphql -f query='
  query {
    project(fullPath: "gitlab-org/gitlab-docs") {
      name
      forksCount
      statistics { wikiSize }
      issuesEnabled
      boards {
        nodes { id name }
      }
    }
  }
'
```

---

## Common Patterns

### List project members
```bash
glab api projects/:fullpath/members/all --paginate
```

### Get project details
```bash
glab api projects/:fullpath
```

### Create a project variable
```bash
glab api projects/:fullpath/variables --method POST \
  -f key="MY_VAR" -f value="my-value" -f protected=true
```

### Trigger a pipeline
```bash
glab api projects/:fullpath/pipeline --method POST \
  -f ref="main"
```

### Download a single file from repo
```bash
glab api projects/:fullpath/repository/files/README.md/raw --raw-field ref=main
```

### List merge request approvals
```bash
glab api projects/:fullpath/merge_requests/15/approvals
```

---

## Gotchas

- **URL encoding**: Project paths with `/` must be URL-encoded (e.g., `group%2Fsubgroup%2Fproject`). Use `:fullpath` placeholder to avoid this.
- **GraphQL pagination**: Both `$endCursor` variable and `pageInfo` fields are required — omitting either will silently stop at the first page.
- **`--form` exclusivity**: Cannot be combined with `--field`, `--raw-field`, or `--input`.
- **Default method changes**: Adding `--field` or `--raw-field` changes the default method from GET to POST. Use `--method GET` explicitly if you want query parameters with GET.
- **Large output**: `--output ndjson` with `--paginate` is more memory-efficient than the default JSON for large result sets.
