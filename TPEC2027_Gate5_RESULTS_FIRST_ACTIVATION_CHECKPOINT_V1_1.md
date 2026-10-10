# TPEC 2027 — Gate 5 Results-First Amendment V1.1 Activation Checkpoint

**Checkpoint ID:** `TPEC2027_Gate5_RESULTS_FIRST_ACTIVATION_CHECKPOINT_V1_1`  
**Date:** 2026-10-09  
**Amendment:** `TPEC2027_Gate5_RESULTS_FIRST_AMENDMENT_V1_1.md`  
**Amendment SHA-256:** `dd6c997988cce434a33340c5d6dbd11c30c65d4c37173fd23de61b03d510063f`  
**Checkpoint type:** three independent adversarial reviews before activation  
**Scientific execution performed by this checkpoint:** **ZERO solver runs, ZERO ML training runs**

---

# 1. Why V1 was not activated

The earlier `TPEC2027_Gate5_RESULTS_FIRST_AMENDMENT_V1` remained a draft and was not externally anchored as controlling authority.

A pre-activation integration audit found four blocking ambiguities:

1. **DEV10 training leakage:** the draft did not explicitly exclude all records from the 10 development-evaluation dates from Day-3 training and validation.
2. **Action-space mismatch:** the frozen teacher action includes EV service, SWRO, and BESS dispatch, while the draft ML/heuristic definitions omitted BESS from the proposal.
3. **Safety-interface ambiguity:** the draft named B1-ZR as a common shield/fallback but did not fully define local physical rejection, successor certification, and executable fallback semantics.
4. **Future-EV information ambiguity:** the draft did not settle whether future daily EV obligations/capacity envelopes are announced information.

V1.1 corrects all four in one integrated successor. V1 must remain non-authoritative.

---

# 2. Static identity and provenance checks

The reviewed V1.1 bytes have SHA-256:

```text
dd6c997988cce434a33340c5d6dbd11c30c65d4c37173fd23de61b03d510063f
```

The amendment binds its scientific semantics to the existing frozen implementation identities:

```text
teacher oracle SHA256 : 3a06bf6358b1e15f32a1ae645707f20afd0fdbcd0ff01cf277dd91badc2ab58c
B1-ZR SHA256          : 0d0c43651982fafd5f965b069cf0c6ae12391c5e9684666c4223de3517010df1
input SHA256          : 260e3f4ea97a3e06ea3f4de3191d5071997131f8e56e83a59ff7dd711ec829ad
fold-map SHA256       : a5034858b62562dadadd85e4d149db4a005ee45fec8ce7a0ce59dd6f6bc11724
highspy               : 1.15.1
```

The frozen teacher oracle's canonical first action contains:

```text
p1_kw
p34_kw
qro_m3h
battery_net_kw
battery_charge_kw
battery_discharge_kw
swro_on
firming_kw
curtail_kw
```

V1.1 now uses the matching full proposal action:

```text
p1_kw
p34_kw
qro_m3h
battery_net_kw
swro_on
```

with deterministic charge/discharge decoding.

The frozen state transition propagates SOC using battery charge/discharge, tank inventory using SWRO flow, and remaining EV energy using the executed EV powers. V1.1 explicitly binds the controller simulator to those transitions.

The frozen oracle describes future daily EV service as an **announced service target** and uses the per-hour EV-capacity schedule in its continuation formulation. V1.1 makes that information assumption explicit and applies it identically to heuristic, MPC, ML, and B1-ZR.

---

# 3. Zero-result static integration audit

The following conditions were checked against the V1.1 text before this checkpoint:

```text
PASS  amendment identity is V1.1
PASS  no accidental V1.1.1 strings
PASS  no old Tier/Category-A training nomenclature
PASS  DEV10 date-wide exclusion from ML training and validation
PASS  DEV10 excluded from normalization and RQ5 training subsets
PASS  canonical full action includes BESS
PASS  battery net sign and charge/discharge decoding are explicit
PASS  current-interval physical audit is explicit
PASS  firming-cap and residual-balance rejection are explicit
PASS  successor B1-ZR certificate is explicit
PASS  constructive fallback calls frozen B1.construct_backup
PASS  fallback first action uses battery_net_kw = 0
PASS  fallback physical failure is fail-closed
PASS  future EV target/cap information assumption is explicit
PASS  future realized renewable leakage is prohibited
PASS  no new non-flexible electrical base-load term was introduced
PASS  ML has four continuous action heads plus one SWRO ON/OFF logit
PASS  ML loss includes normalized continuous Huber + binary BCE
PASS  MPC extracts the full first action
PASS  primary MPC does not receive an embedded terminal-B1 advantage
PASS  Day-2 denominator guards are explicit
PASS  Day-3 ML-vs-MPC denominator guard is explicit
PASS  Q0 exclusion remains permanent
PASS  QUAL48 remains sealed
PASS  AC-power / keep-awake / power-event controls are explicit
PASS  mandatory zero-solve implementation preflight is explicit
```

No result-producing experiment was run during this audit.

---

