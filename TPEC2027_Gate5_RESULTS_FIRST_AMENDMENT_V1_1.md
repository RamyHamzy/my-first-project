# TPEC 2027 — Gate 5 Results-First Amendment V1.1

**Document ID:** `TPEC2027_Gate5_RESULTS_FIRST_AMENDMENT_V1_1`  
**Date:** 2026-10-09  
**Status:** **INTEGRATED SUCCESSOR, READY FOR THREE-REVIEWER ACTIVATION CHECKPOINT**  
**Scope:** Gate 5 implementation and reproducibility for the TPEC 2027 ML critical-load EMS paper  
**Paper direction:** ML remains the core contribution. Reduced-interface and exact-optimization work remain supporting baselines, references, and safety evidence.

---

# 0. Purpose of this amendment

This amendment changes the **sequence of work**, not the frozen physical science.

The previous workflow required increasingly complete optimizer certification before any ML prototype was allowed. That sequence produced a rigorous audit trail but delayed the experiment that determines whether the paper's ML contribution is scientifically useful.

This amendment adopts a **results-first, decision-driven development protocol**:

> **freeze provenance → construct a verified development ledger → select 10 development episodes before seeing controller outcomes → build a strong heuristic and a causal MPC baseline → compute perfect-foresight lower bounds → quantify whether meaningful performance headroom exists → train one simple ML prototype → evaluate it closed-loop under the same safety layer → make an explicit GO / MODIFY / STOP decision.**

The amendment preserves the existing requirements for:

- physical feasibility;
- source and data provenance;
- reproducibility;
- outcome-blind data partitioning;
- untouched final evaluation data;
- explicit solver status;
- no reinterpretation of timed-out incumbents as optima;
- no post-hoc threshold selection;
- three-reviewer gate checks.

It removes only the requirement that every development label or every hard MILP case be globally certified before ML development begins.

---

# 1. Authority and supersession

## 1.1 What this amendment supersedes

`TPEC2027_Gate5_RESULTS_FIRST_AMENDMENT_V1` was a draft only. It was **never externally anchored or activated**. A pre-activation integration audit found four blocking ambiguities: DEV10 training leakage, incomplete controller action space, an under-specified B1-ZR execution interface, and unresolved future-EV information semantics. V1.1 corrects all four in one integrated successor.

Upon external anchoring and activation, this amendment supersedes the **development sequencing requirement** that:

- Strategy C must be completed through C9 before ML development;
- Strategy D must be run on all nine forensic hard cases before ML development;
- DEV24 / QUAL48 solver qualification must be completed before any ML prototype;
- a production-complete globally certified teacher-label corpus is required before obtaining the first ML result.

Specifically:

> **C9 shall not be run under the superseded Stage-1 protocol.**

> **A full D1–D9 campaign shall not be run as a blocking prerequisite.**

Strategy D is retained only as the limited diagnostic defined in §18.

## 1.2 What this amendment does not supersede

The amendment does **not** alter:

- the frozen physical EV–SWRO–BESS–renewable model;
- the T1 firming-energy objective;
- the physical feasibility tolerances;
- the frozen solver-status taxonomy;
- the rule that timeout incumbents are not optima;
- the frozen GAP-OR admissibility semantics for runs that are claimed as certified;
- B1-ZR's sufficient-certificate semantics;
- the frozen source hierarchy;
- the frozen data/fold provenance;
- the requirement for an untouched final test population;
- Gate 6 through Gate 11;
- the requirement for a three-reviewer checkpoint before advancing a formal gate.

## 1.3 Activation condition

This amendment becomes operative only after:

1. the exact Markdown bytes are SHA-256 hashed;
2. the file is committed to the controlling repository/branch;
3. the commit or equivalent external anchor is recorded;
4. the three-reviewer activation checkpoint in §24 is signed PASS.

Until those four conditions are met:

```text
PRODUCTION AUTHORIZED = FALSE
FINAL ML TRAINING AUTHORIZED = FALSE
RESULTS-FIRST PROTOTYPE EXECUTION = NOT YET ACTIVE
```

After activation, only **development-prototype ML** is authorized under this document. Production/final-training authorization remains separate.

---

# 2. Frozen status at amendment entry

The amendment starts from the following immutable development record.

## 2.1 Strategy B

```text
Strategy: T1_B_BASELINE_3600
Completed: 9/9
Admissible: 1
Unresolved: 8
Orphaned: 0
Eligible 9/9: FALSE
```

B is permanently ineligible under the superseded Stage-1 winner rule.

## 2.2 Strategy C

At amendment entry:

```text
Strategy: T1_C_PROOF_FOCUS_1800
Completed: 8/9
Admissible: 1
Unresolved: 7
Orphaned: 0
C9: NOT RUN
Eligible 9/9: FALSE
```

The only C case certified to date is record 41, which was also the only B-certified hard case. Therefore the demonstrated certifiable-set expansion from C is zero.

C9 is intentionally terminated by this amendment because it cannot change the scientific decision that C is ineligible.

## 2.3 Strategy D

```text
Strategy: T1_D_PROOF_FOCUS_3600
Completed: 0/9
Role after amendment: LIMITED NON-BLOCKING DIAGNOSTIC ONLY
```

## 2.4 Qualification and production populations

```text
QUAL48 accessed: FALSE
Production authorized: FALSE
Final ML training authorized: FALSE
```

QUAL48 remains sealed and is not repurposed as the 10-episode ML development set.

---

# 3. Paper objective under the amended protocol

The paper investigates whether a learned controller can schedule **deadline-constrained EV charging** and **inventory-constrained SWRO desalination** under shared renewable, battery, and firming constraints while preserving joint feasibility and reducing online decision cost.

The paper's ML claim is not:

> “A neural network can imitate optimizer labels.”

The intended claim is:

> **A learned proposal policy, combined with the same certified safety/fallback mechanism used by the comparators, can approach the operational quality of causal optimization with substantially lower and more predictable online computation, while maintaining auditable closed-loop feasibility behavior.**

The most important scientific questions become:

### RQ1 — Closed-loop feasibility
Can the learned proposal mechanism complete sequential operation while meeting EV, tank, BESS, and shared-resource requirements?

### RQ2 — Firming quality
How much firming reduction does ML achieve relative to:
- a strong deterministic heuristic;
- causal online MPC;
- a perfect-foresight physical lower bound?

### RQ3 — Online computation
How do ML, heuristic, and MPC compare in deployed decision time, including safety-filter time and MPC time-limit censoring?

### RQ4 — Safety-layer dependence
How often does B1-ZR reject each controller's proposals, how conservative are those rejections, and is the safety layer materially more important for ML than for optimization-based proposals?

### RQ5 — Teacher-label quality
How sensitive is learned-policy performance to the T1 bound quality of the development labels when sample count is controlled separately?

---

# 4. Canonical controller action and common safety architecture

The frozen teacher's first action contains EV service, SWRO production, and BESS dispatch. The amended controller comparison therefore uses the **same full proposal action space** for heuristic, MPC, and ML.

## 4.1 Canonical proposal action

Every proposal is represented as:

```text
p1_kw
p34_kw
qro_m3h
battery_net_kw
swro_on
```

with sign convention:

\[
P_{\mathrm{batt,net}}>0
\quad\Longleftrightarrow\quad
\text{battery discharge},
\]

\[
P_{\mathrm{batt,net}}<0
\quad\Longleftrightarrow\quad
\text{battery charge}.
\]

The executable charge/discharge pair is deterministically decoded as

\[
P_{\mathrm{dis}}=\max(P_{\mathrm{batt,net}},0),
\qquad
P_{\mathrm{ch}}=\max(-P_{\mathrm{batt,net}},0).
\]

Thus simultaneous charging and discharging is impossible by construction.

The frozen action identity is consistent with the authenticated teacher oracle, whose first action includes `p1_kw`, `p34_kw`, `qro_m3h`, `battery_net_kw`, `battery_charge_kw`, `battery_discharge_kw`, `swro_on`, `firming_kw`, and `curtail_kw`.

## 4.2 Deterministic local decoder

Before any continuation certificate is evaluated, the proposal is decoded with the same deterministic local rules for all three controllers:

