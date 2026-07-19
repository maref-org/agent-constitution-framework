# Pattern 3: Agent Lifecycle Management

## What It Solves

In a multi-agent system, how do you know which agents are running? How do you detect when an agent crashes? How do you assign tasks without conflicts? Without lifecycle management, agents operate as independent silos — they don't know about each other, duplicate work, and failures go undetected.

Agent Lifecycle Management provides a **registration → health monitoring → task execution → retirement** protocol that works across heterogeneous agents.

## Key Concepts

### 1. Agent Registration & Identity

Every agent in the ecosystem must register before performing work:

**Registration fields:**
- `agent_name`: Human-readable identifier
- `agent_type`: Category (runner, patrol, daemon, server)
- `capabilities`: Declared abilities (free-form list)
- `started_at`: ISO-8601 timestamp

**Identity format**: `agent_name@agent_type` — a unique identifier for audit trails and logging.

**Lifecycle events:**
- **Register**: On startup, the agent announces itself to a registry
- **Heartbeat**: Periodic signal that the agent is still alive
- **Deregister**: On graceful shutdown, the agent removes itself
- **Abnormal detection**: The patrol/health system detects missing heartbeats and marks the agent as dead

### 2. Health State Machine

Each agent exposes a health state with automatic transitions:

```
         ┌──────────┐
         │ starting │  ← Agent launched, not ready
         └────┬─────┘
              │ ready signal
              ▼
         ┌──────────┐
         │ healthy  │  ← Normal operation, accepting tasks
         └────┬─────┘
              │
    ┌─────────┼──────────┐
    │         │          │
    ▼         ▼          ▼
┌────────┐ ┌────────┐ ┌──────┐
│degraded│ │  dead  │ │healthy│
│(limited│ │(missed │ │(still │
│function)│ │3 HB)   │ │fine)  │
└────────┘ └────────┘ └──────┘
```

| State | Definition | Automatic Action |
|-------|-----------|-----------------|
| `starting` | Agent launched, not yet ready | No tasks assigned |
| `healthy` | Normal operation | Tasks can be assigned |
| `degraded` | Running with limited functionality (e.g., dependent service unavailable) | Low-priority tasks only |
| `dead` | Heartbeat missed for 3× the expected interval | Alert → log → attempt auto-restart |

**Heartbeat protocol:**
- Default interval: 30 seconds (configurable via environment variable)
- Expected fields: `agent_name`, `timestamp`, `status`, `current_task_id` (if any)
- Storage: Registry records the last N heartbeats for diagnostics

### 3. Task Queue Protocol

For runner-type agents that execute tasks from a shared queue:

```
                    ┌─────────────────────────────────────┐
                    │           Queue Directory            │
                    │         agent-queue/        │
                    ├─────────────────────────────────────┤
                    │  task_001.json  │ status: pending    │
                    │  task_002.json  │ status: running    │← claimed by runner-a
                    │  task_003.json  │ status: completed  │
                    └─────────────────────────────────────┘
                               ▲
                               │ claim via atomic rename
                               │
                    ┌──────────────────┐
                    │  Runner Agent    │
                    │  (multiple       │
                    │   instances)     │
                    └──────────────────┘
```

**States:**
1. `pending` → Task is available for claiming
2. `running` → Agent has claimed the task, wrote `agent_name` + `started_at`
3. `completed` → Agent finished, wrote `finished_at` + `output`
4. `failed` → Agent failed, wrote `error` + `retry_count`
   - If `retry_count < max_retries`: auto-reset to `pending`
   - If `retry_count >= max_retries`: remains `failed` for human review

**Key design: Lock-free claiming**
Multiple agents can compete for tasks without a central scheduler. They use **atomic filesystem operations** (e.g., `rename` with `O_EXCL`, or CAS on a status field) to claim tasks. An agent reads a task, atomically transitions it to `running` with its name, and if the transition succeeds — it owns the task.

## When to Apply

- You have **multiple agents** that could conflict over shared resources
- You need **visibility** into which agents are running and what they're doing
- You want **failure detection** without relying on external monitoring
- You have **competing consumers** that need a shared task queue

## Common Pitfalls

- **No heartbeat**: Without heartbeats, you can't distinguish between "agent is thinking" and "agent crashed 10 minutes ago." Always implement heartbeats with a dead-man's-switch timeout.
- **Identity collision**: Two agents with the same `agent_name@agent_type` will corrupt each other's state. Enforce uniqueness at the registry level.
- **Queue poisoning**: A failed task that keeps resetting to `pending` (because `retry_count < max_retries`) can consume all available workers in a tight loop. Implement exponential backoff on retries.
- **No deregistration on shutdown**: Clean shutdown should remove the agent from the registry. Otherwise, a crash loop will accumulate zombie registrations.

## Integration with Governance

Agent lifecycle data feeds into the broader governance system:

- **Audit trail**: Every task execution is logged with `agent_name`, timestamps, and outcome
- **Health monitoring**: Degraded agents can be automatically excluded from critical task assignments
- **Capacity planning**: Registry data shows how many agents of each type are running
- **Security**: Unknown (unregistered) agents can be detected and flagged
