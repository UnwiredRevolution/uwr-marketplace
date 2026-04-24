---
name: remotelink
description: >-
  This skill provides domain knowledge for RemoteLink, a script execution and
  endpoint management platform. It should be used when the user asks about
  RemoteLink tasks, jobs, clients, groups, monitors, or patch profiles — for
  example: "why did this job fail", "what tasks run on this client", "show
  upcoming scheduled jobs", "find clients in a group", "what does this task do",
  "investigate job failures", or "query RemoteLink data". Also relevant when the
  user encounters naming confusion (e.g., Groups vs GroupBases) or needs to
  understand the execution activity chain. Requires a connected RemoteLink MCP
  server providing search, get_doc, query_database, and execute_script tools.
version: "0.4"
---

# RemoteLink Skill

Domain wisdom, naming traps, and workflow guidance that the MCP tools can't tell you.
Use MCP tool descriptions for parameter details and `get_doc` for live schema — this skill covers what those sources miss.

## Core Glossary

- **Task** — A reusable script definition. References **Actions** by ID inside its `SerializedScript` JSON column and supplies parameters for each.
- **Action** — A parameterized operation (run script, copy file, etc.). Tasks embed action config in `SerializedScript` JSON (each step has `_actionId` + params). For reverse lookup (find tasks by action type), use `TaskActionVersionUsages` join table.
- **Job** — A scheduled execution plan: *when* to run a Task, *who* to run it on. Types: **Server** (server-only), **Interactive** (server+client with shared variables), **Monitor** (silent condition watcher).
- **Client** — A registered endpoint. Has one or more **Agents** (ClientCredentials rows), one Primary. The agent executes Actions.
- **Group** — A logical collection of Clients. **Static** (fixed membership) or **Dynamic** (rule-based). DB table is `GroupBases`, not `Groups`.
- **Monitor** — Condition watcher that triggers a Job. Types: File, Job, Request. Distinct from a Monitor-type Job.
- **Alert/Event** — Passive notification record from system activity.
- **Product** — Software title with versioned **Packages**.
- **Patch Profile** — Windows update rules linking Patch Approvals to Groups.

## Execution Activity Chain

```
Job (scheduled plan)
 ├─ what: Job → Tasks → Actions (via SerializedScript JSON, not FK)
 ├─ who:  Job → ClientJobs → Clients  (direct)
 │        Job → JobGroupBases → GroupBases → ClientGroupBases → Clients  (via groups)
 └─ activity:
      Job → JobRunRecord (one execution)
              → ClientJobRunRecord (one client's participation)
                  ├─ ClientJobRunAttempt (one try; may retry)
                  │      → ClientJobRunAttemptLog (detailed output)
                  └─ ClientJobRunEvents (event type indicators)
```

## Naming Traps

Things that bite every agent on first encounter:

| You might try | Actual name | Why |
|---|---|---|
| `Groups` | **`GroupBases`** | `Groups` doesn't exist as a queryable table. `GroupBases` unifies static and dynamic groups, with a `Discriminator` column (`'StaticGroup'`/`'DynamicGroup'`). |
| `Jobs` → `Clients` (direct) | **`ClientJobs`** join table | Direct Job→Client assignment uses `ClientJobs` (columns: `Client_Id`, `Job_Id`). Not obvious from the Job entity. |
| `Groups` → `Clients` | **`ClientGroupBases`** join table | Group membership is in `ClientGroupBases` (columns: `GroupBase_Id`, `Client_Id`), not `ClientGroups`. |
| Reading `Tasks.Actions` via JOIN | **`SerializedScript`** column | Task→Action link is a JSON array in `SerializedScript`, not a relational FK. Each element has `_actionId` (GUID) and parameter values. For reverse lookup (tasks using an action), use `TaskActionVersionUsages`. |
| `Result` / `Status` columns as strings | **Always integers** | All Result/Status columns are ints. Never assume meanings from column names — check actual values via `get_doc` or sample data. |
| Column not found in docs | **Query the real schema** | The database is EF6 Code First — the MCP docs are generated from C# entities and may not include join tables, computed columns, or convention-based names. Use `query_database` on `INFORMATION_SCHEMA.COLUMNS` to see the actual SQL schema. |

