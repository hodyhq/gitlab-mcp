# hodyhq fork of `@zereight/mcp-gitlab`

Fork of [`zereight/gitlab-mcp`](https://github.com/zereight/gitlab-mcp) @ v2.1.21, MIT.
We run our own copy so the GitLab MCP can do **everything** without roadblocks.

## What this fork adds

Upstream's `push_files` hardcoded `action: "create"` (couldn't update existing files)
and the file tools couldn't carry **binary** content. This fork makes file commits
first-class:

- **`push_files`** — each file may set:
  - `action`: `create` | `update` | `delete` | `move` (default `create`)
  - `encoding`: `text` | `base64` (use `base64` for binaries; content passed through
    untouched, binary-safe)
  - `previous_path`: source path for `action: "move"`
  - `content` is now optional (omit for `delete`)
- **`create_or_update_file`** — adds optional `encoding: "base64"` for binary files.

Everything else is unchanged and stays in sync with upstream behaviour (the global
`GITLAB_REPO_FILE_ENCODING` toggle still works as the default).

## Why

Building [ipme.wtf](https://github.com/hodyhq/ipme-wtf) we hit two walls: `push_files`
errored on existing files ("file already exists") and neither tool could commit the
logo PNGs. Verified fix: an end-to-end test committed a real PNG via `push_files`
(`encoding: "base64"`) and the bytes round-tripped exactly.

## Changes (vs upstream v2.1.21)

- `schemas.ts` — `FileOperationSchema`, `PushFilesSchema.files`, `CreateOrUpdateFileSchema`: per-file `action` / `encoding` / `previous_path`, optional `content`.
- `index.ts` — `createCommit` honours per-file action + binary-safe base64; `createOrUpdateFile` accepts `encoding`; both tool handlers thread the new fields.

## Run

```bash
npm install && npm run build
node build/index.js     # GITLAB_API_URL + GITLAB_PERSONAL_ACCESS_TOKEN in env
```
