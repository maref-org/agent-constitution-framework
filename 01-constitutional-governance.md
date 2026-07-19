# Pattern 1: Constitutional Governance

## What It Solves

In an AI agent ecosystem, there are many sources of authority: configuration files, agent instructions, CI rules, design documents, runtime policies. When they conflict, which one wins? Without a clear hierarchy, agents make arbitrary choices — sometimes with security or safety consequences.

A **constitution** is a single, supreme document that defines the foundational rules of the system. All other policies derive from it and must be consistent with it.

## Key Concepts

### 1. Constitution Supremacy

The constitution sits at the top of the authority hierarchy. Every downstream document (agent configs, CI rules, design docs, release policies) must be consistent with the constitution. If a conflict is found, the constitution prevails.

```
Constitution (supreme law)
  └── Agent configuration files (CLAUDE.md, AGENTS.md, etc.)
  └── CI/CD rule definitions
  └── Design documents and architecture decisions
  └── Release and disclosure policies
```

**Implementation requirement**: Every downstream document should contain an explicit statement like: *"If this document conflicts with the constitution, the constitution prevails."*

### 2. Amendment Process

Constitutions should be **difficult to change, but not impossible.** A formal amendment process prevents casual modification:

1. **Proposal**: A change document is drafted, describing the motivation and specific modifications
2. **Human review**: A human (not an agent) reviews and approves the change
3. **Version bump**: The constitution version is incremented (semver-style)
4. **Downstream sync**: Within a defined timeframe (e.g., 24 hours), all affected downstream documents are updated to reference the new version
5. **Audit trail**: Each amendment is logged with date, author, and scope of impact

### 3. Self-Check Mechanism

A constitution without enforcement is aspirational. A self-check mechanism has two layers:

**Layer 1: Human-readable principles** — A checklist that an agent or reviewer can walk through:

```
☐ Repository remote configuration is authorized
☐ No internal code paths are referenced in public modules
☐ Cross-boundary API calls declare version contracts
☐ All agents are registered with heartbeats active
☐ External agent configuration files reference the constitution
```

**Layer 2: Machine-readable CI rules** — A table mapping constitutional articles to automated checks:

| Article | CI Rule File | Trigger | Enforcement |
|---------|-------------|---------|-------------|
| Remote isolation | `pre-push` hook | Any `git push` | Unauthorized remote → block |
| Import boundary | `check-boundaries.yml` | PR to internal code | Direct path import → fail |
| API versioning | `check-api-version.yml` | PR to API schema | Missing version field → fail |

## When to Apply

- You have **multiple agents** operating in the same codebase with different tools and permissions
- You're managing a **mixed open-source/proprietary** codebase
- You have **multiple configuration files** (CLAUDE.md, .cursorrules, opencode.jsonc, etc.) that need consistent rules
- You're experiencing **contradictory behavior** between agents due to conflicting instructions

## Common Pitfalls

- **Constitution too long**: A constitution should be the *supreme* document, not an exhaustive manual. Keep it to foundational rules. Detailed procedures belong in subordinate documents.
- **No enforcement mechanism**: A constitution that is never checked against reality is theater. Pair every article with either a human-readable checklist item or a machine-enforceable CI rule.
- **No amendment process**: Without a formal amendment process, the constitution becomes frozen (nobody dares change it) or irrelevant (everyone ignores it and makes their own rules).
- **Sync debt**: After amending the constitution, failing to update downstream documents creates drift. Set a hard deadline (e.g., 24 hours) and audit compliance.

## Example: Authority Hierarchy

When multiple documents give different instructions for the same situation, resolve by priority:

1. **Constitution** (supreme — foundational rules)
2. **Project state definition** (current phase, active constraints)
3. **File management scheme** (where files live, how they route)
4. **Agent behavior guidelines** (per-agent configuration)
5. **Historical planning documents** (reference only, not binding)
