# CLAUDE.md — Root (Project Constitution)

> **This file is the project's persistent memory and operating manual.**
> Every agent, in every session, reads this file first — before any other instruction.
> It is the single source of truth for project identity, architecture, and inter-agent protocol.

---

## 1. Project Identity

**Product:** Digital Twin SaaS  
**Purpose:** A model-based probabilistic digital twin platform that mirrors industrial/operational processes, enables scenario simulation, and delivers optimized operating decisions through a user-facing dashboard.  
**Stage:** Active development — multi-agent pipeline.

---

## 2. Repository Layout

```
/
├── CLAUDE.md                  ← You are here (Root Constitution)
├── README.md
├── context/                   ← All inter-agent artifacts live here (JSON only)
│   ├── schema.json            ← Owned by OR
│   ├── model_spec.json        ← Owned by OR
│   ├── policy.json            ← Owned by RL
│   └── session_log.json       ← Owned by PM
├── agents/
│   ├── pm/CLAUDE.md
│   ├── or/CLAUDE.md
│   ├── ml/CLAUDE.md
│   ├── rl/CLAUDE.md
│   └── ux/CLAUDE.md
├── sim_api.py                 ← Owned by ML
└── dashboard/                 ← Owned by UX
```

---

## 3. The Five Agents

| Agent | Role | Primary Artifacts |
|-------|------|-------------------|
| **PM** | Pipeline orchestration, human interface, blockers | `context/session_log.json` |
| **OR** | DB profiling, causal process graph, schema modelling | `context/schema.json`, `context/model_spec.json` |
| **ML** | Probabilistic model fitting, simulator implementation | `sim_api.py` |
| **RL** | Decision optimisation using simulator as Gym env | `context/policy.json` |
| **UX** | Interactive dashboard, uncertainty visualisations | `dashboard/` |

---

## 4. Inter-Agent Communication Protocol

### 4.1 JSON Artifacts Only

**All hand-offs between agents MUST be expressed as JSON files in the `context/` directory.**

- ❌ Prose summaries, natural language notes, or comments as hand-off mechanisms are **prohibited**.
- ✅ Every agent reads the relevant JSON artifact(s) at session start.
- ✅ Every agent writes back to its owned artifact(s) at session end.

### 4.2 Artifact Schema Versioning

Every JSON artifact must contain a top-level `"_meta"` key:

```json
{
  "_meta": {
    "version": "1.0.0",
    "owner": "agent_id",
    "last_updated": "ISO-8601 timestamp",
    "status": "draft | validated | locked"
  }
}
```

### 4.3 Status Lifecycle

```
draft → validated → locked
```

- `draft`: Work in progress, not safe to depend on.
- `validated`: Reviewed and signed off by the owning agent and PM.
- `locked`: Downstream agents may depend on this artifact. No edits without PM approval.

---

## 5. Pipeline Gates

The following gates are **hard blockers**. No agent may start a gated task without the prerequisite being met.

```
[OR] schema.json status == "validated"
        ↓
[ML] may begin model fitting
        ↓
[OR] reviews sim_api.py → signs off in session_log.json
        ↓
[RL] may begin policy optimisation
        ↓
[RL] policy.json contains ≥ 3 evaluated scenarios
        ↓
[UX] may begin dashboard development
```

Gate violations must be logged in `context/session_log.json` and escalated to PM.

---

## 6. Shared Conventions

### 6.1 Language & Runtime
- Python ≥ 3.10 for all simulation and ML code.
- Node.js ≥ 18 for dashboard.
- No agent may introduce a new language dependency without PM approval.

### 6.2 Performance Contract
- `sim_api.py` must respond in **< 500 ms** per call (p95). This is a hard SLA.

### 6.3 Data Boundaries
- **The RL agent must never connect to, read from, or write to the client production database.**
- All DB access is mediated exclusively by the OR agent, which exposes only schema and model specs.

### 6.4 Portability
- No proprietary LLM-specific tool-calling syntax anywhere in agent logic or prompts.
- All agent logic must be portable to any instruction-following model (e.g., Llama 3, Qwen).
- Use standard Markdown and JSON throughout.

---

## 7. Session Protocol

Every agent, at the start of every session:

1. Read this Root `CLAUDE.md`.
2. Read your agent-specific `CLAUDE.md` in `agents/<id>/CLAUDE.md`.
3. Read all `context/` JSON artifacts relevant to your role.
4. Check pipeline gate status before beginning work.
5. Log your session start in `context/session_log.json`.

At the end of every session:
1. Write updated artifacts to `context/`.
2. Log session end, decisions made, and any blockers in `context/session_log.json`.
3. Set artifact `status` field appropriately.

---

## 8. Escalation Path

All communication with the human goes through the **PM agent only**.  
Other agents surface blockers by writing to `context/session_log.json` with `"escalate": true`.

---

*End of Root Constitution.*
