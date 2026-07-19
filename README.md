<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://maref.cc/brand/agent-constitution-framework-dark.svg">
  <img alt="Agent Constitution Framework" src="https://maref.cc/brand/agent-constitution-framework-light.svg">
</picture>

# Agent Constitution Framework

**Governance patterns for AI agent ecosystems — from single-assistant repositories to multi-agent, multi-repository systems.**

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![MAREF](https://img.shields.io/badge/governance-MAREF-7B2FBE)](https://maref.cc)

---

## Why This Exists

AI agents are no longer experimental toys. They write code, execute commands, access files, call APIs, and deploy to production. The question has shifted from *"can the agent do this?"* to *"should the agent do this?"*

Most teams start with **one** Claude Code or Cursor config file and a few ad-hoc rules. That works — until:

- You have **multiple agents** (Claude Code + Cursor + GitHub Copilot + Opencode) giving inconsistent answers because each reads a different config file
- You accidentally **commit an API key** or **internal file path** to a public repository because no one thought to check
- Two agents **duplicate each other's work** because they don't share a task queue
- An agent **pushes to the wrong remote** because there's no preflight check
- You have **20 design documents** with conflicting instructions and no one knows which one is current

This framework captures **six governance patterns** that solve these problems. They were extracted from production operation of a multi-agent, multi-repository AI ecosystem — but written to be **tool-agnostic and adaptable** to your stack.

---

## The Patterns

| # | Pattern | Problem It Solves | Key Concepts |
|---|---------|-------------------|-------------|
| 1 | **[Constitutional Governance](01-constitutional-governance.md)** | Multiple agents read different config files with conflicting rules. Which one wins? | Supremacy hierarchy, amendment process, self-check (human principles + CI enforcement) |
| 2 | **[Progressive Disclosure](02-progressive-disclosure.md)** | Internal context (paths, endpoints, costs, keys) leaks to public repos during release. | 4-tier disclosure model, phase gates, narrativization, leak prevention checklist |
| 3 | **[Agent Lifecycle Management](03-agent-lifecycle.md)** | Agents crash silently, duplicate work, or run without anyone knowing. | Registration + identity, health state machine, lock-free shared task queue |
| 4 | **[External Agent Governance](04-external-agent-governance.md)** | AI coding assistants (Claude Code, Cursor, Copilot) each have their own config and tool access. | Scope map, preflight checks, config requirements, document source hierarchy |
| 5 | **[Document Governance](05-document-governance.md)** | Design docs proliferate, contradict each other, and nobody knows what's current. | Single source of truth, file lifecycle, priority tiers, archive semantics, completion proof |
| 6 | **[Component Boundaries](06-component-boundaries.md)** | Open-source and proprietary code get tangled. Internal dependencies leak to public packages. | Package-level imports, protocol-layer communication, FAIL_MODE degradation, dual identity |

---

## Quick Start

**Pick your biggest pain point and start there.**

```bash
# Clone the framework
git clone https://github.com/maref-org/agent-constitution-framework.git

# Read the pattern that matches your current problem
open agent-constitution-framework/01-constitutional-governance.md
```

### Adoption Roadmap

The patterns are designed to be adopted **incrementally** — you don't need all six to get value:

```
Phase 0 (single agent, single repo)
  └── Start with Pattern 1 (Constitutional Governance)
  └── Add Pattern 5 (Document Governance) if docs are already multiplying

Phase 1 (multiple agents, same repo)
  └── Add Pattern 4 (External Agent Governance) — unify config files
  └── Add Pattern 3 (Agent Lifecycle) — track who's running

Phase 2 (mixed open-source/proprietary)
  └── Add Pattern 2 (Progressive Disclosure) — safe public release
  └── Add Pattern 6 (Component Boundaries) — clean separation

Phase 3 (multi-repo ecosystem)
  └── Layer all six patterns across repositories
  └── Establish constitution hierarchy across repos
```

Each pattern document is self-contained: read it in isolation, implement what fits, skip what doesn't.

---

## Relationship with MAREF

The Agent Constitution Framework is the **policy layer**. [MAREF](https://maref.cc) (Multi-Agent Recursive Evolution Framework) is the **enforcement layer**. They are complementary:

```
              ┌─────────────────────────────────────┐
   Policy     │  Agent Constitution Framework        │
   (WHAT)     │  Defines rules and governance models │
              └──────────────────────┬──────────────┘
                                     │ rules
                                     ▼
              ┌─────────────────────────────────────┐
 Enforcement  │  MAREF Governance Plugin             │
   (HOW)      │  Intercepts tool calls at runtime    │
              │  Enforces policies via sidecar        │
              │  Blocks violations before they happen │
              └─────────────────────────────────────┘
```

- **This repo** answers "what should we prevent and why?"
- **[maref-org/maref-governance](https://github.com/maref-org/maref-governance)** answers "how do we prevent it at runtime?"

---

## FAQ

**Q: Do I need all six patterns to benefit?**  
A: No. Each pattern is independent. Start with the one that matches your current pain point.

**Q: Are these tied to a specific programming language or platform?**  
A: No. The patterns describe concepts (tiered disclosure, heartbeat protocols, import disciplines) that apply to any language or stack.

**Q: Is this specific to Claude Code / Cursor / Copilot?**  
A: The External Agent Governance pattern references common AI coding tools, but the pattern itself — preflight checks, config unification, document hierarchy — applies regardless of which agents you use.

**Q: How much overhead do these patterns add?**  
A: That's up to you. The framework provides principles; you decide the implementation depth. A minimal adoption (one-page constitution + one preflight script) takes an afternoon. Full adoption across 6 patterns is a multi-month journey.

**Q: Is this framework production-tested?**  
A: Yes. The patterns were extracted from a multi-agent, multi-repository AI ecosystem operating in production since early 2026.

---

## License

Apache-2.0 — see [LICENSE](LICENSE).

---

*Built for agent ecosystems that need rules, not just tools.*
