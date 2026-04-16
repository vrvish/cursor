---
name: vrvish-bridge-mcp
description: >-
  Uses the vrvish-bridge-v6 MCP server (MemoryBankSync, CommandDispatcher,
  RefactorHelper) for memory sync, authorised command execution, and refactor
  assistance. Use when the user mentions the bridge MCP, asks to check status,
  read or sync memory or context, run or dispatch local checks or commands,
  or clean up, refactor, or optimise specific files with bridge-backed help.
---

# vrvish-bridge-v6 MCP

## Server

- **MCP server identifier (for `call_mcp_tool`):** `user-vrvish-bridge-v6`
- **Human-readable name:** vrvish-bridge-v6

Before any tool call, read the tool descriptor under the MCP folder if parameters are unclear. Do not invent argument shapes.

---

## 1. Memory bank management — `MemoryBankSync`

**When to use:** User asks to check status, read memory, sync context, or catch up at session start; whenever answers should reflect the latest persisted knowledge stack.

**Action:** Call `MemoryBankSync` **before** answering, when those triggers apply.

**Arguments (schema):**

| Field | Required | Notes |
|--------|----------|--------|
| `syncMode` | Yes | One of: `registry`, `tier2-index`, `tier3-qdrant`, `full` |
| `scope` | No | String array |
| `deltaOnly` | No | Boolean |

**Default:** If the user does not specify a mode, prefer `full` for “sync everything / read my memory / where we left off”; use narrower modes only when they name a specific layer.

---

## 2. Command execution — `CommandDispatcher`

**When to use:** User asks to run a check, dispatch something, or execute a local system command that should go through the bridge policy plane.

**Action:** Build the payload and call `CommandDispatcher`. Execute via the tool; report outcomes—do not only explain how to run locally.

**Arguments (schema):**

| Field | Required | Notes |
|--------|----------|--------|
| `commandId` | Yes | Bridge-registered command identifier |
| `arguments` | No | Object passed to that command |
| `workingDirectory` | No | Working directory string |

If the user gives a raw shell line, map it to the correct `commandId` and `arguments` expected by the bridge (check bridge docs or existing usage). If no mapping exists, say so instead of guessing unsafe commands.

---

## 3. Code refactoring — `RefactorHelper`

**When to use:** User asks to clean up, refactor, or optimise a specific file or path.

**Action:** Call `RefactorHelper` with the target path(s), then apply returned architectural suggestions in the editor. Keep edits scoped to what the user asked for.

**Arguments (schema):**

| Field | Required | Notes |
|--------|----------|--------|
| `targetPaths` | Yes | Array of file or directory paths |
| `intent` | Yes | One of: `analyze`, `plan`, `rewrite`, `migrate` |
| `projectId` | No | String |
| `technologyHints` | No | String array (e.g. stack hints) |

**Intent guide:**

- **`analyze`** — Understand structure and issues before changing code.
- **`plan`** — Produce a refactor plan; user may want review before edits.
- **`rewrite`** — User wants concrete code changes; apply suggestions after validation.
- **`migrate`** — Technology or pattern migration.

Match `intent` to the user’s wording (e.g. “optimise this file” → often `analyze` then `rewrite` if they want edits).

---

## Operational rules

1. Prefer these tools over guessing when the user clearly wants bridge-backed memory, dispatch, or refactor flows.
2. After `MemoryBankSync`, incorporate tool output into the reply; do not ignore sync results.
3. After `CommandDispatcher`, summarise stdout/stderr or structured result faithfully.
4. After `RefactorHelper`, implement changes that align with project rules and the user’s scope; do not expand into unrelated refactors.
