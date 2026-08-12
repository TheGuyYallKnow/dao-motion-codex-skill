# Dao Motion Codex Skill

A Codex skill for safely authoring, inspecting, repairing, validating, staging, applying, and compacting Dao Motion Lab graphs in Roblox Studio.

The skill uses the official Roblox Studio MCP and the plugin-owned Dao Motion gateway. It intentionally never starts Motion Preview, enters Play mode, or writes directly to Dao Motion persistence.

## Install

Clone this repository into your Codex skills directory:

```powershell
git clone https://github.com/TheGuyYallKnow/dao-motion-codex-skill.git "$env:USERPROFILE\.codex\skills\dao-motion"
```

Restart Codex, then invoke the skill with `$dao-motion`.

## Contents

- `SKILL.md` — operating rules and core workflow.
- `references/gateway-contract.md` — Studio gateway protocol.
- `references/authoring-workflows.md` — advanced authoring workflows.
- `agents/openai.yaml` — skill display metadata and default prompt.

## Safety boundary

Visual appearance and runtime behavior remain user-tested. The skill verifies structure through graph validation, diffing, staging, read-back, Apply state, and runtime status only.
