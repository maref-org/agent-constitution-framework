# Pattern 4: External Agent Governance

## What It Solves

Modern development environments include AI coding assistants: Claude Code, Cursor, GitHub Copilot, Opencode, Trae AI, and others. Each has its own configuration file, its own behavior patterns, and its own access to tools. When multiple such agents operate in the same repository, they need consistent rules — but each agent reads a different config file.

External Agent Governance provides a **unified rule set** that works across heterogeneous AI coding tools, with **preflight checks** that run before any agent performs potentially destructive operations.

## Key Concepts

### 1. Scope Definition

Identify which agents operate in your repository and where their config files live:

| Agent Type | Configuration File | Typical Behavior |
|-----------|-------------------|------------------|
| Claude Code | `CLAUDE.md` | LLM CLI, file + exec access |
| Cursor | `.cursorrules` | IDE AI, file access |
| GitHub Copilot | `.github/copilot-instructions.md` | IDE autocomplete |
| Opencode | `opencode.jsonc` | AI CLI, file + exec access |
| Trae AI | `.trae/rules/` | IDE Agent, file access |

All agents listed in the scope must adhere to the same foundational rules. The constitutional governance pattern (Pattern 1) ensures consistency across different config file formats.

### 2. Preflight Checks

Before an external agent performs any operation in the repository, it should execute preflight checks:

1. **Remote verification**: Confirm `git remote -v` shows only authorized remotes. Block if unauthorized remotes exist.
2. **Hook verification**: Confirm security hooks (`pre-push`, `pre-commit`) exist and are executable.
3. **Policy acknowledgment**: Read and confirm understanding of the repository's push restrictions.
4. **Push denial**: Refuse to execute any `git push` or remote modification commands.
5. **Remote config denial**: Refuse to execute any `git remote add` or remote configuration changes.

**Workspace switching**: If the agent switches between directories (e.g., from Repository A to Repository B):
- In Repository A → execute full preflight for Repository A's rules
- In Repository B → execute full preflight for Repository B's rules
- When spawning sub-agents → include preflight requirements in the task instructions

### 3. Configuration File Requirements

Each external agent's configuration file must contain:

1. **Constitution reference**: A clear path or URL pointing to the governing constitution
2. **Security红线 (red lines) summary**: The non-negotiable rules (push restrictions, remote restrictions, credential handling)
3. **Document routing rules**: Where different types of output should be placed (architecture docs → docs/, cross-project strategy → strategic directory, audit records → audit directory)
4. **Supremacy clause**: A statement that if the configuration file conflicts with the constitution, the constitution prevails

### 4. Document Source Hierarchy (Cortical Layer Model)

When external agents reference documentation for decision-making, they should use a prioritized source hierarchy:

```
L0 (authoritative source): Internal knowledge base / strategy directory
    Purpose: Governance, planning, audit documents
    Characteristics: Version-controlled, indexed, single authoritative copy

L1 (code companion): Repository docs/ directory
    Purpose: API reference, development guides, operations manual
    Characteristics: Ships with code, updated alongside implementation

L2 (public reference): Published website / public repository
    Purpose: Community interaction, public information confirmation
    Characteristics: Never used for internal decisions, no internal paths
```

**Rules:**
- Agents should prefer the highest-priority source available (L0 > L1 > L2)
- If L0 doesn't have the needed document, degrade to L1 and log the gap
- L2 is for external communication only — never reference L2 for internal decisions
- Violations of hierarchy rules are governance infractions

## When to Apply

- Your repository is worked on by **multiple AI coding assistants** with different config formats
- Your repository contains **sensitive or proprietary code** that must not be pushed accidentally
- You've experienced or want to prevent **agents pushing to unauthorized remotes**
- You want **consistent behavior** across Claude Code, Cursor, Copilot, etc.

## Common Pitfalls

- **Config drift**: Each agent's config file is maintained independently. Without a centralized source of truth (constitution), they drift apart. The constitution reference solves this.
- **Preflight as an afterthought**: Agents are optimized to start quickly. Adding a preflight step feels like friction — but it's the only reliable way to catch misconfiguration before damage is done.
- **Sub-agent blind spot**: When an agent spawns a sub-agent to execute a task, the sub-agent often doesn't inherit the governance context. The spawning agent must explicitly pass preflight requirements.
- **False sense of coverage**: Just because a config file exists doesn't mean the agent reads it. Verify that each agent type actually consumes its config file in practice.

## Example: Preflight Script Skeleton

```bash
#!/bin/bash
# Preflight check for external AI agents

echo "=== Preflight Check ==="

# 1. Remote verification
REMOTES=$(git remote -v)
echo "[1/5] Checking git remotes..."
if echo "$REMOTES" | grep -q "unauthorized-remote"; then
  echo "❌ Unauthorized remote found. Aborting."
  exit 1
fi
echo "✅ Remotes OK"

# 2. Hook verification
echo "[2/5] Checking security hooks..."
if [ ! -f ".git/hooks/pre-push" ]; then
  echo "❌ pre-push hook missing. Aborting."
  exit 1
fi
echo "✅ Hooks OK"

# 3. Policy acknowledgment
echo "[3/5] Policy acknowledgment required..."
echo "  This repository restricts push targets."
echo "  See CONSTITUTION.md for details."

# 4. Push denial (checked at runtime via hook)
echo "[4/5] Push guard active"

# 5. Remote config denial (checked at runtime via hook)
echo "[5/5] Remote config guard active"

echo "=== Preflight Complete ==="
```