1. clip `p1_kw` and `p34_kw` to the frozen current-interval network/availability caps;
2. decode `swro_on`;
3. if `swro_on = 0`, set `qro_m3h = 0`;
4. if `swro_on = 1`, clip `qro_m3h` to the frozen feasible ON-flow interval represented by the SWRO piecewise-linear graph;
5. clip `battery_net_kw` to the intersection of:
   - the frozen BESS power limit; and
   - the one-step SOC-feasible charging/discharging interval;
6. derive `battery_charge_kw` and `battery_discharge_kw` from the signed net value;
7. obtain SWRO electrical power from the **frozen SWRO graph**, not from a new fitted equation;
8. derive firming and curtailment from the frozen residual power balance using the **realized current** renewable output.

This decoder is not described as a feasibility projection. It only converts a proposal into the canonical one-step action representation and enforces local variable-domain physics that are common to all controllers.

If an implementation-specific local constraint exists in the frozen physical model, it must be enforced here identically for all controllers. No new ramp, switching, or efficiency rule may be introduced unless it already exists in the frozen physical formulation.

## 4.3 Current-interval physical audit and successor certificate

After decoding, every controller proposal passes the **same two-stage acceptance test**.

### Stage A — current-interval physical audit

Using the realized current renewable output, verify:

- EV power is within the frozen current network caps;
- SWRO ON/OFF and piecewise-linear power/flow relations are valid;
- decoded battery charge/discharge respects the frozen BESS power and SOC bounds;
- no simultaneous battery charge/discharge occurs;
- the one-step tank state is within frozen bounds;
- the one-step SOC state is within frozen bounds;
- residual firming is nonnegative and does not exceed the frozen firming-cap variable bound;
- curtailment and firming do not coexist beyond the frozen numerical tolerance;
- all checked residuals satisfy the frozen physical tolerance.

Failure is classified:

```text
LOCAL_PHYSICAL_REJECT
```

and triggers the common fallback.

### Stage B — successor B1-ZR certificate

If Stage A passes, compute the one-hour successor with the frozen state transition:

- SOC from decoded battery charge/discharge and frozen efficiencies;
- tank inventory from `qro_m3h` and frozen water demand;
- remaining EV energy from executed `p1_kw` and `p34_kw`;
- at the model day boundary, apply the frozen daily EV reset/announced-target rule.

The **external frozen B1-ZR predicate** is the acceptance authority on that successor state.

Failure is classified:

```text
B1_ZR_REJECT
```

and triggers the common fallback.

Theorem-A may be recorded as a diagnostic necessary-infeasibility screen, but it does not create a controller-specific safety regime.

The reporting distinction is mandatory:

```text
raw_current_action_failure = LOCAL_PHYSICAL_REJECT
certificate_rejection      = B1_ZR_REJECT after Stage-A pass
total_intervention          = either rejection type
```

A B1-ZR rejection is not interpreted as proof of infeasibility.

## 4.4 Exact constructive fallback

If either Stage A or Stage B rejects the proposal:

1. invoke the frozen `B1.construct_backup(...)` routine from the **current pre-action state** over the current model-day suffix;
2. if no backup schedule is returned, classify the event as:
   ```text
   B1_ZR_BACKUP_CONSTRUCTION_FAILED
   ```
   and stop the episode as a Gate-5 implementation failure;
3. otherwise take the **first row** of the returned backup schedule:
   - `p1_kw` from the backup;
   - `p34_kw` from the backup;
   - `qro_m3h` / water action from the backup;
   - `battery_net_kw = 0`;
4. derive SWRO power, firming, and curtailment from the frozen physical equations and realized current renewable output;
5. run the same current-interval physical audit on the fallback action.

If the fallback action fails the physical audit, classify:

```text
B1_ZR_FALLBACK_PHYSICAL_FAILURE
```

and stop the episode as a Gate-5 implementation failure.

This is the common constructive fallback for heuristic, MPC, and ML.

## 4.5 Primary deployed comparison

```text
HEURISTIC full proposal
        ↓
common local decoder
        ↓
current physical audit
        ↓
successor B1-ZR certificate
        ├─ accept → execute proposal
        └─ any reject → common constructive fallback

CAUSAL MPC full proposal
        ↓
common local decoder
        ↓
current physical audit
        ↓
successor B1-ZR certificate
        ├─ accept → execute proposal
        └─ any reject → common constructive fallback

ML full proposal
        ↓
common local decoder
        ↓
current physical audit
        ↓
successor B1-ZR certificate
        ├─ accept → execute proposal
        └─ any reject → common constructive fallback
```

The primary quantities

```text
z_heur
z_MPC
z_ML
```

mean realized T1 firming energy **after** this identical decoder + B1-ZR + fallback interface.

A separate unfiltered-MPC experiment is allowed only as the secondary RQ4 diagnostic in §21.

---

# 5. B1-ZR semantics

B1-ZR is a **sufficient** continuation certificate.

Therefore:

> **B1-ZR rejection does not prove that a proposed action is infeasible.**

It proves only that the sufficient certificate could not vouch for the action.

Define:

\[
r_{\mathrm{reject}}
=
\frac{N_{\mathrm{B1ZR\ reject}}}{N_{\mathrm{decisions}}}.
\]

This is reported as:

> **certificate-rejection rate / upper bound on continuation-certificate failure among locally physical proposals**

and never as the measured ML infeasibility rate.

Report `LOCAL_PHYSICAL_REJECT` separately. The total fallback/intervention rate includes both local physical rejection and B1-ZR rejection.

If either common acceptance stage rejects a proposal, the same frozen constructive B1-ZR fallback is used for heuristic, MPC, and ML.

If that fallback unexpectedly fails, the episode is a hard implementation failure and Gate 5 stops.

---

# 6. Development/test separation

## 6.1 Q0

The Q0 boundary is closed as follows:

> **Q0 records 0–7 are permanently excluded from ML training, validation, qualification, and final testing.**

They may be retained only for:

- forensic evidence;
- provenance reconstruction;
- regression testing that does not produce a scientific result.

## 6.2 QUAL48

QUAL48 remains sealed.

It is not used for:

- selecting the 10 development episodes;
- tuning the heuristic;
- tuning MPC;
- training the MLP;
- selecting label-quality thresholds;
- Day-2 or Day-3 GO/MODIFY/STOP decisions.

## 6.3 Ten prototype episodes

The 10 prototype episodes are **development-evaluation holdout data permanently**.

Once selected, they may be used for:

- heuristic/MPC/ML closed-loop evaluation;
- controller debugging that does not change their membership;
- the bounded architecture/decision changes explicitly allowed by this amendment;
- GO/MODIFY/STOP decisions.

They are **not eligible for Day-3 ML training or validation**. Every record whose model date is one of the 10 DEV10 dates is excluded from:

- the ML training pool;
- the ML validation pool;
- normalization/statistics fitting;
- training-label threshold derivation;
- RQ5 training subsets.

This exclusion applies to **all states and all historical labels on those dates**, not only to the particular initial state used in the closed-loop episode.

They are permanently excluded from the final untouched test population.

## 6.4 Final test population

The final ML test population must be drawn from the already frozen holdout/test side of the original fold structure and must exclude:

- Q0;
- all 10 prototype episodes;
- any episode used in heuristic/MPC/ML modification;
- QUAL48 unless a later separately anchored amendment explicitly assigns it a new role;
- any episode exposed during threshold selection.

---

# 7. Historical V5/V6 artifact-use rules

Historical artifacts are assigned to one of four provenance classes. These names are deliberately distinct from the abandoned label-quality “Tier A/B/C/D” language.

## `TRAINING_ELIGIBLE`

A historical label may enter a development training pool only if all of the following hold:

- a complete executable **full canonical action** is present or can be losslessly reconstructed from authenticated raw solution data;
- physical feasibility checks pass;
- provenance binds the action to the frozen input/state/model;
- there is no corruption or orphaned lifecycle;
- the record is not Q0;
- the record date is not one of the 10 DEV10 episode dates;
- the record is not part of a protected final/qualification population;
- any T1 uncertainty is retained as continuous metadata;
- any T2/T3 ambiguity is retained explicitly.

## `FORENSIC_ONLY`

Examples:

- runs interrupted by environmental sleep/power events;
- T1-only forensic results without a complete executable teacher action;
- solver experiments whose runtime is not comparable to the amended protocol;
- obsolete qualification attempts retained for diagnosis.

These may not be training labels.

## `PERMANENTLY_EXCLUDED`

Examples:

