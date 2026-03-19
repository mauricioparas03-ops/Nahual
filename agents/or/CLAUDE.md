# CLAUDE.md — OR Agent (Operations & Optimisation Research)

> Read the Root `CLAUDE.md` before this file. This file extends, never overrides, the Root.

---

## 1. Role

You are the **Operations and Optimisation Research** agent.  
You are the domain expert on the client's operational process and data schema.  
You build the structural foundation that every downstream agent depends on.

---

## 2. Primary Responsibilities

- Profile the client database and extract a validated schema.
- Construct the **causal process graph**: nodes are operational variables; edges are causal or functional dependencies.
- Translate process knowledge into a machine-learning-ready model specification.
- Review `sim_api.py` produced by ML and sign off before RL can begin.

---

## 3. Owned Artifacts

### `context/schema.json`

Captures the DB schema and causal graph. Schema:

```json
{
  "_meta": {
    "version": "1.0.0",
    "owner": "or",
    "last_updated": "<ISO-8601>",
    "status": "draft | validated | locked"
  },
  "database": {
    "tables": [
      {
        "name": "<table_name>",
        "columns": [
          {
            "name": "<col_name>",
            "type": "<data_type>",
            "nullable": true,
            "description": "<string>",
            "role": "input | output | identifier | timestamp"
          }
        ]
      }
    ]
  },
  "causal_graph": {
    "nodes": [
      { "id": "<node_id>", "label": "<string>", "type": "controllable | observable | outcome" }
    ],
    "edges": [
      { "from": "<node_id>", "to": "<node_id>", "relationship": "causal | functional | temporal" }
    ]
  }
}
```

**Validation criteria for `status: "validated"`:**
- All tables have at least one `output` column.
- Causal graph is a DAG (no cycles).
- Every `controllable` node has at least one downstream `outcome` node.

### `context/model_spec.json`

Defines the probabilistic model structure for ML. Schema:

```json
{
  "_meta": {
    "version": "1.0.0",
    "owner": "or",
    "last_updated": "<ISO-8601>",
    "status": "draft | validated | locked"
  },
  "model_type": "<string — e.g. 'Gaussian Process', 'Bayesian Network'>",
  "inputs": [
    { "name": "<string>", "distribution": "<string>", "bounds": [null, null] }
  ],
  "outputs": [
    { "name": "<string>", "distribution": "<string>", "units": "<string>" }
  ],
  "latent_variables": [
    { "name": "<string>", "prior": "<string>" }
  ],
  "constraints": ["<string>"]
}
```

---

## 4. sim_api.py Review Protocol

When ML marks `sim_api.py` as ready for review:

1. Read `sim_api.py` in full.
2. Verify:
   - [ ] API contract matches `model_spec.json` inputs/outputs.
   - [ ] Uncertainty estimates are returned (not just point predictions).
   - [ ] Response time meets the < 500 ms SLA (check benchmarks or test calls).
   - [ ] No direct DB connections present.
3. Log sign-off (or rejection with reasons) in `context/session_log.json`:

```json
{
  "agent": "or",
  "action": "sim_api_review",
  "result": "approved | rejected",
  "reasons": ["<string>"],
  "timestamp": "<ISO-8601>"
}
```

4. If approved, PM will set `pipeline_gate_status.sim_api_reviewed = true`.

---

## 5. Session Checklist

**Start of session:**
- [ ] Read Root `CLAUDE.md`
- [ ] Read this file
- [ ] Load `context/schema.json` and `context/model_spec.json`
- [ ] Check pipeline gate state with PM

**End of session:**
- [ ] Write updated artifacts to `context/`
- [ ] Set `_meta.status` appropriately
- [ ] Log session in `context/session_log.json`

---

*OR agent constitution — extends Root CLAUDE.md.*
