# INSTALL.md — OpenCode instructions for installing OCSuperpowers

You are OpenCode. Follow these instructions to install OCSuperpowers from the `akstlab/oc-superpowers` repository.

Do not treat this file as a human shell-command checklist. Use it as your operating procedure: ask the required questions, clone the source repo to a temporary directory, copy the correct files to the selected OpenCode location, optionally route into model customization, verify the result, and delete the temporary clone.

## Required first questions

Ask these before changing files:

```text
Should I install OCSuperpowers into this local project only, or globally for all OpenCode projects on this machine?
```

If the user chooses local and you are not already inside the target repository, ask for the target project path.

Then ask:

```text
After installation, do you want to customize the model used by OCSuperpower and each subagent?
```

If the user says yes, complete the install first, then follow:

```text
https://raw.githubusercontent.com/akstlab/oc-superpowers/main/docs/CUSTOMIZE-MODELS.md
```

Pass the selected install scope and installed agent directory into that flow so the user is not asked the same scope question again.

## Resolve target paths

For a local install, the target agent directory is:

```text
<TARGET_PROJECT>/.opencode/agents
```

The OCSuperpowers reference directory is:

```text
<TARGET_PROJECT>/.opencode/ocsuperpower
```

For a global install, inspect the user's OpenCode configuration location before assuming paths. Prefer OpenCode's own config discovery/help if available:

```bash
opencode --help || true
opencode config --help || true
```

If no OpenCode command reveals the config location, use the conventional global OpenCode config directory:

```text
~/.config/opencode
```

Then use:

```text
<OPEN_CODE_CONFIG>/agents
<OPEN_CODE_CONFIG>/ocsuperpower
```

## Clone source to a temporary directory

Do not assume the current working directory already contains the source package.

Create a temporary directory, clone the repository into it, and copy from that clone:

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
test -d "$SRC/.opencode/ocsuperpower/source-skills"
```

If any required source path is missing, stop and report the missing path.

## Local project install procedure

Use this when the user wants OCSuperpowers available only inside one repository.

```bash
TARGET="/absolute/path/to/project"
mkdir -p "$TARGET/.opencode/agents"
mkdir -p "$TARGET/.opencode/ocsuperpower"
cp "$SRC/.opencode/agents/OCSuperpower.md" "$TARGET/.opencode/agents/"
cp "$SRC"/.opencode/agents/ocsp-*.md "$TARGET/.opencode/agents/"
rm -rf "$TARGET/.opencode/ocsuperpower/source-skills"
cp -R "$SRC/.opencode/ocsuperpower/source-skills" "$TARGET/.opencode/ocsuperpower/"
AGENT_DIR="$TARGET/.opencode/agents"
REF_DIR="$TARGET/.opencode/ocsuperpower"
```

## Global install procedure

Use this when the user wants OCSuperpowers available in all OpenCode projects.

```bash
OC_CONFIG="$HOME/.config/opencode"
mkdir -p "$OC_CONFIG/agents"
mkdir -p "$OC_CONFIG/ocsuperpower"
cp "$SRC/.opencode/agents/OCSuperpower.md" "$OC_CONFIG/agents/"
cp "$SRC"/.opencode/agents/ocsp-*.md "$OC_CONFIG/agents/"
rm -rf "$OC_CONFIG/ocsuperpower/source-skills"
cp -R "$SRC/.opencode/ocsuperpower/source-skills" "$OC_CONFIG/ocsuperpower/"
AGENT_DIR="$OC_CONFIG/agents"
REF_DIR="$OC_CONFIG/ocsuperpower"
```

If OpenCode reveals a different global config path, use that path instead of `~/.config/opencode`.

## Delete the temporary clone

After copying and before final response, delete the temporary clone:

```bash
rm -rf "$TMPDIR"
```

If cleanup fails, report the temp path so the user can remove it manually.

## Agent validity checks

After either install type, check every installed agent file:

1. It starts with YAML frontmatter (`---`).
2. `OCSuperpower.md` has `mode: primary`.
3. Every `ocsp-*.md` has `mode: subagent`.
4. `dispatching-parallel-agents` was not created as a separate agent.
5. The preserved source-skill references exist under the OCSuperpowers reference directory.

Suggested validation command:

```bash
python3 - <<'PY'
from pathlib import Path
import os
agent_dir = Path(os.environ.get('AGENT_DIR', '.opencode/agents')).expanduser()
ref_dir = Path(os.environ.get('REF_DIR', '.opencode/ocsuperpower')).expanduser()
if not agent_dir.exists():
    raise SystemExit(f'missing agent dir: {agent_dir}')
errors = []
files = sorted(agent_dir.glob('OCSuperpower.md')) + sorted(agent_dir.glob('ocsp-*.md'))
for path in files:
    text = path.read_text()
    if not text.startswith('---\n'):
        errors.append(f'{path}: missing YAML frontmatter')
        continue
    end = text.find('\n---\n', 4)
    if end == -1:
        errors.append(f'{path}: unterminated YAML frontmatter')
        continue
    fm = text[4:end]
    if path.name == 'OCSuperpower.md' and 'mode: primary' not in fm:
        errors.append(f'{path}: OCSuperpower must be mode: primary')
    if path.name.startswith('ocsp-') and 'mode: subagent' not in fm:
        errors.append(f'{path}: ocsp agent must be mode: subagent')
    if 'dispatching-parallel-agents' in path.name:
        errors.append(f'{path}: dispatching-parallel-agents should be folded into OCSuperpower, not installed separately')
if not (ref_dir / 'source-skills').exists():
    errors.append(f'{ref_dir / "source-skills"}: missing preserved source skills')
if errors:
    raise SystemExit('\n'.join(errors))
print(f'validated {len(files)} agent files in {agent_dir}')
PY
```

Set `AGENT_DIR` and `REF_DIR` to the installed paths before running the command.

## Optional model customization

If the user asked to customize models, follow the instructions from:

```text
https://raw.githubusercontent.com/akstlab/oc-superpowers/main/docs/CUSTOMIZE-MODELS.md
```

Do this only after install validation passes. Reuse the install scope and `AGENT_DIR` already resolved in this install flow.

## Final response to the user

After installing, report:

- whether the install was local or global;
- the install path;
- the number of installed agent files;
- whether `OCSuperpower.md` is primary;
- whether all `ocsp-*` files are subagents;
- whether model customization was skipped or completed;
- any warnings about config-path assumptions;
- whether the temporary clone was deleted.

Then suggest:

```text
Start OpenCode in your project and choose OCSuperpower as the primary agent.
```
