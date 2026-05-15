# OCSuperpowers for OpenCode

OCSuperpowers is an OpenCode agent pack converted from the uploaded Superpowers skill set.
Set of subagents based on skills from 
https://github.com/obra/superpowers


## Install OCSuperpowers

Paste this into OpenCode:

```text
Read and follow the install instructions from:
Install OCSuperpowers for OpenCode.
https://raw.githubusercontent.com/akstlab/oc-superpowers/main/docs/INSTALL.md
```

## Update OCSuperpowers

Paste this into OpenCode:

```text
Update OCSuperpowers for OpenCode.
Read and follow the update instructions from:
https://raw.githubusercontent.com/akstlab/oc-superpowers/main/docs/UPDATE.md
```

## Customize OCSuperpowers models

Paste this into OpenCode:

```text
Customize OCSuperpowers models for OpenCode.
Read and follow the model customization instructions from:
https://raw.githubusercontent.com/akstlab/oc-superpowers/main/docs/CUSTOMIZE-MODELS.md
```

## What is included

- `.opencode/agents/OCSuperpower.md` — primary agent based on `using-superpowers`.
- `.opencode/agents/ocsp-*.md` — optimized subagents converted from the remaining uploaded skills.
- `.opencode/ocsuperpower/source-skills/` — preserved original uploaded skills for audit/reference.
- `docs/INSTALL.md` — OpenCode-run install procedure.
- `docs/UPDATE.md` — OpenCode-run update procedure.
- `docs/CUSTOMIZE-MODELS.md` — OpenCode-run per-agent model customization procedure.

## Agent structure

`OCSuperpower` is the only primary agent. It owns routing, delegation, parallel-subagent strategy, verification gates, and final completion discipline.


Subagents:

- `ocsp-brainstorming`
- `ocsp-writing-plans`
- `ocsp-git-worktrees`
- `ocsp-subagent-driven-development`
- `ocsp-executing-plans`
- `ocsp-test-driven-development`
- `ocsp-systematic-debugging`
- `ocsp-verification-before-completion`
- `ocsp-requesting-code-review`
- `ocsp-receiving-code-review`
- `ocsp-finishing-development-branch`
- `ocsp-writing-skills`

## Recommended first use

Start OpenCode in a project and select `OCSuperpower`.

```text
Use OCSuperpower for this task. First route to the right subagent workflow, then proceed with verification before completion.
```