- Q0 records 0–7;
- invalid/corrupt records;
- orphaned starts without trustworthy completion;
- records whose action/model binding cannot be authenticated.

## `FINAL_EVAL_ELIGIBLE`

**No V5 production artifact is automatically `FINAL_EVAL_ELIGIBLE`.**

Any historical artifact exposed during development is excluded from final ML evaluation.

## Runtime rule

V5 solver runtimes are not used in the amended RQ3 runtime comparison.

---

# 8. Development-label semantics

Categorical Tier A/B/C/D terminology is abandoned as the primary scientific representation.

Each usable development label stores continuous metadata.

At minimum:

```text
label_id
source_run
state_id
episode/date
teacher_action
target_p1_kw
target_p34_kw
target_qro_m3h
target_battery_net_kw
target_swro_on
physical_feasibility_status

T1_primal
T1_dual_bound
T1_relative_gap
T1_absolute_gap
T1_native_status

T1_epsilon_ceiling_used_for_T2
T2_native_status
T2_objective

T3_native_status
T3_objective

environmental_interruption_flag
model_hash
input_hash
state_hash
```

A timed-out historical run does not become a training label merely because an incumbent existed. If an unresolved raw solution is considered for `TRAINING_ELIGIBLE`, the development-ledger builder must independently reconstruct its canonical first action from the authenticated solution vector and re-run the frozen physical residual checks. The historical production label record itself is not edited or reclassified.

For a feasible label with unresolved T1 optimality, the required wording is:

> **T1-bounded feasible teacher trajectory; T2/T3 global lexicographic optimality undefined.**

The document must never describe a complete three-stage teacher trajectory as “approximately optimal” solely because the T1 gap is small.

---

# 9. Selection of the 10 development episodes

The 10 episodes must be fixed **before** any of the following are inspected for those episodes:

- heuristic firming;
- PF lower bounds;
- MPC firming;
- MPC solve gaps;
- ML results;
- B1-ZR activation outcomes.

The selected 10 dates immediately become a **development-evaluation holdout** and are excluded from every training/validation pool as specified in §6.3.

## 9.1 Candidate-day eligibility

A date is eligible if:

1. all 24 hourly model intervals are present;
2. the entire 24-hour episode belongs to the development side of the frozen fold map;
3. at least the following 24 raw hours exist so a 24-hour moving MPC horizon can be formed near the episode end;
4. at least seven previous complete model days exist for causal forecast construction and the one allowed forecast modification;
5. the date is not Q0-exposed;
6. the date is not part of QUAL48;
7. the date is not part of the frozen final-test side of the fold map.

## 9.2 Outcome-blind feature vector

The TPEC frozen power-balance model is the EV + SWRO + BESS + renewable/firming model. No new non-flexible electrical base-load term is introduced by this amendment.

For every eligible day \(d\), compute only pre-controller/exogenous quantities:

\[
f_d =
[
E_{\mathrm{ren},d},
P_{\mathrm{ren,min},d},
P_{\mathrm{ren,max},d},
E_{\mathrm{EV,req},d},
C_{\mathrm{EV,cap},d},
S_{\mathrm{scarcity},d},
\sin(2\pi m_d/12),
\cos(2\pi m_d/12)
].
\]

Where:

- \(E_{\mathrm{ren},d}\): total daily realized PV + wind energy;
- \(P_{\mathrm{ren,min},d}\): minimum hourly renewable power;
- \(P_{\mathrm{ren,max},d}\): maximum hourly renewable power;
- \(E_{\mathrm{EV,req},d}\): total announced daily EV service target across the frozen EV groups;
- \(C_{\mathrm{EV,cap},d}\): total daily EV network-capacity energy, obtained from the frozen per-hour EV cap profiles;
- \(S_{\mathrm{scarcity},d}\): an exogenous scarcity proxy:
  \[
  S_{\mathrm{scarcity},d}
  =
  E_{\mathrm{EV,req},d}
  +
  E_{\mathrm{water,min},d}
  -
  E_{\mathrm{ren},d},
  \]
  where \(E_{\mathrm{water,min},d}\) is the frozen daily water requirement multiplied by the minimum electrical energy per unit water permitted by the frozen SWRO graph, used **only as a selection covariate**;
- \(m_d\): calendar month.

No controller output, solver result, certificate result, MIP gap, runtime, or label status may appear in this vector.

## 9.3 Deterministic maximin selection

Normalize each non-circular scalar feature to \([0,1]\) using the minimum and maximum over the eligible development-day population. A feature with zero population range contributes zero distance.

Then select exactly 10 dates:

1. **Seed day:** day with maximum \(S_{\mathrm{scarcity},d}\); tie-break by earliest timestamp.
2. **Greedy expansion:** repeatedly choose the remaining day maximizing its minimum Euclidean distance in normalized feature space to the already selected set.
3. **Tie-break:** earliest timestamp.
4. Continue until 10 dates are selected.

The selected identities are written to:

```text
TPEC2027_G5A_DEV10_EPISODES_V1_1.csv
```

and SHA-256 hashed before any controller/reference result is run.

No selected episode may later be replaced because it is:

- inconvenient;
- difficult;
- easy;
- poorly bounded;
- unfavorable to ML;
- unfavorable to MPC;
- unfavorable to the heuristic.

## 9.4 Initial physical state

Each independent 24-hour episode begins from a controlled initial state:

\[
SOC_0 = \frac{SOC_{\min}+SOC_{\max}}{2},
\]

\[
V_0 = \frac{V_{\min}+V_{\max}}{2}.
\]

The 00:00 EV remaining-energy state is set to the frozen announced daily target for that model day.

Before controller execution, the initial state must pass the frozen physical residual checks and B1-ZR starting-state certificate.

If the exact midpoint state is not B1-ZR certified, do **not** replace the episode. Instead select the nearest certified state from the already frozen 3×3 SOC/tank grid using:

1. Euclidean distance from the physical midpoint after range normalization;
2. lexical grid identifier as tie-break.

The selected initial state and the full announced EV target/cap schedule are recorded in the episode manifest.

---

# 10. Causal information available to heuristic, MPC, and ML

The paper does not introduce a trainable forecasting contribution.

The causal information set is therefore frozen explicitly and is identical across heuristic, MPC, and ML.

## 10.1 Current interval

At decision time \(t\), every controller observes:

- the true current physical state;
- true current PV and wind generation;
- current EV remaining-energy state;
- current tank state;
- current BESS SOC;
- the current interval's frozen EV network caps;
- all static physical parameters needed to interpret the state.

## 10.2 Announced EV obligations

The frozen Gate-4B model treats daily aggregate EV service targets as **announced service targets** and uses the per-hour EV network-capacity schedule inside the teacher and B1-ZR continuation formulation.

V1.1 therefore makes this modeling assumption explicit:

> **The aggregate daily EV service target and its frozen per-hour availability/capacity envelope are contractual/announced information available to the EMS when that model day enters the controller horizon.**

Accordingly, heuristic, MPC, ML, and B1-ZR all receive the same announced EV target/cap schedule.

This is not an ML feature inferred from future realized renewable output. It is part of the frozen EV service model.

## 10.3 Future renewable forecast

For future renewable generation, preserve the frozen seasonal-naive semantics:

- current interval: use observed PV and wind;
- future interval \(t+\ell\), \(\ell\ge1\): use the same-hour PV/wind from the most recently completed model day, repeated every 24 hours.

Symbolically,

\[
\widehat{P}_{\mathrm{PV},t+\ell|t}
=
P_{\mathrm{PV},t+\ell-24},
\]

\[
\widehat{P}_{\mathrm{wind},t+\ell|t}
=
P_{\mathrm{wind},t+\ell-24}.
\]

Water demand follows the frozen known demand schedule.

No future realized renewable value is supplied to a causal controller.

## 10.4 Bounded single MPC-improvement forecast

The primary forecast above is not tuned.

If and only if Day 2 enters the `INDETERMINATE` branch and the time-limit diagnostic in §15.4 does not take priority, the one allowed forecast modification is:

> replace previous-day same-hour renewable persistence by the **median of the same hour over the previous seven completed model days**.

The EV announced-target/cap information set does not change.

No second forecast modification is permitted under V1.1.

---

# 11. Perfect-foresight physical reference

Each of the 10 development episodes receives one offline T1 reference solve.

## 11.1 Purpose

The reference provides a lower bound on the minimum 24-hour realized firming energy under the frozen physical model.