# 4. Reviewer 1 — optimization and mathematical programming

## Challenge set

Reviewer 1 independently challenged:

- whether the PF reference is a valid lower bound on realized 24-hour firming;
- whether any forecast-world optimum is accidentally treated as a realized-world lower bound;
- whether weak lower bounds can create a false GO;
- whether MPC timeout incumbents are being called optimal;
- whether the common controller action matches the frozen physical model;
- whether BESS omission could make successor SOC invalid;
- whether current firming-cap violations could slip through a successor-only B1 check;
- whether the PF/MPC/ML aggregate ratios have zero-denominator failure modes.

## Findings

### A. PF lower-bound semantics — PASS

V1.1 uses the realized 24-hour trajectory only for the offline PF physical reference. Optimality proof is not required; a valid dual lower bound is sufficient.

A forecast-world optimization is not reused as a realized-world lower bound.

### B. STOP-only use of physical headroom — PASS

`H_phys^UB` is used only to support STOP.

A looser lower bound increases the apparent upper headroom and therefore makes STOP harder to trigger. It cannot manufacture GO.

All 10 predetermined episodes remain in the primary population, preventing solver-tractability selection.

### C. MPC timeout semantics — PASS

MPC calls are classified:

```text
OPTIMAL_OR_CERTIFIED
FEASIBLE_INCUMBENT
NO_FEASIBLE_INCUMBENT
INVALID
```

A timeout is not sufficient for action use. A usable timeout incumbent must pass physical residual checks and provide a valid complete first action.

### D. Full physical action — PASS

V1.1 includes the BESS proposal and uses the frozen sign convention:

```text
battery_net > 0  -> discharge
battery_net < 0  -> charge
```

This closes the V1 action-space mismatch and allows SOC to be propagated under the same frozen transition as the teacher.

### E. Current physical audit before B1 — PASS

The action is now checked for:

- EV caps;
- SWRO graph;
- battery power/SOC;
- tank/SOC successor bounds;
- firming cap;
- firming/curtailment residual structure;
- numerical tolerances.

Only then is successor B1-ZR evaluated.

This prevents an action that has a B1-certifiable successor but violates the current interval's physical balance/cap from being executed.

### F. Ratio guards — PASS

V1.1 now defines numerical guards for:

- heuristic-headroom denominators;
- PF capture denominator;
- regret denominator;
- direct ML-vs-MPC degradation denominator.

Undefined percentages become `N/A` rather than unstable numbers.

## Reviewer 1 decision

```text
PASS
```

No optimization/mathematical blocker remains before implementation.

---

# 5. Reviewer 2 — ML, control, and leakage

## Challenge set

Reviewer 2 independently challenged:

- whether DEV10 can leak into training through the development fold;
- whether normalization can leak DEV10 statistics;
- whether the ML action is actually comparable with MPC;
- whether SWRO discreteness is ignored;
- whether future realized information leaks into ML/MPC;
- whether the EV information structure is ambiguous;
- whether the safety filter changes across controller classes;
- whether the Day-3 prototype invites uncontrolled architecture search.

## Findings

### A. DEV10 leakage — PASS

V1.1 makes DEV10 a date-wide development-evaluation holdout.

Every record on a DEV10 date is excluded from:

```text
training
validation
normalization-statistics fitting
training-label threshold derivation
RQ5 training subsets
```

The ledger must explicitly verify:

```text
n_train_records_from_DEV10_dates = 0
n_validation_records_from_DEV10_dates = 0
```

Any nonzero count is a hard failure.

### B. Full comparable action space — PASS

Heuristic, MPC, and ML all propose:

```text
p1_kw
p34_kw
qro_m3h
battery_net_kw
swro_on
```

The ML architecture has four normalized continuous heads plus one binary SWRO logit.

### C. SWRO decoding — PASS

The binary ON/OFF state is explicit. The continuous flow is decoded through the same frozen feasible SWRO graph used by the physical model.

No new approximate SWRO physics is introduced.

### D. Causal information parity — PASS

The future renewable forecast is causal previous-day same-hour persistence.

The daily aggregate EV service target and per-hour EV capacity envelope are explicitly treated as announced contractual information, matching the frozen model's announced-target semantics.

Future realized renewable generation is prohibited from heuristic, MPC, and ML inputs.

### E. Common safety architecture — PASS

All three controller types use the same:

```text
local decoder
current physical audit
successor B1-ZR certificate
constructive fallback
```

The only primary experimental difference is the proposal mechanism.

### F. Bounded ML adaptation — PASS

The Day-3 prototype has:

```text
fixed 6 x 10 GELU trunk
fixed action heads
one seed
one training run
no pre-Day-3 hyperparameter sweep
one allowed post-result modification only
```

This is sufficiently bounded for a results-first prototype.

## Reviewer 2 decision

```text
PASS
```

No ML/control/leakage blocker remains before implementation.

---

# 6. Reviewer 3 — adversarial IEEE reviewer and reproducibility

## Challenge set

Reviewer 3 independently challenged:

