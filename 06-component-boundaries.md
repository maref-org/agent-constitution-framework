# Pattern 6: Component Boundary Patterns

## What It Solves

When you have a system with both open-source and proprietary components, or multiple packages within a monorepo, the boundaries between them must be explicit and enforced. Without clear boundary rules, internal code leaks into public packages, proprietary algorithms become accidentally open-sourced, and dependency graphs become tangled.

Component Boundary Patterns provide **import discipline**, **protocol-layer communication**, **degradation strategies**, and a **dual identity model** for components that exist in both internal and external contexts.

## Key Concepts

### 1. Import Discipline: Package-Level vs. Path-Level

**Rule**: Cross-boundary dependencies must use package-level imports, never path-level imports.

```
❌ Bad:  import ../public/packages/maref/src/core
✅ Good: import maref
```

**Why**: Path-level imports create tight coupling to the internal directory structure. If the directory structure changes (e.g., moving from monorepo to multi-repo), every path-level import breaks. Package-level imports resolve through the package manager, which provides versioning, dependency resolution, and clean boundaries.

**Enforcement**:
- CI rule: Scan for path patterns that cross package boundaries (`../`, `../../packages/`)
- Fail CI if any cross-boundary path imports are found
- Exception: Internal-to-internal imports within the same package can use relative paths

### 2. Protocol-Layer Cross-Boundary Communication

When two components need to communicate across a boundary (e.g., proprietary → open-source), they should do so through a **protocol layer**, not through direct function calls.

**Protocol requirements:**

```
Request:
{
  "trace_id":    "<uuid-v4>",       // Required: full-chain trace ID
  "timestamp":   "<ISO-8601>",      // Required: message creation time
  "source":      "<component-id>",  // Required: caller identity
  "api_version": "1.0.0",           // Required: semver contract
  "payload":     { ... }            // Actual request data
}

Response:
{
  "trace_id":    "<uuid-v4>",       // Must echo request trace_id
  "status":      "ok" | "error" | "degraded",
  "payload":     { ... } | null,
  "error":       { ... } | null
}
```

**Required fields on every cross-boundary call:**
- `trace_id`: Enables end-to-end tracing across component boundaries
- `timestamp`: Enables ordering, timeout detection, and audit
- `source`: Enables per-component authorization and rate limiting
- `api_version`: Enables contract evolution without breaking consumers

**Rejection rules:**
- Missing `trace_id` → reject (400 Bad Request)
- Missing `timestamp` or `source` → warn, but allow (degraded mode)
- Version mismatch → resolve by consumer's declared compatibility range

### 3. Degradation Strategies (FAIL_MODE)

Every cross-boundary call must declare a failure mode:

| Mode | Behavior | Use Case |
|------|----------|----------|
| `open` | Service unavailable → skip the call, mark result as `governance_bypassed=true`, continue execution | Low-safety scenarios, non-critical features |
| `closed` | Service unavailable → block the request, fail the calling operation | High-safety scenarios, security-critical paths |

**Decision factors for choosing FAIL_MODE:**
- Does this call affect security or data integrity? → `closed`
- Is this call for analytics, logging, or non-critical enrichment? → `open`
- Can the system produce a correct result without this call? → `open`
- Would silently skipping this call create a safety risk? → `closed`

### 4. Dual Identity Model

Some components exist simultaneously as an **internal reference implementation** and an **external standalone product**.

```
                    ┌──────────────────────────────┐
                    │       Component X             │
                    ├──────────────────────────────┤
                    │  Identity A                   │  Identity B
                    │  (reference implementation)   │  (standalone product)
                    ├──────────────────────────────┤
                    │  Used by: internal system     │  Used by: external community
                    │  Location: internal workspace  │  Location: public repository
                    │  Versioning: pinned to system  │  Versioning: independent semver
                    └──────────────────────────────┘
```

**Rules for dual-identity components:**

1. **Single codebase, two entry points**: The same source code can serve both identities, but they have different packaging and different entry points
2. **Synchronization direction is one-way**: Internal → External. Community feedback flows back through human review, not automatic PR merges
3. **External identity must not expose internal state**: The external build strips internal-only code paths behind feature flags or build-time exclusions
4. **License independence**: The external product has its own license, unaffected by the internal system's license

## When to Apply

- You have a **mixed open-source / proprietary** codebase with shared components
- Your system has **multiple packages** with complex dependency relationships
- You're planning to **extract components** from a monorepo to standalone repositories
- You need **tracing and audit** across service boundaries

## Common Pitfalls

- **Path-import habit**: Developers naturally use relative imports because they're faster to write. CI enforcement is the only reliable prevention.
- **Protocol as ceremony**: Adding `trace_id`, `timestamp`, and `api_version` feels like overhead — until you need to debug a cross-component failure and have no traceability.
- **FAIL_MODE inconsistency**: Different components choose different failure modes for the same type of call, creating unpredictable system behavior. Standardize FAIL_MODE assignment by call category.
- **Dual identity drift**: Over time, the internal and external versions of a component diverge. Establish a regular sync cadence and automated compatibility tests.

## Example: CI Import Boundary Check

```yaml
jobs:
  check-boundaries:
    steps:
      - name: Check no path-based cross-boundary imports
        run: |
          violations=$(grep -rn "from '\.\." internal/src/ | grep -v "__init__" || true)
          if [ -n "$violations" ]; then
            echo "❌ Cross-boundary path imports detected:"
            echo "$violations"
            exit 1
          fi
      - name: Check all cross-boundary calls declare api_version
        run: |
          violations=$(grep -rn "call_service\|send_request" --include="*.py" -A5 \
            | grep -v "api_version" || true)
          if [ -n "$violations" ]; then
            echo "⚠️ Calls without api_version detected"
          fi
```
