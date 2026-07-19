# Pattern 5: Document Governance

## What It Solves

As an AI agent ecosystem grows, documents proliferate: design docs, architecture decisions, audit reports, configuration files, release notes, internal wikis. Without governance, the same information exists in multiple places with conflicting versions, outdated documents are confused with current ones, and nobody knows which document is authoritative.

Document Governance provides **single-source-of-truth discipline**, a **file lifecycle model**, and **priority rules** for resolving conflicts.

## Key Concepts

### 1. Single Source of Truth

For any given piece of information, there should be exactly one authoritative location. All other references are derived or cached.

**Rule**: If the same information appears in two places, one of them must reference the other as its source. Both must not be independently maintained.

```
❌ Bad: Two files both contain the API endpoint list
✅ Good: One file has the endpoints, other files reference it
```

**Enforcement**: When a document is updated, sweep all documents that reference it and verify consistency. This can be automated with cross-reference checks.

### 2. File Lifecycle

Every document passes through a defined lifecycle:

```
Draft (working directory)
  → Review (inbox/review queue)
  → Published (target directory, finalized)
  → Archived (superseded or completed)
```

**States:**

| State | Definition | Access | Can Be Modified? |
|-------|-----------|--------|-----------------|
| Draft | Being written, not yet reviewed | Author only | Yes |
| Review | Submitted for review, awaiting approval | Reviewers | Minor corrections only |
| Published | Approved and finalized | All readers | Only via amendment |
| Archived | Superseded by newer version or completed | Archived directory | No (historical record) |

**Transition rules:**
- Draft → Review: Author submits, must include completion evidence
- Review → Published: Reviewer approves, document is finalized
- Published → Archived: A newer version supersedes it, or the task/project is complete

### 3. Priority / Authority Hierarchy

When multiple documents give conflicting instructions, resolve by priority tier:

```
Tier 1: Constitution (supreme rules of the system)
Tier 2: Project state definition (current phase, active constraints)
Tier 3: File management scheme (routing, naming, archiving rules)
Tier 4: Agent behavior guidelines (per-agent or per-repository configs)
Tier 5: Historical documents (reference only, not binding)
```

This hierarchy ensures that foundational rules (Tier 1) always override tactical decisions (Tier 4) and outdated plans (Tier 5).

### 4. Archive Semantics

Archiving a document has specific semantics that must be explicitly declared:

| Archive Category | Semantics | Entry Condition |
|-----------------|-----------|-----------------|
| Completed | Task done, document delivered | Meets completion proof standard |
| Superseded | Old version replaced by new | Explicit supersession annotation + path to replacement |
| Milestone | Project milestone artifact | Milestone achieved + all CI checks passing |

**Anti-pattern**: Moving low-quality documents to an archive directory to improve quality metrics. Archiving doesn't change the quality of a document — it only changes its location. Quality issues must be fixed, not hidden.

### 5. Completion Proof Standard

Any claim of "done" must be accompanied by verifiable evidence. At least one of:

1. **Code path coordinates**: Physical file path + line range of changes (e.g., `src/api/handler.ts:154-165`)
2. **Command execution log**: Raw terminal output showing the before/after state
3. **CI run link**: GitHub Actions run ID or equivalent, with pass/fail status
4. **Test result summary**: Raw test framework output (e.g., `vitest run` output with counts)

If none of the above can be provided → the item is considered a **declaration**, not a completion. It doesn't count toward any completion metric and is not displayed as "done" in dashboards.

## When to Apply

- You have **more than 20-30 design/planning documents** and can't easily find the current version
- Different team members (or agents) reference **different versions** of the same document
- You're spending time **resolving contradictions** between documents instead of implementing
- You have **archived documents** that people still reference as if they were current

## Common Pitfalls

- **No lifecycle enforcement**: Without CI checks or review workflows, documents stay in "Draft" forever and are treated as authoritative.
- **Hierarchy ignorance**: Lower-tier documents contradicting the constitution go unnoticed because nobody checks. Enforce with automated cross-referencing.
- **Archive as dumping ground**: When a document is superseded, readers still find the old version first. Use redirects or directory manifests to guide readers to the current version.
- **Completion theater**: Claims of "done" without verifiable evidence create a false sense of progress. The completion proof standard is a forcing function for accountability.

## Example: CI Check

```yaml
# Verify no two documents claim to be the authoritative source for the same thing
jobs:
  document-consistency:
    steps:
      - name: Check for duplicate authoritative claims
        run: |
          duplicates=$(grep -r "authoritative" docs/ | cut -d: -f1 | sort | uniq -d)
          if [ -n "$duplicates" ]; then
            echo "❌ Multiple documents claim authority:"
            echo "$duplicates"
            exit 1
          fi
      - name: Check for draft documents in published directory
        run: |
          drafts=$(grep -rl "status: draft" docs/published/ 2>/dev/null)
          if [ -n "$drafts" ]; then
            echo "⚠️ Published directory contains draft documents:"
            echo "$drafts"
          fi
```