## Hidden Tables

Tables the MCP won't surface through obvious searches:

- **`ClientJobRunEvents`** — Has a `Type` column with event indicators (e.g., `ClientAlreadyRunningJob`, `Failed`, `ClientConfirmTimeout`). JOIN on `ClientJobRunRecordId`. Good for quick triage — for full failure details (exceptions, stack traces), use `execute_script` with `await rl.publicApi.getJobRunRecord({ IncludeAttemptLogs: true })`.
- **`UpcomingJobRuns`** — Filter `IsNextScheduledRun = 1` to get the actual next scheduled run. Without this filter, you'll also get future recovery runs.

## Tool Gotchas

- **`execute_script` is for reads AND writes** — It runs JavaScript in a Jint sandbox. `await rl.publicApi.getJobRunRecord({ ..., IncludeClientJobRunEvents: true })` returns richer data than raw SQL (human-readable result strings, event traces). Essential for failure diagnosis.
- **`rl.publicApi.*` calls are async** — Scripts run inside an async IIFE, so top-level `await` works. Every `rl.publicApi.*` method returns a Promise — `await` it (or chain `.then`). Forgetting `await` yields a Promise object, not the data.
- **Logging is via `console.*`** — `console.log` / `.debug` / `.info` / `.warn` / `.error`.
- **`return` for data, `console.*` for diagnostics** — An explicit top-level `return` surfaces a value back to `execute_script` as its result. Prefer returning structured data over dumping it into logs; reserve `console.*` for progress and context.
- **`ClientJobRunAttemptLogs.LogData` is varbinary** — Can't read via SQL. Use `execute_script` with `await rl.publicApi.getJobRunRecord({ IncludeAttemptLogs: true })` instead.
- **No `getTask` API** — There's no `rl.publicApi.getTask()`. To read task content, use `query_database` on `Tasks.SerializedScript`.
- **Discover API methods before calling them** — Use `search` with relevant keywords to find available `rl.publicApi.*` methods. Method names are camelCase (e.g., `getJobRunRecord`, not `GetJobRunRecord`).
- **`query_database` returns max 100 rows** — Use `COUNT(*)`, `GROUP BY`, or `TOP N` to work within this limit. Check for `"truncated": true` in results.
- **Extract only needed fields in scripts** — `console.log(JSON.stringify(fullResult))` can produce 700K+ characters. Select specific properties and `return` only what you need.

## Common Workflows

**Failure investigation:**
1. `execute_script` → `await rl.publicApi.getJobRunRecord({ IncludeAttemptLogs: true })` — primary path for full failure details (exceptions, stack traces, script output). LogData is varbinary, not SQL-readable.
2. Quick triage alternative: `query_database` on `ClientJobRunEvents` (JOIN on `ClientJobRunRecordId`, check `Type` column for event indicators).
3. Use `FailCount`/`SuccessCount` on `JobRunRecords` for reliable aggregates — avoids decoding `Result` integer values on `ClientJobRunRecords`.

**What does a task do?**
1. `query_database`: `SELECT Name, SerializedScript FROM Tasks WHERE Name LIKE '%keyword%'`
2. Parse the JSON array — each element has `_actionId` and parameters
3. `search` for the action names to understand what each step does

**Upcoming schedule + targets:**
1. `UpcomingJobRuns` with `IsNextScheduledRun = 1` → JOIN `JobRunRecords` → `Jobs`
2. Direct clients: `Jobs` → `ClientJobs` → `Clients`
3. Via groups: `Jobs` → `JobGroupBases` → `GroupBases` → `ClientGroupBases` → `Clients`

**Follow the activity chain** for run investigation:
Job → JobRunRecord → ClientJobRunRecord → ClientJobRunAttempt → ClientJobRunAttemptLog (each level adds detail)
