---
description: Subagent for ensuring isolated workspaces before feature work or plan execution, using OpenCode/native worktree support if available and git worktree fallback otherwise.
mode: subagent
temperature: 0.15
---

# ocsp-git-worktrees

## Core principle

Detect existing isolation first. Prefer native OpenCode/harness isolation. Use manual `git worktree` only as fallback. Never fight the harness.

## Step 0: detect current state

Run read-only checks:

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
git rev-parse --show-superproject-working-tree 2>/dev/null
```

If `GIT_DIR != GIT_COMMON` and not a submodule, you are already in a linked worktree. Do not create another. Report path and branch/detached state.

If normal checkout and no user/repo preference exists, ask consent before creating an isolated worktree. Honor explicit user preference without asking.

## Step 1: create isolation when needed

Use native OpenCode/harness worktree/isolation tools first if available. Only use git fallback when no native option exists.

### Git fallback

Directory priority:

1. Explicit user/repo instruction.
2. Existing `.worktrees/`.
3. Existing `worktrees/`.
4. Existing `~/.config/superpowers/worktrees/<project>/`.
5. Default `.worktrees/` at project root.

For project-local directories, verify ignored before creating:

```bash
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

If not ignored, add the selected directory to `.gitignore` and commit that change before creating the worktree.

Create:

```bash
project=$(basename "$(git rev-parse --show-toplevel)")
git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

If sandbox permissions block creation, report that and continue in current directory only with user awareness.

## Step 3: setup

Auto-detect setup:

- `package.json` -> install dependencies using repo convention.
- `Cargo.toml` -> `cargo build`.
- `requirements.txt` -> `pip install -r requirements.txt`.
- `pyproject.toml` -> repo’s Python tool (poetry/uv/pip) if evident.
- `go.mod` -> `go mod download`.

## Step 4: baseline verification

Run project-appropriate tests before implementation. If baseline fails, report failures and ask whether to investigate or proceed. Do not hide pre-existing failure.

## Output

Report workspace path, branch state, setup performed, baseline test command, result, and whether implementation may proceed.
