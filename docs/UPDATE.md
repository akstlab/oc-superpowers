# UPDATE.md — OpenCode instructions for updating OCSuperpowers

You are OpenCode. Follow these instructions to update an existing OCSuperpowers install from the `akstlab/oc-superpowers` repository.

Do not treat this as a human shell-command checklist. Ask the required questions, locate the installed files, preserve customizations, clone the latest source to a temporary directory, perform the update, optionally route into model customization, verify the result, and delete the temporary clone.

## Required first questions

Ask these before changing files:

```text
Am I updating OCSuperpowers in this local project, or updating the global OpenCode install?
```

If local and you are not already in the target project, ask for the target project path.

Then ask:

```text
After updating, do you want to customize or change the model used by OCSuperpower and each subagent?
```

If the user says yes, complete the update first, then follow:

```text
https://raw.githubusercontent.com/akstlab/oc-superpowers/main/docs/CUSTOMIZE-MODELS.md
```

Pass the selected update scope and installed agent directory into that flow so the user is not asked the same scope question again.

## Resolve installed paths

For local updates:

```text
<TARGET_PROJECT>/.opencode/agents
```

For global updates, inspect the installed OpenCode configuration path. Prefer OpenCode's own config discovery/help if available:

```bash
opencode --help || true
opencode config --help || true
```

If no OpenCode command reveals it, use the conventional global OpenCode config directory:

```text
~/.config/opencode
```

Then use:

```text
<OPEN_CODE_CONFIG>/agents
```

If the selected install path does not exist, stop and ask whether to perform a fresh install using:

```text
https://raw.githubusercontent.com/akstlab/oc-superpowers/main/docs/INSTALL.md
```

## Preserve existing model choices

Before overwriting any agent file, inspect the existing installed agents for `model:` frontmatter.

Preserve those model assignments unless the user explicitly says to replace them with package defaults or new selections.

Collect a mapping like:

```text
OCSuperpower.md -> provider/model-id
ocsp-writing-plans.md -> provider/model-id
...
```

Use this mapping after copying updated files.

## Back up before updating

Always create a timestamped backup of the existing install.

For local installs:

```bash
mkdir -p "$TARGET/.opencode-backups"
BACKUP="$TARGET/.opencode-backups/ocsuperpowers-$(date +%Y%m%d-%H%M%S)"
cp -R "$TARGET/.opencode" "$BACKUP"
```

For global installs:

```bash
mkdir -p "$(dirname "$OC_CONFIG")/opencode-backups"
BACKUP="$(dirname "$OC_CONFIG")/opencode-backups/ocsuperpowers-$(date +%Y%m%d-%H%M%S)"
cp -R "$OC_CONFIG" "$BACKUP"
```

## Clone source to a temporary directory

Do not assume the current working directory already contains the source package.

```bash
TMPDIR="$(mktemp -d)"
git clone --depth 1 https://github.com/akstlab/oc-superpowers.git "$TMPDIR/oc-superpowers"
SRC="$TMPDIR/oc-superpowers"
```

If `git` is unavailable, ask the user to provide a local path to an extracted copy of `akstlab/oc-superpowers` and set `SRC` to that path.

Before copying, verify the required source files exist:

```bash
test -f "$SRC/.opencode/agents/OCSuperpower.md"
find "$SRC/.opencode/agents" -maxdepth 1 -type f -name 'ocsp-*.md' | sort
```

If any required source path is missing, stop and report the missing path.

## Update files

For local installs:

```bash
mkdir -p "$TARGET/.opencode/agents"

cp "$SRC/.opencode/agents/OCSuperpower.md" "$TARGET/.opencode/agents/"
cp "$SRC"/.opencode/agents/ocsp-*.md "$TARGET/.opencode/agents/"

AGENT_DIR="$TARGET/.opencode/agents"
```

For global installs:

```bash
mkdir -p "$OC_CONFIG/agents"

cp "$SRC/.opencode/agents/OCSuperpower.md" "$OC_CONFIG/agents/"
cp "$SRC"/.opencode/agents/ocsp-*.md "$OC_CONFIG/agents/"

AGENT_DIR="$OC_CONFIG/agents"
```

## Re-apply preserved model frontmatter

After copying, re-apply the saved `model:` mapping to each matching agent file.

Only edit YAML frontmatter. Do not rewrite agent bodies unless the user explicitly asks.

If an old model is no longer supported by the user's configured OpenCode providers, report that and ask whether to keep it as text, remove it, or replace it using:

```text
https://raw.githubusercontent.com/akstlab/oc-superpowers/main/docs/CUSTOMIZE-MODELS.md
```

## Delete the temporary clone

After copying, preserving model values, and validating, delete the temporary clone:

```bash
rm -rf "$TMPDIR"
```

If cleanup fails, report the temp path so the user can remove it manually.

## Update validity checks

Verify:

1. `OCSuperpower.md` exists and has `mode: primary`.
2. Every `ocsp-*.md` exists and has `mode: subagent`.
3. `dispatching-parallel-agents` does not exist as a standalone agent.
4. Existing `model:` assignments were preserved or intentionally replaced.

Suggested checks:

```bash
find "$AGENT_DIR" -maxdepth 1 -type f \( -name 'OCSuperpower.md' -o -name 'ocsp-*.md' \) | sort
grep -R "^model:" "$AGENT_DIR" 2>/dev/null || true
```

## Optional model customization

If the user asked to customize models, follow the instructions from:

```text
https://raw.githubusercontent.com/akstlab/oc-superpowers/main/docs/CUSTOMIZE-MODELS.md
```

Do this only after update validation passes. Reuse the update scope and `AGENT_DIR` already resolved in this update flow.

## Final response to the user

After updating, report:

- whether the update was local or global;
- the updated path;
- the backup path;
- the number of installed agent files;
- which `model:` values were preserved, changed, or skipped;
- whether model customization was skipped or completed;
- whether validation passed;
- whether the temporary clone was deleted;
- any warnings or manual follow-ups.