It is **not** an online controller and is **not** subject to B1-ZR.

## 11.2 Information

The PF problem uses the realized 24-hour exogenous trajectory for that episode.

## 11.3 Horizon

Exactly 24 hourly intervals from the episode's frozen initial condition.

## 11.4 Objective

T1 only:

> minimize total 24-hour firming energy.

No T2/T3 optimality claim is made by the PF reference.

## 11.5 Solver budget

```text
time limit: 600 s per episode
threads: 1
parallel: off
seed: 0
same numerical feasibility tolerances as the frozen model
```

The exact HiGHS version and relevant options must be recorded in the run manifest.
## 11.6 Optimality is not required

A time-limited run is useful if it returns a valid finite dual lower bound.

For episode \(d\):

\[
z_{LB,PF,d}\le z^*_{PF,d}.
\]

If the solver-derived lower bound is unavailable or invalid, use the strongest mathematically valid authenticated lower bound available.

Because T1 firming energy is non-negative:

\[
z_{LB,PF,d}=0
\]

is always an admissible final fallback lower bound.

The use of the trivial bound must be flagged explicitly.

## 11.7 Why loose bounds remain in the primary analysis

The physical headroom bound is used **only in the STOP direction**.

A looser lower bound makes the apparent physical headroom larger.

Therefore bound looseness:

- can make STOP harder to trigger;
- can make the result less informative;
- cannot create a false GO.

For that reason, no episode is dropped from the primary population because its lower bound is loose.

## 11.8 Required PF metadata

```text
episode_id
initial_state
terminal_SOC_of_best_incumbent
terminal_tank_of_best_incumbent
remaining_terminal_EV_obligation
dual_lower_bound
primal_incumbent_if_any
relative_gap_if_defined
absolute_gap_if_defined
native_status
elapsed_s
time_limit_binding
trivial_bound_used
reference_horizon
terminal_condition_definition
model/input hashes
```

---

# 12. Strong heuristic proposal baseline

The heuristic is intentionally simple but must not be a strawman.

It uses least-laxity / earliest-deadline EV scheduling, a tank-threshold SWRO rule, and a deterministic residual BESS rule. Its output is the same full canonical action used by MPC and ML.

## 12.1 EV mandatory-charge calculation

For each currently serviceable EV group \(i\):

- remaining required energy: \(e_i\);
- remaining available charging intervals including current interval: \(n_i\);
- maximum current/future charging power from the announced cap schedule: \(p_i^{\max}\);
- interval length: \(\Delta t\).

Use the frozen EV-service convention for delivered charging energy. Define the current minimum service needed to preserve the possibility of completing the remaining announced obligation under maximum future service.

Sort EV groups by:

1. increasing laxity;
2. earliest deadline / service endpoint;
3. fixed group identifier.

The exact formula must be derived from the frozen EV service equality used by the physical model. No new charging-efficiency factor may be introduced unless it is present in that frozen equality.

## 12.2 SWRO threshold rule

Define:

\[
V_{\mathrm{mid}}
=
\frac{V_{\min}+V_{\max}}{2}.
\]

First compute the minimum current production required to keep the next tank state above \(V_{\min}\).

Then:

- if tank inventory is below \(V_{\mathrm{mid}}\), use available current renewable surplus to increase SWRO production toward the midpoint;
- if tank inventory is at or above \(V_{\mathrm{mid}}\), propose only the minimum amount needed by the one-step tank rule;
- decode the resulting proposal through the frozen SWRO ON/OFF and piecewise-linear feasible-flow graph.

No future realized renewable information is used.

## 12.3 Opportunistic EV charging

After mandatory EV service and the SWRO proposal are formed, compute current renewable residual using the **realized current** PV + wind.

Any positive residual is allocated to connected EV groups in least-laxity order, up to the current frozen network caps and remaining announced energy requirement.

## 12.4 Deterministic BESS proposal

Let \(P_{\mathrm{flex}}\) be the electrical power required by the proposed EV + SWRO action after the frozen SWRO graph is evaluated, and let \(P_{\mathrm{ren}}\) be realized current renewable power.

If \(P_{\mathrm{ren}}>P_{\mathrm{flex}}\):

- propose battery charging from the surplus;
- limit charging by the frozen BESS power bound and one-step SOC headroom;
- set `battery_net_kw < 0`.

If \(P_{\mathrm{ren}}<P_{\mathrm{flex}}\):

- propose battery discharge against the deficit;
- limit discharge by the frozen BESS power bound and one-step SOC energy availability;
- set `battery_net_kw > 0`.

If equal, propose `battery_net_kw = 0`.

Any remaining deficit becomes firming and any remaining surplus becomes curtailment through the frozen residual-only balance.

## 12.5 Common safety application

The full heuristic proposal

```text
[p1_kw, p34_kw, qro_m3h, battery_net_kw, swro_on]
```

is passed through the common decoder and B1-ZR/fallback interface in §4.

The heuristic is never granted a weaker or stronger safety regime than MPC or ML.

---

# 13. Causal receding-horizon MPC comparator

The MPC comparator represents the online optimization method that ML is intended to approximate or replace.

## 13.1 Primary MPC objective

The primary online MPC optimizes **T1 firming energy only**.

This is deliberate because:

- T1 is the headline quality objective;
- the Day-2 headroom analysis is T1-only;
- the PF lower bound is T1-only;
- avoiding T2/T3 online stages keeps the comparator computationally interpretable.

The MPC is not used to claim lexicographic T2/T3 optimality.

## 13.2 Prediction horizon

```text
24 hourly intervals at every decision
```

The MPC uses the causal information set in §10.

The online MPC optimization itself uses `terminal_b1 = FALSE`; first-action continuation safety is imposed uniformly by the external common B1-ZR filter in §4. This prevents MPC from receiving a different embedded safety regime from the heuristic or ML controller.

## 13.3 Receding-horizon operation

At each hourly decision:

1. observe the true current state;
2. build the 24-hour causal renewable forecast and announced EV target/cap horizon from §10;
3. solve the frozen T1 physical model for at most 60 s;
4. if a physically verified feasible incumbent exists, extract its full first action:
   ```text
   p1_kw
   p34_kw
   qro_m3h
   battery_net_kw
   swro_on
   ```
5. if no feasible incumbent exists, use the full heuristic proposal for that interval as the MPC proposal;
6. pass the full proposal through the same decoder and successor B1-ZR/fallback interface as every other controller;
7. execute one interval using realized current renewable output;
8. advance SOC, tank, and remaining EV states with the frozen transition;
9. repeat.

## 13.4 Solver settings

```text
time limit: 60 s per decision
threads: 1
parallel: off
seed: 0
highspy: 1.15.1
frozen physical feasibility tolerances
no case-specific option changes
no retries
```

Warm-starting is **OFF in V1.1** unless the frozen solver interface already provides an outcome-independent deterministic warm-start mechanism. Enabling a new warm-start method would require another versioned amendment.

## 13.5 Time-limit status handling

A time-limited MPC decision is not automatically valid.

Each call is classified as:

```text
OPTIMAL_OR_CERTIFIED
FEASIBLE_INCUMBENT
NO_FEASIBLE_INCUMBENT
INVALID
```

`FEASIBLE_INCUMBENT` requires:

- a finite incumbent vector;
- frozen bound/integrality/linear residual checks to pass;
- a valid full first action after extraction.

A timeout alone never implies a usable action.

- `OPTIMAL_OR_CERTIFIED`: full first action may be used as the proposal.
- `FEASIBLE_INCUMBENT`: full first action may be used as the proposal after the checks above.
- `NO_FEASIBLE_INCUMBENT`: use the heuristic proposal before B1-ZR.
- `INVALID`: hard implementation failure requiring investigation.

A timeout incumbent is never called optimal.

---

# 14. MPC runtime reporting

The MPC runtime result must not be reduced to one mean.

Report:

```text
time_limit_s
fraction_hitting_time_limit
fraction_OPTIMAL_OR_CERTIFIED
fraction_FEASIBLE_INCUMBENT
fraction_NO_FEASIBLE_INCUMBENT
p50_elapsed_all
p90_elapsed_all
p95_elapsed_all
mean_elapsed_all
p50_elapsed_nonbinding
p90_elapsed_nonbinding
mean_elapsed_nonbinding
gap_distribution_for_time_limited_feasible_incumbents
node_count_distribution_if_available
```

