# Pattern 2: Progressive Disclosure

## What It Solves

AI agent systems often develop capabilities and knowledge that shouldn't be immediately published to the public — not because they're malicious, but because they contain internal context (file paths, architecture decisions, cost data, API keys) that is meaningless or harmful outside the organization.

Progressive Disclosure provides a **graduated release framework**: different tiers of content are released at different times, to different audiences, with different levels of transformation.

## Key Concepts

### 1. Four-Tier Disclosure Model

```
T3 (internal raw)      → Zero external access
  原始内部内容，仅限核心团队

T2 (research partners) → 4-week cooling period → strategic partners
  脱敏后分享给研究合作方

T1 (developer docs)    → 2-week cooling period → public docs/
  通用化设计原则输出到开发者文档

T0 (public release)    → 2-week cooling after T1 → standalone repos
  独立、自洽的公开发布
```

Each tier defines:

- **Access scope**: Who can see it
- **Cooling period**: Minimum time before promotion to the next tier
- **Transformation required**: What must be done before promotion

### 2. Phase Gates for Content Release

Before content moves from a higher tier to a lower tier, it must pass through gates:

| Gate | Check | Blocking |
|------|-------|----------|
| Gate 0 | No internal file paths (e.g., `/Volumes/`, `C:\Users\`) | ❌ Block if any found |
| Gate 1 | No internal repository names or URLs | ❌ Block if any found |
| Gate 2 | No provider names, cost data, or pricing | ❌ Block if any found |
| Gate 3 | No IP addresses, network topology, or timestamps | ❌ Block if any found |
| Gate 4 | No API keys, tokens, or credentials | ❌ Block if any found |
| Gate 5 | Narrativization complete (see below) | ❌ Block if not transformed |

### 3. Narrativization (Context Stripping)

Before internal content can be published externally, it must be **narrativized** — stripped of internal context and rewritten as a self-contained narrative:

1. **Strip**: Remove all file paths, team names, internal architecture references
2. **Abstract**: Replace specific implementation details with general design principles
3. **Rewrite**: Produce a self-consistent public narrative that doesn't require internal knowledge to understand
4. **Verify**: Run an automated check to confirm no internal paths, keys, or tokens remain

### 4. Leak Prevention Checklist

The following content categories are **never** appropriate for public release:

- File system paths (any platform)
- Internal repository names and URLs
- Provider names and cost/pricing data
- IP addresses and internal network topology
- Precise timestamps of internal events
- Dependency graphs (can be reverse-engineered for attack surface)
- API keys, tokens, or credentials of any kind
- Team member names (unless explicitly approved for public attribution)

## When to Apply

- You're developing in a **private workspace** and publishing to **public repositories**
- Your system contains a mix of **proprietary algorithms** and **open-source components**
- You have **research partners** who get early access to certain capabilities
- You've experienced (or want to prevent) **accidental leaks** of sensitive context

## Common Pitfalls

- **No cooling period**: Moving directly from T3 to T0 (internal → public) without intermediate tiers means no time for review. Mandate minimum cooling periods proportional to sensitivity.
- **Narrativization as an afterthought**: The most common leak pattern is a developer thinking "this file doesn't contain anything sensitive" when it contains paths, endpoints, or architecture context that enables attack surface reconstruction.
- **Automated checks are not enough**: Gates 0-4 (path scanning, key detection) can be automated. Gate 5 (narrativization quality) requires human judgment. Don't skip it.
- **Tier drift**: Content that was T1 at creation may become T3-capable as the system evolves. Periodically re-audit published content against current sensitivity standards.

## Example: CI Pipeline

```yaml
# Triggered on PR to public release branch
jobs:
  disclosure-check:
    steps:
      - name: Gate 0 — Path leak check
        run: check-for-paths.sh
      - name: Gate 1 — Internal URL check
        run: check-for-internal-urls.sh
      - name: Gate 4 — Credential check
        run: check-for-credentials.sh
      - name: Gate 5 — Narrativization check
        run: check-narrative-transform.sh
```
