# Digital Twin SaaS

A model-based probabilistic digital twin platform that mirrors industrial and operational processes, enables scenario simulation, and delivers optimised operating decisions through an interactive user dashboard.

---

## What This Project Does

This platform builds a **digital twin** of a client's operational system by:

1. Profiling the client's database schema and constructing a causal process graph.
2. Fitting a probabilistic machine learning model (Model-Based ML) that captures the system's behaviour and uncertainty.
3. Exposing the model as a fast simulation API (`sim_api.py`, < 500 ms response).
4. Training a reinforcement learning policy that uses the simulator to optimise operating decisions.
5. Delivering an interactive web dashboard where operators can explore scenarios, adjust controls, and receive uncertainty-aware recommendations.

---

## Architecture Overview

```
Client DB
    ↓
[OR Agent] → schema.json, model_spec.json
    ↓
[ML Agent] → sim_api.py  (< 500 ms SLA)
    ↓
[RL Agent] → policy.json (≥ 3 evaluated scenarios)
    ↓
[UX Agent] → dashboard/
    ↑
[PM Agent] — orchestrates pipeline, interfaces with human
```

All inter-agent communication happens exclusively through JSON artifacts in the `context/` directory. No agent uses prose for hand-offs.

---

## Repository Structure

```
/
├── CLAUDE.md                  ← Root project constitution (read first, every session)
├── README.md                  ← This file
├── context/                   ← Inter-agent JSON artifacts
│   ├── schema.json            ← DB schema + causal graph (OR)
│   ├── model_spec.json        ← Probabilistic model definition (OR)
│   ├── policy.json            ← RL policy + evaluated scenarios (RL)
│   └── session_log.json       ← Session history + pipeline gates (PM)
├── agents/
│   ├── pm/CLAUDE.md           ← PM agent instructions
│   ├── or/CLAUDE.md           ← OR agent instructions
│   ├── ml/CLAUDE.md           ← ML agent instructions
│   ├── rl/CLAUDE.md           ← RL agent instructions
│   └── ux/CLAUDE.md           ← UX agent instructions
├── sim_api.py                 ← Simulator API (ML)
└── dashboard/                 ← Interactive frontend (UX)
```

---

## The Five Agents

| Agent | Full Name | Core Deliverable |
|-------|-----------|-----------------|
| **PM** | Project Manager | Pipeline orchestration, human interface |
| **OR** | Operations & Optimisation Research | `schema.json`, `model_spec.json` |
| **ML** | Machine Learning | `sim_api.py` |
| **RL** | Reinforcement Learning | `policy.json` |
| **UX** | User Experience | `dashboard/` |

---

## Pipeline Gates

Work proceeds sequentially through three hard gates:

**Gate 1 — Schema Validated**
`schema.json` must have `status: "validated"` before ML begins model fitting.

**Gate 2 — Simulator Reviewed**
OR must sign off on `sim_api.py` (checking API contract, uncertainty output, and < 500 ms SLA) before RL begins policy optimisation.

**Gate 3 — Policy Scenarios Met**
`policy.json` must contain ≥ 3 evaluated scenarios with `status: "validated"` before UX begins dashboard development.

---

## Getting Started

### Prerequisites

- Python ≥ 3.10
- Node.js ≥ 18
- Git

### Setup

```bash
git clone <repo-url>
cd digital-twin-saas
pip install -r requirements.txt
npm install --prefix dashboard
```

### Running the Simulator

```python
from sim_api import simulate

result = simulate({
    "input_variable_1": 42.0,
    "input_variable_2": 0.85
})

print(result["predictions"])   # point estimates
print(result["uncertainty"])   # std devs / CI half-widths
print(result["runtime_ms"])    # should be < 500
```

### Launching the Dashboard

```bash
cd dashboard
npm run dev
```

---

## Key Design Principles

**Model-agnostic.** No proprietary LLM tool-calling syntax is used anywhere. Agent logic is portable to any instruction-following model (Llama 3, Qwen, etc.).

**Uncertainty-first.** Every prediction surface in this system — from `sim_api.py` to the dashboard — includes uncertainty estimates. Point predictions alone are never acceptable outputs.

**Data boundary.** The RL agent never touches the client database. All DB access is mediated by OR, which exposes only schema and model artifacts.

**JSON contracts.** Agents communicate exclusively through versioned JSON files in `context/`. Prose hand-offs are prohibited.

---

## Contributing

1. Read `CLAUDE.md` (root) and your agent's `CLAUDE.md` before every session.
2. Check pipeline gate status in `context/session_log.json`.
3. Write all inter-agent outputs to `context/` as JSON.
4. Log every session, decision, and blocker in `context/session_log.json`.
5. Surface blockers to the PM — never bypass the pipeline gates.

---

## License

[To be defined]
