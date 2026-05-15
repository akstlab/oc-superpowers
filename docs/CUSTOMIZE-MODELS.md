# CUSTOMIZE-MODELS.md — OpenCode instructions for customizing OCSuperpowers models

You are OpenCode. Follow these instructions to help the user customize per-agent models for OCSuperpowers.

Do not start by editing files. First determine whether this flow was called from install/update with a known scope and agent directory. If the scope or agent directory is not already known, ask the user whether to customize the local project install or the global OpenCode install.

Then discover which OpenCode providers and models are already configured on this machine. Show those configured options first. Use `https://models.dev/api.json` only as a live supplemental catalog for model IDs, not as a replacement for the user's actual configured providers.

## What to customize

OpenCode agent prompts are markdown files with YAML frontmatter. A per-agent model is configured by adding or updating a `model:` field:

```yaml
---
description: Use this agent for ...
mode: subagent
model: provider/model-id
temperature: 0.2
---
```

Only edit the YAML frontmatter unless the user explicitly asks to change prompts.

## Required first questions

If this flow was launched from `INSTALL.md` or `UPDATE.md` and the install scope plus `AGENT_DIR` are already known, do not ask the scope question again.

Otherwise, ask:

```text
Should I customize models for the local project install or the global OpenCode install?
```

If local and you are not already inside the target repository, ask for the target project path.

After discovering configured providers/models, ask:

```text
Here are the OpenCode providers/models I found configured on this machine. Which provider should OCSuperpowers use?
```

After provider choice, ask:

```text
Which model ID should I assign to each OCSuperpowers agent group?
```

## Resolve target agent directory

Local install:

```text
<TARGET_PROJECT>/.opencode/agents
```

Global install:

Prefer OpenCode's own config discovery/help. If no OpenCode command reveals the config location, use the conventional path:

```text
~/.config/opencode/agents
```

If the selected agent directory does not contain `OCSuperpower.md`, stop and ask whether to install first using:

```text
https://raw.githubusercontent.com/akstlab/oc-superpowers/main/docs/INSTALL.md
```

## Discover OpenCode config and configured providers

Research the local OpenCode installation instead of assuming a fixed provider list.

Use this order:

1. Ask the installed OpenCode CLI for supported commands. Prefer any command that lists config, auth, providers, or models:

```bash
opencode --help || true
opencode config --help || true
opencode auth --help || true
opencode models --help || true
opencode provider --help || true
```

2. If a command exists that displays configured providers or models, run it and use that output as the primary source.

3. Inspect likely project config files in the target project without exposing secrets:

```bash
find "$TARGET" -maxdepth 2 -type f \( -name 'opencode.json' -o -name 'opencode.jsonc' -o -name 'opencode.*.json' -o -name 'opencode.*.jsonc' \) -print 2>/dev/null
find "$TARGET/.opencode" -maxdepth 3 -type f -print 2>/dev/null
```

4. Inspect likely global config/state locations. OpenCode commonly uses an XDG-style user config directory; many installs use `~/.config/opencode`. Some provider/auth/model state can also appear under XDG data or state directories depending on version/package manager. Check these locations, but do not print secrets:

```bash
find ~/.config/opencode -maxdepth 4 -type f -print 2>/dev/null
find ~/.local/share/opencode -maxdepth 4 -type f -print 2>/dev/null
find ~/.local/state/opencode -maxdepth 4 -type f -print 2>/dev/null
```

5. Search the discovered files for non-secret provider/model references. Redact values that look like API keys, tokens, passwords, or secrets:

```bash
grep -RInE '"provider"|"model"|provider:|model:' "$TARGET"/.opencode ~/.config/opencode ~/.local/share/opencode ~/.local/state/opencode 2>/dev/null \
  | sed -E 's/(api[_-]?key|token|secret|password)([^[:space:]]*)[[:space:]]*[:=][[:space:]]*[^, ]+/\1\2: [REDACTED]/Ig' || true
```

6. Show the user a concise summary of configured providers/models found. Do not expose raw config files or secrets.

## Use models.dev as a live model catalog

After showing configured providers/models, use the live catalog only as a supplement:

```text
https://models.dev/api.json
```

If network access is available, fetch and parse it. Match the user's configured provider names against the live provider keys. Then list model IDs for the selected provider.

If network access is unavailable, do not invent a static model list. Use only the models discovered from the user's OpenCode config/CLI, or ask the user to paste the exact `provider/model-id` they want.

## Recommended model groups

Use stronger reasoning models for orchestration, planning, debugging, review, and skill-writing. Use faster/cheaper models for mechanical execution only when the repo has reliable tests and the task risk is low.

| Group | Agent files | Recommended profile |
|---|---|---|
| Primary orchestration | `OCSuperpower.md` | strongest coding/reasoning model; good long context; reliable tool use |
| Planning and architecture | `ocsp-brainstorming.md`, `ocsp-writing-plans.md` | strong reasoning model; good long context |
| Debugging, review, verification | `ocsp-systematic-debugging.md`, `ocsp-requesting-code-review.md`, `ocsp-receiving-code-review.md`, `ocsp-verification-before-completion.md` | strong reasoning model; conservative; evidence-focused |
| Implementation and TDD | `ocsp-subagent-driven-development.md`, `ocsp-executing-plans.md`, `ocsp-test-driven-development.md` | strong coding model; can be cost-optimized if tests are reliable |
| Git workflow | `ocsp-git-worktrees.md`, `ocsp-finishing-development-branch.md` | fast, reliable coding model usually sufficient |
| Skill authoring | `ocsp-writing-skills.md` | strong instruction-writing and reasoning model |

## Edit procedure

1. Resolve `AGENT_DIR`.
2. Back up the current agents before editing.
3. Discover and show configured OpenCode providers/models.
4. Ask the user to choose provider first.
5. Ask the user for model choices by group.
6. Edit only `model:` in YAML frontmatter.
7. Preserve `description`, `mode`, `temperature`, and all body text.
8. Validate the final mapping.

## Safe frontmatter editing script

Use this script only after the user has chosen exact model IDs.

```bash
python3 - <<'PY'
from pathlib import Path
import os

agent_dir = Path(os.environ.get('AGENT_DIR', '.opencode/agents')).expanduser()
assignments = {
    # Fill these after the user chooses exact provider/model IDs.
    # 'OCSuperpower.md': 'provider/model-id',
}

for name, model in assignments.items():
    path = agent_dir / name
    if not path.exists():
        raise SystemExit(f'missing agent: {path}')
    text = path.read_text()
    if not text.startswith('---\n'):
        raise SystemExit(f'{path} has no YAML frontmatter')
    end = text.find('\n---\n', 4)
    if end == -1:
        raise SystemExit(f'{path} has unterminated YAML frontmatter')
    fm = text[4:end].splitlines()
    body = text[end + len('\n---\n'):]
    out = []
    wrote = False
    for line in fm:
        if line.startswith('model:'):
            out.append(f'model: {model}')
            wrote = True
        else:
            out.append(line)
    if not wrote:
        insert_at = 0
        for i, line in enumerate(out):
            if line.startswith('mode:'):
                insert_at = i + 1
                break
        out.insert(insert_at, f'model: {model}')
    path.write_text('---\n' + '\n'.join(out) + '\n---\n' + body)

print(f'updated {len(assignments)} agent model assignments in {agent_dir}')
PY
```

Set `AGENT_DIR` to the real agent directory before running.

## Validation

After editing, verify:

```bash
test -f "$AGENT_DIR/OCSuperpower.md"
find "$AGENT_DIR" -maxdepth 1 -type f \( -name 'OCSuperpower.md' -o -name 'ocsp-*.md' \) -print | sort
grep -R "^model:" "$AGENT_DIR" 2>/dev/null || true
```

Then inspect the YAML frontmatter of each edited file and confirm `mode:` did not change.

## Final response to the user

After customization, report:

- whether the edited install was local or global;
- which config/agent directory was edited;
- which configured providers/models were discovered;
- whether `models.dev` was reachable;
- the final model assigned to each agent group;
- any models that could not be verified against OpenCode's configured providers;
- validation status.