Time-limited observations are **right-censored with respect to natural time-to-proof**, even though their deployed decision latency is exactly the observed time limit.

The paper must distinguish:

- deployed MPC decision latency;
- natural solve-time distribution among non-binding calls;
- proof status at the time limit.

---

# 15. Day-2 headroom metrics and gate

## 15.1 Physical-model headroom upper bound

Use the energy guard

\[
\epsilon_E = 10^{-6}\ \mathrm{kWh}.
\]

For an episode with \(z_{\mathrm{heur},d}>\epsilon_E\):

\[
H_{\mathrm{phys},d}^{UB}
=
\frac{
z_{\mathrm{heur},d}
-
z_{LB,PF,d}
}{
z_{\mathrm{heur},d}
}.
\]

If \(z_{\mathrm{heur},d}\le\epsilon_E\), report the episode percentage as `N/A`; the absolute firming values remain in the table.

For the aggregate, if

\[
\sum_d z_{\mathrm{heur},d}>\epsilon_E,
\]

define:

\[
H_{\mathrm{phys}}^{UB}
=
\frac{
\sum_d z_{\mathrm{heur},d}
-
\sum_d z_{LB,PF,d}
}{
\sum_d z_{\mathrm{heur},d}
}.
\]

If aggregate heuristic firming is \(\le\epsilon_E\), the firming-quality branch automatically returns `STOP_QUALITY_ALREADY_ZERO` because there is no material heuristic firming left to reduce.

This is an **upper bound** on the fraction of heuristic firming removable under the physical model.

It is not a guarantee that a causal controller can capture the bound.

## 15.2 Demonstrated causal MPC improvement

When

\[
\sum_d z_{\mathrm{heur},d}>\epsilon_E,
\]

define:

\[
H_{\mathrm{MPC}}
=
\frac{
\sum_d z_{\mathrm{heur},d}
-
\sum_d z_{\mathrm{MPC},d}
}{
\sum_d z_{\mathrm{heur},d}
}.
\]

If aggregate heuristic firming is \(\le\epsilon_E\), report `H_MPC = N/A`.

Because MPC can be worse than the heuristic, \(H_{\mathrm{MPC}}\) may be negative.

The demonstrated causal improvement available from the tested controller family is therefore summarized as:

\[
H_{\mathrm{causal,dem}}
=
\max(0,H_{\mathrm{MPC}}).
\]

Do not call the PF-to-MPC difference "value of information."

Use:

> **PF-to-MPC opportunity gap**

because the difference can contain:

- foresight value;
- forecast error;
- finite MPC horizon;
- terminal effects;
- solver time-limit effects;
- MPC formulation deficiency.

## 15.3 Strategic Day-2 threshold

Freeze:

\[
\boxed{\delta_H = 0.05}
\]

That is, **5% aggregate firming reduction relative to the heuristic** is the predeclared minimum magnitude for keeping a standalone firming-quality contribution as a central TPEC claim.

This is an **author-declared strategic research threshold**, not a statistical significance level and not a literature-derived physical constant.

It is frozen before the 10-episode results are inspected.

## 15.4 Three-way Day-2 decision

### STOP QUALITY CLAIM

If:

\[
H_{\mathrm{phys}}^{UB}<\delta_H,
\]

then even the physical-model upper bound leaves less than the predeclared quality headroom.

Action:

```text
DROP firming-improvement as a central paper claim.
DO NOT tune ML for firming.
Continue only if runtime/safety/composability contribution remains compelling.
```

### GO TO ML

If:

\[
H_{\mathrm{MPC}}\ge\delta_H,
\]

then meaningful causal firming improvement has already been demonstrated.

Action:

```text
Proceed to the Day-3 ML prototype.
```

### INDETERMINATE

If:

\[
H_{\mathrm{MPC}}<\delta_H\le H_{\mathrm{phys}}^{UB},
\]

then causal quality opportunity remains unresolved.

Only **one** bounded MPC-improvement cycle is allowed.

Priority:

1. if MPC time-limit binding fraction exceeds 25% **or** any `NO_FEASIBLE_INCUMBENT` event occurred, change only:
   ```text
   per-decision MPC time limit: 60 s → 120 s
   ```
2. otherwise change only:
   ```text
   forecast: previous-day same-hour persistence
   → previous-7-day same-hour median
   ```

All other MPC settings remain unchanged.

Re-run the same 10 frozen episodes exactly once.

If the amended MPC still gives:

\[
H_{\mathrm{MPC}}<\delta_H,
\]

the firming-quality branch terminates.

No second MPC-design modification is allowed under V1.1.

---

# 16. Day-3 ML prototype

The first ML model is intentionally simple and is not architecture-searched.

## 16.1 Model class

Use a standard feed-forward MLP with the literature-aligned hidden trunk:

```text
6 hidden layers
10 neurons per hidden layer
GELU activation
```

The output layer is split into:

```text
4 linear continuous heads:
    p1_kw
    p34_kw
    qro_m3h
    battery_net_kw

1 binary logit head:
    swro_on
```

No architecture search occurs before the Day-3 decision.

## 16.2 Inputs

The ML information set must match the causal information available to MPC.

It includes:

- current SOC;
- current tank inventory;
- current EV remaining-energy state;
- current interval EV network caps;
- announced EV target/cap information over the causal horizon;
- current realized PV and wind;
- the same 24-hour causal renewable forecast supplied to MPC;
- static frozen parameters required to interpret normalized state/action quantities.

It excludes:

- future realized renewable values;
- PF lower bounds;
- solver dual bounds;
- teacher solver status;
- future controller outcomes.

## 16.3 Full-action targets and inference decoding

The MLP predicts the **same full proposal action** used by the teacher and MPC.

Continuous targets:

```text
target_p1_kw
target_p34_kw
target_qro_m3h
target_battery_net_kw
```

Binary target:

```text
target_swro_on
```

At inference:

1. inverse-transform continuous outputs using training-only statistics;
2. set `swro_on = 1` when sigmoid(logit) \(\ge0.5\), otherwise 0;
3. apply the common deterministic local decoder in §4.2;
4. apply the common successor B1-ZR/fallback interface in §4.3–§4.4.

No controller-specific repair is allowed.

## 16.4 Training targets

Targets come only from `TRAINING_ELIGIBLE` development labels defined in §7.

A training label must provide or losslessly reconstruct the complete canonical action.

Unresolved T1 labels are usable only when:

- the action is physically verified;
- the provenance binding is intact;
- their continuous T1 quality metadata and realized T1 epsilon ceiling are retained.

## 16.5 Standardization and loss

All continuous input/output normalization statistics are computed from the **training subset only**.

No statistic may use:

- any record from a DEV10 date;
- final-test data;
- QUAL48.

Loss:

\[
\mathcal{L}
=
\operatorname{mean}\left(
\operatorname{Huber}(
\hat{a}_{\mathrm{cont}},
a_{\mathrm{cont}}
)
\right)
+
\operatorname{BCEWithLogits}(
\hat{s}_{\mathrm{on}},
s_{\mathrm{on}}
).
\]

All four continuous action components are normalized before the Huber term, so no hand-picked physical-unit weighting is introduced in V1.1.

## 16.6 Split and DEV10 exclusion

Use the frozen development training/validation structure from the existing fold map, **after removing every record whose model date is one of the 10 DEV10 dates**.

The exclusion is date-wide:

```text
DEV10 date -> no training record
DEV10 date -> no validation record
DEV10 date -> no normalization statistic
DEV10 date -> no label-quality threshold derivation
```

Do not create a random record-level split that mixes neighboring temporal states.

Before training, the development-ledger builder must emit and verify:

```text
n_train_records_from_DEV10_dates = 0
n_validation_records_from_DEV10_dates = 0
```

Any nonzero value is a hard leakage failure.

## 16.7 Training procedure

For the Day-3 prototype:

```text
optimizer: Adam
learning rate: 1e-3
loss: normalized continuous Huber + swro_on BCE
batch size: 128
max epochs: 200
early-stopping patience: 20 validation epochs
seed: 20261009
one training run
no hyperparameter sweep
```

The exact software versions, seed, training/validation manifest hashes, and DEV10 exclusion hash are recorded.

---

# 17. Day-3 quality and runtime metrics

## 17.1 ML firming regret upper bound

For the 10-episode aggregate:

