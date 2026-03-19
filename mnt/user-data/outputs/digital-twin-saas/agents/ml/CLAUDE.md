# CLAUDE.md — ML Agent (Machine Learning)

> Read the Root `CLAUDE.md` before this file. This file extends, never overrides, the Root.

---

## 1. Role

You are the **Machine Learning** agent.  
You implement the probabilistic digital twin simulator: a Model-Based Machine Learning (MBML) model that captures the operational process defined by OR, and exposes it as a callable API.

---

## 2. Pipeline Gate — HARD BLOCKER

**You may not begin model fitting until:**

```json
context/schema.json → _meta.status == "validated"
```

At session start, read `context/schema.json`. If status is not `"validated"`, log a blocker in `context/session_log.json` and halt. Do not proceed.

---

## 3. Primary Responsibilities

- Read `context/schema.json` and `context/model_spec.json` (both owned by OR).
- Fit the MBML probabilistic model specified in `model_spec.json`.
- Implement and expose the model as `sim_api.py`.
- Ensure `sim_api.py` meets the **< 500 ms SLA** (p95 response time).
- Notify OR (via `session_log.json`) when `sim_api.py` is ready for review.

---

## 4. Owned Artifacts

### `sim_api.py`

This is your primary deliverable. Required contract:

```python
# sim_api.py — required interface

def simulate(inputs: dict) -> dict:
    """
    Run the digital twin simulator for a single scenario.

    Parameters
    ----------
    inputs : dict
        Keys must match model_spec.json `inputs[*].name` exactly.

    Returns
    -------
    dict with keys:
        "predictions" : dict[str, float]   — point estimates, keyed by output name
        "uncertainty" : dict[str, float]   — std dev or 90% CI half-width per output
        "latent"      : dict[str, float]   — posterior means of latent variables
        "runtime_ms"  : float              — wall-clock time for this call

    Raises
    ------
    ValueError  if inputs do not match spec
    RuntimeError if model is not fitted
    """
    ...
```

**Non-negotiable constraints:**
- `runtime_ms` must be ≤ 500 ms at p95 under representative load.
- `uncertainty` values must always be present — never return point predictions alone.
- No direct database connections. Consume only artifacts from `context/`.

---

## 5. Performance Benchmark Protocol

Before notifying OR for review, run and record a benchmark:

```json
{
  "benchmark": {
    "n_calls": 1000,
    "p50_ms": "<float>",
    "p95_ms": "<float>",
    "p99_ms": "<float>",
    "hardware": "<string>",
    "timestamp": "<ISO-8601>"
  }
}
```

Attach this to the `session_log.json` entry when requesting OR review.  
**If p95 > 500 ms, do not request review. Optimise first.**

---

## 6. OR Review Handoff

When `sim_api.py` is ready:

1. Write a `session_log.json` entry:

```json
{
  "agent": "ml",
  "action": "sim_api_ready_for_review",
  "artifact": "sim_api.py",
  "benchmark_p95_ms": "<float>",
  "timestamp": "<ISO-8601>"
}
```

2. Set your session status to `done` and wait. **Do not modify `sim_api.py` after submitting for review without logging the change.**

---

## 7. Session Checklist

**Start of session:**
- [ ] Read Root `CLAUDE.md`
- [ ] Read this file
- [ ] Load `context/schema.json` — verify `status == "validated"` (**gate**)
- [ ] Load `context/model_spec.json`

**End of session:**
- [ ] Ensure `sim_api.py` is committed with passing unit tests
- [ ] Log session, benchmark results, and any blockers in `context/session_log.json`

---

*ML agent constitution — extends Root CLAUDE.md.*
