# Agent Constitution Framework

**A governance pattern collection for AI agent ecosystems.**

Builders of AI agent systems inevitably face a common set of challenges: How do you ensure agents follow the rules? How do you safely disclose capabilities to the public? How do you manage agents across multiple repositories? How do you handle the boundary between open-source and proprietary code?

This framework captures **generalizable governance patterns** extracted from production experience operating a multi-agent, multi-repository AI ecosystem. Each pattern is documented as a standalone guide — adapt the concepts to your own context rather than copying verbatim.

## Motivation

As AI agents become more autonomous and are granted more tools (file write, command execution, network access), the question shifts from *"can the agent do this?"* to *"should the agent do this?"* Traditional code review and CI/CD pipelines are insufficient because agents operate at machine speed, generate novel code paths, and cross repository boundaries.

Governance patterns help answer:

- **Constitutional governance**: What is the supreme rule set, and how do you enforce it consistently?
- **Progressive disclosure**: How do you safely release internal capabilities to the public without leaking sensitive context?
- **Agent lifecycle**: How do you register, monitor, and retire agents in a multi-agent system?
- **External agent governance**: How do you control AI coding assistants that operate inside your repository?
- **Document governance**: How do you maintain a single source of truth across dozens of design documents?
- **Component boundaries**: How do you keep open-source and proprietary code cleanly separated while allowing them to communicate?

## Patterns

| # | Pattern | Description | Source Concepts |
|---|---------|-------------|-----------------|
| 1 | [Constitutional Governance](01-constitutional-governance.md) | Supreme rule hierarchy, amendment process, self-check mechanism | Constitutional supremacy, self-audit |
| 2 | [Progressive Disclosure](02-progressive-disclosure.md) | Multi-tier disclosure model, phase gates, leak prevention | T3-T0 tiers, narrative transformation |
| 3 | [Agent Lifecycle Management](03-agent-lifecycle.md) | Registration, health states, task queue protocol | Agent identity, heartbeat, task queue |
| 4 | [External Agent Governance](04-external-agent-governance.md) | Scope definition, preflight checks, config requirements | Code agent preflight, document hierarchy |
| 5 | [Document Governance](05-document-governance.md) | Single source of truth, priority hierarchy, archive semantics | File lifecycle, completion proof |
| 6 | [Component Boundary Patterns](06-component-boundaries.md) | Import discipline, protocol-layer communication, degradation strategies | Package isolation, FAIL_MODE, dual identity |

## How to Use

Each pattern document is self-contained. You can:

1. **Read sequentially** if you're building governance from scratch
2. **Pick individual patterns** that match your current pain point
3. **Adopt incrementally** — start with Constitutional Governance (the foundation), then layer on Progressive Disclosure or Agent Lifecycle as your system grows

The patterns intentionally avoid prescribing specific tools or platforms. They describe **what** to govern and **why**, not the specific implementation. Adapt the tier definitions, checklists, and state machines to your technology stack.

## Relationship to MAREF

This framework is the **policy layer** — it defines *what rules should exist*. [MAREF](https://maref.cc) (Multi-Agent Recursive Evolution Framework) provides the **enforcement layer** — it intercepts tool calls and enforces the rules at runtime. They are complementary:

```
 ┌─────────────────────────────────────┐
 │  Agent Constitution Framework       │  ← Policy (this repo)
 │  "What should we prevent?"          │
 ├─────────────────────────────────────┤
 │  MAREF Governance Plugin            │  ← Enforcement (maref-org/maref-governance)
 │  "How do we prevent it?"            │
 └─────────────────────────────────────┘
```

## License

Apache-2.0 — see [LICENSE](LICENSE).