- whether the heuristic is a strawman;
- whether MPC is a fair deployed comparator;
- whether B1-ZR rejection is misrepresented as infeasibility;
- whether runtime conclusions can be driven by the chosen time limit;
- whether weak PF bounds are silently dropped;
- whether environmental sleep events can contaminate runtime;
- whether C/D solver work can again become an open-ended branch;
- whether the workflow produces real paper-level results quickly.

## Findings

### A. Baseline credibility — PASS

The heuristic includes:

- least-laxity / deadline-aware EV service;
- tank-aware SWRO scheduling;
- deterministic BESS residual dispatch;
- the same safety interface as ML and MPC.

It is a credible classical baseline rather than a deliberately weak comparator.

### B. Online MPC fairness — PASS

MPC is evaluated as a deployed causal controller, not merely as an offline optimum.

It receives the same safety layer as heuristic and ML.

Its runtime reporting includes:

- time-limit binding fraction;
- status fractions;
- all-call latency distribution;
- non-binding latency distribution;
- time-limited feasible-incumbent gaps.

This prevents a single mean runtime from hiding censoring.

### C. B1-ZR semantics — PASS

V1.1 separates:

```text
LOCAL_PHYSICAL_REJECT
B1_ZR_REJECT
total intervention
```

B1-ZR rejection remains “not certified,” not “proven infeasible.”

A tri-state rejection audit remains mandatory.

### D. Weak-bound population handling — PASS

No predetermined DEV10 episode is removed because its PF bound is weak.

Weak bounds remain conservative because physical headroom is used only to support STOP, not GO.

### E. Environmental contamination — PASS

Long result-producing runs require:

```text
ACLineStatus = 1
keep-awake active
post-run sleep/resume/power audit
```

An externally interrupted run is retained as `ENVIRONMENTAL_INVALID`; exactly one identical environment-only replay is permitted, and no model/solver setting may change.

### F. Terminating branches — PASS

V1.1 prohibits:

- finishing C9 merely for process completion;
- full D1–D9 as blocking work;
- repeated MPC tuning;
- repeated ML hyperparameter search;
- replacing unfavorable DEV10 episodes;
- dropping weak PF-bound episodes;
- changing thresholds after results.

The limited D diagnostic is exactly three predeclared cases and non-blocking.

### G. Tangible-result schedule — PASS

The first mandatory scientific result is the Day-2 table:

```text
heuristic
causal MPC
PF lower bound
physical headroom
MPC achieved improvement
runtime censoring
certificate behavior
```

The second is the Day-3 heuristic-vs-MPC-vs-ML closed-loop table.

The protocol therefore directly addresses the failure mode that caused the previous month of activity without paper-level outcomes.

## Reviewer 3 decision

```text
PASS
```

No adversarial-review blocker remains before implementation.

---

# 7. Overall activation checkpoint

Reviewer decisions:

```text
Reviewer 1 — Optimization / mathematical programming : PASS
Reviewer 2 — ML / control / leakage                  : PASS
Reviewer 3 — IEEE / reproducibility                   : PASS
```

## Checkpoint decision

```text
PASS TO EXTERNAL ANCHOR
```

The amendment is scientifically approved for activation **only if the externally anchored bytes have exactly this SHA-256**:

```text
dd6c997988cce434a33340c5d6dbd11c30c65d4c37173fd23de61b03d510063f
```

No later edit may retain the V1.1 identifier or this checkpoint.

If the amendment bytes change, this checkpoint is void and must be regenerated.

---

# 8. Authorization after exact-byte external anchor

After both the exact V1.1 amendment and this activation checkpoint are externally anchored:

```text
C9 UNDER OLD PROTOCOL                 = CANCELLED
FULL D1–D9 AS BLOCKING WORK           = CANCELLED
QUAL48                                = SEALED
PRODUCTION                            = NOT AUTHORIZED
FINAL ML TRAINING                     = NOT AUTHORIZED

DAY-0 RESULTS-FIRST IMPLEMENTATION    = AUTHORIZED
ZERO-SOLVE IMPLEMENTATION PREFLIGHT   = REQUIRED
DEV10 SELECTION/HASH                  = AUTHORIZED
DEV10 RESULT-PRODUCING RUNS           = BLOCKED UNTIL PREFLIGHT PASS
LIMITED D DIAGNOSTIC                  = AUTHORIZED, NON-BLOCKING
```

After the zero-solve implementation preflight passes, the first result-producing execution is:

> **the strong heuristic + causal MPC + perfect-foresight T1 reference on the same 10 frozen development-evaluation episodes.**

---

# 9. Immediate next action

1. externally anchor `TPEC2027_Gate5_RESULTS_FIRST_AMENDMENT_V1_1.md` at SHA-256 `dd6c997988cce434a33340c5d6dbd11c30c65d4c37173fd23de61b03d510063f`;
2. externally anchor this checkpoint;
3. record repository commit(s), parent commit, UTC timestamp(s), and exact remote-byte verification;
4. only then begin Day-0 implementation and DEV10 selection;
5. perform zero optimization solves and zero ML training until the mandatory implementation preflight passes.