\[
R_{\mathrm{ML}}^{UB}
=
\frac{
\sum_d z_{\mathrm{ML},d}
-
\sum_d z_{LB,PF,d}
}{
\sum_d z_{LB,PF,d}
},
\]

only if the aggregate denominator exceeds the numerical reporting guard

\[
\epsilon_E = 10^{-6}\ \mathrm{kWh}.
\]

If the denominator is \(\le\epsilon_E\), report only the absolute bound:

\[
\sum_d z_{\mathrm{ML},d}
-
\sum_d z_{LB,PF,d}.
\]

## 17.2 Conservative physical-headroom capture index

\[
C_{\mathrm{PF}}
=
\frac{
\sum_d z_{\mathrm{heur},d}
-
\sum_d z_{\mathrm{ML},d}
}{
\sum_d z_{\mathrm{heur},d}
-
\sum_d z_{LB,PF,d}
}.
\]

Use only when:

\[
\sum z_{\mathrm{heur}}
-
\sum z_{LB,PF}
>
\epsilon_H.
\]

Otherwise report `N/A`.

Interpretation:

> conservative physical-headroom capture index

not exact fraction of true available headroom.

## 17.3 MPC-improvement capture ratio

\[
C_{\mathrm{MPC}}
=
\frac{
\sum_d z_{\mathrm{heur},d}
-
\sum_d z_{\mathrm{ML},d}
}{
\sum_d z_{\mathrm{heur},d}
-
\sum_d z_{\mathrm{MPC},d}
}.
\]

Use only if:

\[
\sum z_{\mathrm{heur}}
-
\sum z_{\mathrm{MPC}}
>0.
\]

Otherwise report `N/A` and use:

\[
\Delta z_{\mathrm{ML-MPC}}
=
\sum_d z_{\mathrm{ML},d}
-
\sum_d z_{\mathrm{MPC},d}.
\]

## 17.4 Direct ML-versus-MPC degradation

If

\[
\sum_d z_{\mathrm{MPC},d}>\epsilon_E,
\]

define:

\[
D_{\mathrm{ML|MPC}}
=
\frac{
\sum_d z_{\mathrm{ML},d}
-
\sum_d z_{\mathrm{MPC},d}
}{
\sum_d z_{\mathrm{MPC},d}
}.
\]

If the aggregate MPC denominator is \(\le\epsilon_E\), report `D_ML|MPC = N/A` and use the absolute difference

\[
\Delta z_{\mathrm{ML-MPC}}
=
\sum_d z_{\mathrm{ML},d}
-
\sum_d z_{\mathrm{MPC},d}.
\]

This is the primary non-inferiority quantity for comparison with the closest published approximate-MPC competitor when its denominator is defined.

---

# 18. Source-backed Day-3 comparator rule

The closest direct competitor is:

> Changrui Liu, Shengling Shi, Anil Alan, Ganesh Kumar Venayagamoorthy, and Bart De Schutter, "Approximate model predictive control for microgrid energy management via imitation learning," *Engineering Applications of Artificial Intelligence*, vol. 182, article 115837, 2026. DOI: 10.1016/j.engappai.2026.115837.

The final published paper reports that the learned policy has economic performance **comparable** to EMPC and approximately one-order-of-magnitude lower computation time. It evaluates closed-loop economic cost against expert EMPC across a large test set and identifies a nominal EMPC configuration with three fuel generators and a 1-hour prediction window.

The indexed text does not provide one scalar nominal percentage degradation that can be copied directly into this protocol.

Therefore the amendment freezes the **extraction procedure**, not an invented number.

## 18.1 Pre-result Liu extraction

Before any TPEC Day-3 ML test result is inspected:

1. obtain the final published Figure 5 economic-cost comparison;
2. extract the nominal configuration:
   ```text
   N_fg = 3
   T = 12 five-minute steps = 1 hour
   ```
3. digitize the proposed-IL and expert-EMPC economic-cost values from the figure using a recorded plot-digitization workflow;
4. compute:
   \[
   \epsilon_{\mathrm{Liu}}
   =
   \max\left(
   0,
   \frac{J_{\mathrm{IL}}-J_{\mathrm{EMPC}}}{J_{\mathrm{EMPC}}}
   \right);
   \]
5. include digitization uncertainty conservatively by using the upper end of the extracted uncertainty interval;
6. round **upward** to the nearest 0.5 percentage point;
7. write the result and extraction evidence to:
   ```text   TPEC2027_LIU2026_NONINFERIORITY_THRESHOLD_V1_1.json
   ```
8. hash and anchor that file before viewing the TPEC ML test result.

This prevents the threshold from being fitted to the new results.

## 18.2 Day-3 non-inferiority criterion

The source-backed quality criterion is:

\[
D_{\mathrm{ML|MPC}}
\le
\epsilon_{\mathrm{Liu}}.
\]

`C_MPC` remains a descriptive companion metric; the source-backed threshold is applied to the directly commensurate ML-versus-optimizer degradation.

## 18.3 Runtime context

Liu et al. report approximately an order-of-magnitude online computation reduction.

Therefore the TPEC paper must report:

\[
\rho_t
=
\frac{
\operatorname{median}(t_{\mathrm{ML}}+t_{\mathrm{B1ZR}})
}{
\operatorname{median}(t_{\mathrm{MPC,deployed}})
}.
\]

A value near or below 0.1 is a strong literature-aligned runtime result, but **V1.1 does not use \(\rho_t\) as a hard GO/STOP threshold** because MPC time-limit binding can materially affect the denominator.

---

# 19. Day-3 GO / MODIFY / STOP rule

The Day-3 prototype produces an actual paper-level decision.

## 19.1 Hard safety requirement

For the common-filter architecture:

```text
physical violations after B1-ZR/fallback: 0
fallback implementation failures: 0
```

Any violation is a Gate-5 implementation failure.

## 19.2 GO

GO if all hold:

1. the 10 development episodes complete under ML+B1-ZR;
2. there are zero post-filter physical violations;
3. the direct non-inferiority criterion passes:
   \[
   D_{\mathrm{ML|MPC}}\le\epsilon_{\mathrm{Liu}};
   \]
4. raw ML proposal behavior is non-degenerate, meaning the controller does not simply rely on fallback for nearly every decision.

For item 4, no arbitrary numerical rejection threshold is used at this stage. The measured rejection/fallback rate is reported and interpreted jointly with the rejection audit.

## 19.3 MODIFY

If safety passes but the Liu-based quality condition fails, permit **one** ML modification cycle only.

The modification must be predeclared before rerunning and may change only one of:

```text
network width
loss weighting across action components
training-label inclusion threshold
```

No data split or evaluation episode may change.

After one rerun:

- pass → GO;
- fail → drop the near-MPC-quality claim.

## 19.4 STOP / PIVOT

If:
- post-filter physical feasibility fails, or
- the single allowed ML modification still fails the source-backed quality criterion,

then:

> the paper may continue only if the measured runtime/safety/composability contribution is independently strong enough to justify a revised claim.

No further tuning loop is allowed under V1.1.

---

# 20. Rejection audit for B1-ZR

For ML proposals rejected by B1-ZR, select a predeclared audit sample.

## 20.1 Sample rule

Before inspecting continuation outcomes:

- if total ML rejections \(N_R\le30\): audit all;
- if \(N_R>30\): audit 30 using deterministic evenly spaced indices over chronological rejection order.

## 20.2 Continuation outcomes

Each audited rejection is classified:

```text
CONFIRMED_FEASIBLE
CONFIRMED_INFEASIBLE
UNRESOLVED
```

- `CONFIRMED_FEASIBLE`: a valid continuation is found, proving B1-ZR was conservative for that proposal.
- `CONFIRMED_INFEASIBLE`: infeasibility is actually certified by the frozen continuation formulation.
- `UNRESOLVED`: neither conclusion is certified within the audit budget.

No timeout is reinterpreted as infeasible.

## 20.3 Reporting

Report:

- total rejection rate;
- audited count;
- confirmed-feasible fraction;
- confirmed-infeasible fraction;
- unresolved fraction;
- a binomial confidence interval for confirmed-infeasible fraction when sample size permits.

---

# 21. Secondary unfiltered-MPC safety experiment

After the primary Day-2/Day-3 results are complete, MPC may be re-run without the B1-ZR layer on the same 10 development episodes.

Purpose:

