# agent-ledger-sim

Simulator, analysis code, and proofs for

> **A Fault-Tolerant Position Ledger for Agentic AI Trading Systems Sharing
> a Brokerage Account**
> Ramakant Yadav (Scalar Field) and Neha Prasad
> Submitted to the 2027 IEEE International Conference on AI Engineering and
> Innovation (AIEI 2027).

The simulator is a discrete-event model of `N` AI trading agents that share
one brokerage account. The platform keeps a per-agent ledger that must stay
consistent with the broker's aggregate holdings under partial fills,
lost or delayed fill reports, API timeouts, client retries, agent crashes,
and out-of-band trades by the account owner. Faults are injected, and the
simulator measures encroaching fills, unexplained drift, detection
latency, availability, and polling overhead for four ledger policies.

`PROOFS.md` contains the full case analyses for Propositions 1–3 and the
observation-floor derivation used in Section V-C of the paper. It uses the
code's internal names; a terminology table at the top maps them to the
paper's terms.

## Ledger policies

| Policy | Description |
|--------|-------------|
| P0 | No reconciliation. Trusts the local ledger and books unresolved orders optimistically. Upper bound on exposure only. |
| P1 | Per-order check. Polls positions before every order and rejects orders on instruments with unresolved shortfalls. |
| P2(tau) | Periodic. Polls every `tau` seconds and rebases after an investigation delay, but never halts trading. |
| P3(tau) | Proposed design. P2's detection plus per-agent claims, freezes on shortfalls, and the pre-trade backing check. |

All policies use idempotent target-position execution unless configured
otherwise (`retry_mode="delta"` reproduces naive delta retries for the
ablation in Table II).

## Requirements

Python 3.10 or newer. The simulator (`sim/model.py`, `sim/run.py`,
`sim/smoke.py`) uses only the standard library. Plotting (`sim/plots.py`)
needs `numpy` and `matplotlib`:

```bash
python3 -m venv .venv && .venv/bin/pip install -r requirements.txt
```

## Reproducing the paper

```bash
# About 7,800 simulation runs; a few minutes on a laptop using all cores.
.venv/bin/python sim/run.py            # writes results/results.csv

# Figures 2-4 and every number quoted in Sections V-VI.
.venv/bin/python sim/plots.py          # writes figures/*.pdf, results/summary.txt

# Quick sanity check (seconds).
.venv/bin/python sim/smoke.py
```

`sim/run.py` covers the fault-intensity sweep, the interval sweep for P2
and P3, the staleness sweep, the fleet-size sweep, the retry and
enforcement-point ablations, the bursty-fault run, the per-parameter
robustness sweeps, and the replay of two production agent cohorts whose
anonymized wake-time offsets are in `sim/traces/wake_times.csv`.

Results are means over seeds with normal-approximation 95% confidence
intervals (100 seeds per cell; 15 for the scaling and robustness sweeps).

## Layout

```
sim/model.py     broker, ledger, agents, fault injection, policies P0-P3
sim/run.py       experiment grid; writes results/results.csv
sim/plots.py     figures and the numeric summary
sim/smoke.py     fast consistency checks
sim/traces/      anonymized production wake-time offsets for the replay
PROOFS.md        full proofs of Propositions 1-3 and the observation floor
```

## License

MIT. See `LICENSE`.
