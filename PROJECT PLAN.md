# Context Engineering Plan — Spring Boot + Angular 19 + Oracle on OpenCode

## 1. What this project is

A parent repository with three subfolders — a Spring Boot backend, an Angular 19
frontend, and an Oracle database layer — each driven by its own **OpenCode agent**.
Every agent has persistent, file-based memory so it doesn't need to be
re-briefed at the start of every session.

```
parent-project/
├── AGENTS.md
├── opencode.json
├── .opencode/agent/
│   ├── backend.md
│   ├── frontend.md
│   └── database.md
├── shared-context/
│   ├── memory.md
│   ├── plan.md
│   └── state.md
│
├── spring-boot-backend/
│   ├── AGENTS.md
│   ├── .context/
│   │   ├── memory.md
│   │   ├── plan.md
│   │   └── state.md
│   └── src/...
│
├── angular-frontend/
│   ├── AGENTS.md
│   ├── .context/
│   │   ├── memory.md
│   │   ├── plan.md
│   │   └── state.md
│   └── src/...
│
└── oracle-database/
    ├── AGENTS.md
    ├── .context/
    │   ├── memory.md
    │   ├── plan.md
    │   └── state.md
    └── sql/...
```

## 2. Why context engineering, in plain terms

An AI agent's chat window is like a whiteboard: useful while you're standing
in front of it, wiped clean the moment you walk away. Context engineering
means writing the important parts down in files instead of leaving them on
the whiteboard, split into three purposes:

| File | Answers | Changes |
|---|---|---|
| `memory.md` | "What have we learned and decided, standing?" | Rarely |
| `plan.md` | "What are we working toward right now?" | When priorities shift |
| `state.md` | "Exactly where did we leave off?" | Every session |

A small `shared-context/` folder at the parent level holds the same three
files for decisions that cross module boundaries — e.g. a new API field that
both the backend and frontend need to know about.

## 3. Why / when / how

**Why use this pattern**
- Removes the need to re-explain architecture and conventions every session
- Keeps a written, reviewable trail of decisions (good for onboarding and audits)
- Lets three specialist agents work without stepping on each other's domain

**When it's worth it**
- Projects spanning more than one module and more than one working session
- Teams where more than one person (or agent) touches the same codebase
- Not worth the overhead for a single-file script or a one-off prototype

**How it stays healthy**
- Each agent reads its own three files, plus the shared ones, before working
- Each agent updates its own `state.md` before a session ends
- Anything cross-cutting gets promoted to `shared-context/`
- Files are reviewed and pruned weekly so stale notes don't mislead an agent

## 4. The session loop

1. **Read context** — agent loads its `AGENTS.md`, `memory.md`, `plan.md`, `state.md`, and the shared-context equivalents.
2. **Do the work** — scoped to that agent's own folder only.
3. **Update `state.md`** — what's done, what's next, what's blocked.
4. **Promote to shared** — anything the other two agents need lands in `shared-context/`.

## 5. Implementation steps

1. Install OpenCode: `curl -fsSL https://opencode.ai/install | bash`
2. Create the parent folder and the three subfolders (move existing code in as-is).
3. Generate/write the root `AGENTS.md` (the `/init` command inside `opencode` can draft one).
4. Register the three agents in `opencode.json`.
5. Add a `.context/` folder with `memory.md`, `plan.md`, `state.md` to each subfolder.
6. Add the root `shared-context/` folder with the same three files.
7. Start a session, switch to the relevant agent (Tab key), and confirm it reads its context files.
8. End every session by updating `state.md` — and `shared-context/` if the change is cross-cutting.
9. Commit `AGENTS.md`, `.context/`, and `shared-context/` to git like any other documentation.
10. Review and prune `memory.md` files weekly.

## 6. Starter templates

### `spring-boot-backend/.context/memory.md`
```markdown
# Backend Memory

## Standing decisions
- All REST endpoints are versioned under /api/v1
- DTOs are separate from JPA entities, always
- Errors return a consistent { code, message } shape

## Conventions
- Package by feature, not by layer
- Every entity maps to one Oracle table owned by the database agent

## Lessons learned
- (add as they happen — one line each, dated)
```

### `spring-boot-backend/.context/plan.md`
```markdown
# Backend Plan — Sprint of 2026-07-14

## Goal
Ship the order-management endpoints end to end.

## In scope
- [ ] POST /api/v1/orders
- [ ] GET /api/v1/orders/{id}
- [ ] Validation + error handling

## Depends on
- oracle-database: `orders` table migration
```

### `spring-boot-backend/.context/state.md`
```markdown
# Backend State — last updated 2026-07-16

## Done
- POST /api/v1/orders implemented + unit tested

## In progress
- GET /api/v1/orders/{id} — controller written, service pending

## Next session should
- Finish the service layer, then wire up validation
```

### `spring-boot-backend/AGENTS.md`
```markdown
# Spring Boot Backend — Agent Instructions

You are the backend specialist for this repository. Work only inside
`spring-boot-backend/` unless explicitly asked to cross into another module.

## Before doing anything
1. Read `.context/memory.md`, `.context/plan.md`, `.context/state.md`
2. Read `../shared-context/memory.md` and `../shared-context/state.md`

## Stack
Java 21, Spring Boot 3.x, Maven, JUnit 5, Oracle JDBC

## After finishing work
Update `.context/state.md`. If the change affects the frontend or
database schema, add a note to `../shared-context/state.md`.
```

### `parent-project/opencode.json`
```json
{
  "$schema": "https://opencode.ai/config.json",
  "default_agent": "backend",
  "agent": {
    "backend": { "description": "Spring Boot specialist — spring-boot-backend/ only" },
    "frontend": { "description": "Angular 19 specialist — angular-frontend/ only" },
    "database": { "description": "Oracle SQL specialist — oracle-database/ only" }
  }
}
```

> Mirror the same `memory.md` / `plan.md` / `state.md` / `AGENTS.md` pattern for
> `angular-frontend/` (Angular 19, standalone components, signals, styling
> conventions) and `oracle-database/` (schema naming, indexing, migration
> conventions), swapping in the stack details for each.

## 7. Setup checklist

**Foundations**
- [ ] Install OpenCode and confirm it runs
- [ ] Create the parent-project folder with the three subfolders inside it
- [ ] Confirm existing code sits in its own subfolder, untouched

**Agents & config**
- [ ] Write the root `AGENTS.md`
- [ ] Create `opencode.json` registering the three agents
- [ ] Write a persona `AGENTS.md` inside each subfolder

**Context files**
- [ ] Add `.context/memory.md`, `plan.md`, `state.md` to each subfolder
- [ ] Create `shared-context/` with the same three files at the root
- [ ] Fill in each `memory.md` with real, current conventions

**Habits**
- [ ] Run one real session per agent and confirm it reads context unprompted
- [ ] End that session by updating `state.md`
- [ ] Commit `AGENTS.md`, `.context/`, `shared-context/` to git
- [ ] Set a recurring weekly reminder to prune stale memory entries