> determine whether physical-feasibility failures are observed for the optimization-based proposal mechanism when the common certificate/fallback layer is removed.

Required wording if no failures occur:

> **No unfiltered-MPC physical-feasibility failure was observed on the tested development episodes.**

Do not generalize this to:

> the safety layer is unnecessary for MPC in general.

If an unfiltered MPC trajectory violates the frozen physical constraints, stop that trajectory at the first invalid transition and record the failure.

This experiment is secondary and must not alter Day-2/Day-3 controller tuning.

---

# 22. Limited Strategy-D diagnostic

Strategy D is no longer a blocking gate.

Run exactly three cases, once each, with the already frozen Strategy-D settings and 3600-s T1 limit.

The selection is deliberately favorable to certification.

Use the three smallest clean unresolved relative gaps observed across completed B/C evidence:

```text
record 165 — EP_20230213_S1
record   4 — EP_20230104_S0
record 164 — EP_20230213_S0
```

Run order:

```text
165 → 4 → 164
```

No retries.

No additional D cases may be added under V1.1.

If none certifies, the allowed statement is:

> **None of the three predeclared near-threshold unresolved cases certified within 3600 s under the proof-focused configuration.**

Do not claim that every nominally harder case would also fail.

The diagnostic may run on Day 4 or in parallel after the Day-3 decision. It may not delay the first ML result.

---

# 23. RQ5 continuous teacher-quality study

Teacher-quality sensitivity and sample-count sensitivity are separated.

## 23.1 Quality-controlled sweep

Before looking at results, compute the available distribution of T1 relative gaps in the development ledger.

Freeze the following threshold grid **by quantiles of the available development-label gap distribution**, not by hand-picked percentages:

```text
q25
q50
q75
q100
```

The actual numeric gap thresholds are written to a threshold manifest before model training.

For each threshold:

- include labels at or below the threshold;
- subsample to the same training count \(N_{\min}\), equal to the smallest eligible set size across thresholds;
- use fixed seeds:
  ```text
  1103
  2207
  3301
  4409
  5519
  ```
- train/evaluate the same MLP recipe.

This tests label-quality effect at fixed sample count.

## 23.2 Learning curve

Separately hold the inclusion rule fixed at `q100` and train on:

```text
25%
50%
75%
100%
```

of the eligible training pool using the same fixed seeds.

This tests sample-count effect separately.

The 10 DEV10 dates remain evaluation-only for this sensitivity study. All records on those dates are excluded from every RQ5 training and validation pool.

---

# 24. Within-day versus multi-day scope

The 10 independent 24-hour episodes provide:

> **within-day sequential closed-loop evidence**

They do **not** provide evidence of persistent multi-day drift because storage/tank initial conditions are reset at each independent episode start.

The manuscript must not describe the 10-episode prototype as multi-day continuous closed-loop validation.

After the architecture passes Day 3, one separate continuous stress test is required before final paper claims.

For that test:

\[
SOC_{d+1,0}=SOC_{d,24},
\]

\[
V_{d+1,0}=V_{d,24},
\]

while EV obligations reset according to the frozen daily rule.

The continuous test is performed on a later held-out development stress interval or final-test interval according to the subsequent frozen evaluation plan.

---

# 25. Decision-driven experiment rule

The old artifact-frequency rule is retired.

The governing rule is:

> **Every computational experiment must answer a predeclared decision question.**

Before every solve/train/run, the run sheet must state:

```text
QUESTION
INPUT POPULATION
FROZEN SETTINGS
OUTPUT METRICS
DECISION RULE
WHAT CHANGES IF PASS
WHAT CHANGES IF FAIL
```

A hash, receipt, plot, or log that does not change or resolve a scientific decision is supporting evidence, not a milestone.

---

# 26. Seven-day results-first execution schedule

| Day | Work | Mandatory output | Decision |
|---|---|---|---|
| **0** | Anchor this amendment; build provenance ledger; select and hash DEV10 | amendment anchor + ledger + DEV10 manifest | protocol active? |
| **1** | Implement strong heuristic and causal forecast; run static/unit checks | executable heuristic + forecast verification | baseline valid? |
| **1–2** | Run PF T1 references and causal MPC on DEV10 | PF lower bounds + MPC closed-loop table + runtime censoring table | Day-2 gate |
| **2** | Run heuristic on DEV10; compute \(H_{\mathrm{phys}}^{UB}\) and \(H_{\mathrm{MPC}}\) | **first real EMS results table** | STOP / GO / INDETERMINATE |
| **3** | Build development-label ledger; train one MLP; evaluate ML on same DEV10 | heuristic vs MPC vs ML closed-loop table | GO / MODIFY / STOP |
| **4** | One allowed modification if needed; rejection audit; limited D diagnostic | corrected ML result + rejection evidence + D evidence | freeze architecture? |
| **5** | RQ5 fixed-N gap sweep + learning curve | quality-sensitivity table/figure | teacher-quality interpretation |
| **6** | Freeze final architecture and final-test protocol | final evaluation manifest | ready for untouched test? |
| **7** | Run adversarial three-reviewer checkpoint | PASS / CONDITIONAL / FAIL | advance to Gate 6 or pivot |

---

# 27. Mandatory Day-2 output table

The Day-2 artifact must contain, for every one of the 10 frozen episodes:

```text
episode_id
z_heur
z_MPC
z_LB_PF
H_phys_UB
H_MPC
PF_native_status
PF_bound_type
PF_trivial_bound_used
MPC_time_limit_binding_fraction
MPC_no_incumbent_fraction
heuristic_local_physical_reject_rate
MPC_local_physical_reject_rate
heuristic_B1ZR_rejection_rate
MPC_B1ZR_rejection_rate
heuristic_total_intervention_rate
MPC_total_intervention_rate
episode_complete_heur
episode_complete_MPC
```

And aggregate:

```text
sum_z_heur
sum_z_MPC
sum_z_LB_PF
H_phys_UB_aggregate
H_MPC_aggregate
Day2_decision
```

This table is the first mandatory paper-level result.

---

# 28. Mandatory Day-3 output table

For every episode:

```text
episode_id
z_heur
z_MPC
z_ML
z_LB_PF

H_phys_UB
H_MPC
D_ML_given_MPC
C_PF
C_MPC

heuristic_local_physical_reject_rate
MPC_local_physical_reject_rate
ML_local_physical_reject_rate

heuristic_B1ZR_rejection_rate
MPC_B1ZR_rejection_rate
ML_B1ZR_rejection_rate

heuristic_total_intervention_rate
MPC_total_intervention_rate
ML_total_intervention_rate

heuristic_fallback_rate
MPC_fallback_rate
ML_fallback_rate

EV_completion_heur
EV_completion_MPC
EV_completion_ML

tank_min_heur
tank_min_MPC
tank_min_ML

SOC_min_heur
SOC_min_MPC
SOC_min_ML

episode_complete_heur
episode_complete_MPC
episode_complete_ML

heuristic_decision_time
MPC_decision_time
ML_inference_time
B1ZR_time
ML_total_time
```

Aggregate headline metrics:

```text
H_phys_UB
H_MPC
D_ML_given_MPC
C_PF
C_MPC
R_ML_UB
ML rejection rate
ML full-fallback rate
median ML+B1ZR time
median deployed MPC time
MPC time-limit binding fraction
```

---

# 28.1 Environmental execution control

Because prior forensic runs showed that laptop sleep/battery events can corrupt wall-clock runtime evidence, every PF, MPC, limited-D, and other long result-producing solver run must use the same environmental guard:

```text
Windows ACLineStatus = 1 before start
keep-awake request active during run
lid remains open
post-run System-event audit for sleep/resume/power transitions
```

If an operating-system power/sleep event overlaps a result-producing run:

1. retain the interrupted attempt and its logs;
2. classify it:
   ```text
   ENVIRONMENTAL_INVALID
   ```
3. do not use its runtime or scientific result;
4. permit exactly one **environment-only replay** of the same frozen episode/run with identical code, inputs, solver settings, and seed after AC power is restored;
5. record both attempts in the provenance ledger.

An environmental replay is not a scientific retry and may not change any model or solver setting.

If the replay is also environmentally interrupted, stop execution and resolve the hardware/power environment before further runs.

---

# 29. Reproducibility artifacts

The amended workflow must produce, at minimum:

```text
TPEC2027_Gate5_RESULTS_FIRST_AMENDMENT_V1_1.md
TPEC2027_G5A_DEV10_EPISODES_V1_1.csv
TPEC2027_G5A_DEV10_MANIFEST_V1_1.json
TPEC2027_G5A_DEVELOPMENT_LABEL_LEDGER_V1_1.*
TPEC2027_G5A_PF_REFERENCE_RESULTS_V1_1.*
TPEC2027_G5A_MPC_RESULTS_V1_1.*
TPEC2027_G5A_HEURISTIC_RESULTS_V1_1.*
TPEC2027_LIU2026_NONINFERIORITY_THRESHOLD_V1_1.json
TPEC2027_G5A_ML_PROTOTYPE_RESULTS_V1_1.*
TPEC2027_G5A_REJECTION_AUDIT_V1_1.*
TPEC2027_G5A_DAY2_DECISION_V1_1.md
TPEC2027_G5A_DAY3_DECISION_V1_1.md
```

Every artifact must carry:

- exact source/input hashes;
- exact code hashes;
- software versions;
- creation timestamp;
- upstream manifest identifier.

---

# 30. External comparator note

The amendment uses Liu et al. (2026) as the closest direct approximate-MPC comparator because that work:

- learns an MLP approximation of mixed-integer EMPC;
- evaluates the learned controller in closed loop;
- compares economic performance against expert EMPC;
- explicitly treats online mixed-integer solve time as a deployment limitation;
- reports comparable economic performance and approximately one-order-of-magnitude computation reduction.

The source is used only for the **Day-3 ML-versus-MPC non-inferiority context and runtime context**.

It is not used to claim that the physical system, objective, loads, or safety formulation are identical to the TPEC system.

---

# 30.1 Mandatory implementation preflight before any controller solve

Activation of this amendment authorizes implementation work, but no DEV10 controller/reference solve may begin until a zero-result preflight verifies all of the following:

```text
CHK_FULL_ACTION_SCHEMA
CHK_BATTERY_NET_SIGN_AND_SOC_TRANSITION
CHK_SWRO_ON_OFF_DECODER
CHK_COMMON_DECODER_IDENTITY_ACROSS_CONTROLLERS
CHK_CURRENT_PHYSICAL_AUDIT
CHK_FIRMING_CAP_AND_RESIDUAL_BALANCE
CHK_SUCCESSOR_B1_INTERFACE
CHK_CONSTRUCTIVE_FALLBACK_FIRST_ROW
CHK_FALLBACK_BATTERY_NET_ZERO
CHK_DAY_BOUNDARY_EV_RESET
CHK_ANNOUNCED_EV_TARGET_CAP_INFORMATION
CHK_CAUSAL_RENEWABLE_FORECAST_NO_FUTURE_REALIZED_LEAKAGE
CHK_DEV10_DATE_WIDE_TRAIN_VALID_EXCLUSION
CHK_QUAL48_NOT_ACCESSED
CHK_Q0_EXCLUDED
CHK_NO_CONTROLLER_RESULT_EXISTS_BEFORE_DEV10_HASH
CHK_AC_POWER_GUARD_AVAILABLE
CHK_SLEEP_EVENT_AUDIT_AVAILABLE
```

The preflight performs **zero optimization solves and zero ML training runs**.

Any failure blocks execution and must be corrected in one integrated implementation successor before any result-producing experiment begins.

---

# 31. Three-reviewer activation checkpoint

Before execution, three independent reviews are required.

## Reviewer 1 — optimization / mathematical validity

Must verify:

- PF lower-bound validity;
- no forecast-world bound is misused as a realized-world lower bound;
- headroom formulas;
- timeout/incumbent semantics;
- MPC status handling;
- no unsupported optimality claims.

Decision:

```text
PASS / CONDITIONAL PASS / FAIL
```

## Reviewer 2 — ML / control methodology

Must verify:

- date-wide DEV10 exclusion from training, validation, normalization, and RQ5 pools;
- causal information parity, including the explicit announced-EV assumption;
- identical full canonical action space across heuristic, MPC, and ML;
- identical decoder + B1-ZR + fallback regime;
- BESS/SOC and EV/tank successor propagation;
- MLP inputs exclude PF/oracle-only information;
- one-modification stop rules are enforceable.

Decision:

```text
PASS / CONDITIONAL PASS / FAIL
```

## Reviewer 3 — adversarial IEEE review

Must challenge:

- whether the heuristic is a credible non-strawman baseline;
- whether MPC is a fair online comparator;
- whether the main ML claim can survive the proposed metrics;
- whether runtime reporting is censored honestly;
- whether environmental sleep/power contamination is fail-closed;
- whether the early STOP rules prevent post-hoc rescue.

Decision:

```text
PASS / CONDITIONAL PASS / FAIL
```

Activation requires no unresolved blocking FAIL.

---

# 32. Formal authorization state after activation

If the activation checkpoint passes:

```text
C9 under old protocol: CANCELLED BY AMENDMENT
FULL D1–D9 campaign: CANCELLED AS BLOCKING WORK
QUAL48: SEALED
PRODUCTION: NOT AUTHORIZED
FINAL ML TRAINING: NOT AUTHORIZED

RESULTS-FIRST DEVELOPMENT LEDGER: AUTHORIZED
DEV10 HEURISTIC EXPERIMENT: AUTHORIZED
DEV10 PF REFERENCE: AUTHORIZED
DEV10 CAUSAL MPC: AUTHORIZED
DAY-3 DEVELOPMENT MLP PROTOTYPE: AUTHORIZED AFTER A TERMINATED DAY-2 DECISION (GO, or explicit runtime/safety pivot after STOP/INDETERMINATE exit)
LIMITED D DIAGNOSTIC: AUTHORIZED, NON-BLOCKING
```

---

# 33. Stop rules

The following loops are prohibited under V1.1:

- no repeated solver-option tuning after the single allowed MPC improvement;
- no repeated MLP hyperparameter search after the single allowed Day-3 modification;
- no replacing unfavorable DEV10 episodes;
- no dropping weak PF bounds from the primary population;
- no adding extra D cases because the three predeclared cases were inconvenient;
- no changing \(\delta_H\) after seeing Day-2 results;
- no changing the Liu extraction rule after seeing Day-3 results;
- no moving DEV10 episodes into the final test set;
- no treating B1-ZR rejection as proof of infeasibility;
- no treating timeout incumbents as proven optima.

Any further methodological change requires a new versioned amendment.

---

# 34. Final amendment decision

This amendment formally changes Gate 5 from:

> **certification-first, ML-later**

to:

> **decision-driven development with rigorous lower bounds, strong causal baselines, common safety architecture, and immediate closed-loop ML evidence.**

The first scientific milestone is no longer a solver receipt.

It is:

> **the Day-2 table comparing the strong heuristic, causal MPC, and perfect-foresight physical lower bound on 10 frozen development episodes.**

The second is:

> **the Day-3 closed-loop comparison of heuristic, MPC, and ML under the identical B1-ZR filter/fallback.**

Those two results determine whether the TPEC ML paper has a defensible quality contribution, a runtime/safety contribution, or requires a pivot.

---

# References

1. C. Liu, S. Shi, A. Alan, G. K. Venayagamoorthy, and B. De Schutter, “Approximate model predictive control for microgrid energy management via imitation learning,” *Engineering Applications of Artificial Intelligence*, vol. 182, art. 115837, 2026. DOI: 10.1016/j.engappai.2026.115837.

2. Existing TPEC 2027 frozen Gate-2/Gate-4/Gate-5 records, manifests, B1-ZR certificate definitions, fold map, state-source artifacts, and V5/V6 forensic audit trail remain controlling for all scientific details not explicitly amended here.

## Controlling frozen implementation identities used by V1.1

```text
teacher oracle SHA256 : 3a06bf6358b1e15f32a1ae645707f20afd0fdbcd0ff01cf277dd91badc2ab58c
B1-ZR SHA256          : 0d0c43651982fafd5f965b069cf0c6ae12391c5e9684666c4223de3517010df1
input SHA256          : 260e3f4ea97a3e06ea3f4de3191d5071997131f8e56e83a59ff7dd711ec829ad
fold-map SHA256       : a5034858b62562dadadd85e4d149db4a005ee45fec8ce7a0ce59dd6f6bc11724
highspy               : 1.15.1
```

These identities are provenance anchors. The amendment does not authorize changing their scientific semantics.
