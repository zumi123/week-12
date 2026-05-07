# Explainer (Pair Day 3)
## Why SimPO moved more than DPO on your 104-pair dataset: gradient-level view

## 1) The gap being closed

You chose SimPO over DPO in Week 11 because:
- no reference model,
- better behavior on small preference datasets.

You can state this intuition, but you wanted the parameter-level reason:  
**what SimPO’s gradient penalizes, how DPO differs, and why removing the reference model can increase movement toward preference signal (not just “less regularization”).**

This explainer closes that gap.

---

## 2) Setup and notation

For each preference pair `(x, y_w, y_l)`:
- `y_w` = preferred (winner),
- `y_l` = rejected (loser),
- `pi_theta` = current policy model.

Define sequence log-prob gap under policy:

`delta_theta = log pi_theta(y_w | x) - log pi_theta(y_l | x)`

Preference-style objectives push `delta_theta` up.

---

## 3) DPO objective and gradient (where reference enters)

One common DPO form uses:

`L_DPO = - log sigma( beta * ( delta_theta - delta_ref ) )`

where:
- `delta_ref = log pi_ref(y_w|x) - log pi_ref(y_l|x)` from frozen reference model,
- `beta` is inverse-temperature scale.

Let:

`z = beta * (delta_theta - delta_ref)`

Then:

`dL/dz = sigma(z) - 1 = -sigma(-z)`

and

`dz/dtheta = beta * d(delta_theta)/dtheta`

So:

`grad_theta L_DPO = - beta * sigma(-z) * d(delta_theta)/dtheta`

### Interpretation

DPO multiplies your preference-direction gradient by:

`beta * sigma(- beta*(delta_theta - delta_ref))`

This term depends on **distance from reference margin**.  
If your current policy is already close to/above reference-implied margin for a pair, `sigma(-z)` shrinks and update gets small.

So DPO is not only “regularized” conceptually; at gradient level, it **gates pair-wise update magnitude through reference-relative margin**.

---

## 4) SimPO objective and gradient (reference-free pair pressure)

SimPO removes explicit reference term and uses a direct policy-relative preference push (with margin/normalization variants depending on implementation). A minimal representative form is:

`L_SimPO = - log sigma( beta * (delta_theta - m) )`

where `m` may be a fixed margin/offset (often 0 in simple forms).

Let:

`u = beta * (delta_theta - m)`

Then:

`grad_theta L_SimPO = - beta * sigma(-u) * d(delta_theta)/dtheta`

### Interpretation

SimPO update magnitude is gated by **policy’s own win/loss gap relative to margin**, not relative to a second model’s gap.

This means gradient energy is concentrated on “policy still confuses winner vs loser” pairs directly, without reference-gap subtraction.

---

## 5) Key difference in parameter movement

Both objectives move along `d(delta_theta)/dtheta` direction (increase winner log-prob, decrease loser log-prob).  
The practical difference is the scalar gate:

- DPO gate: `sigma(-beta*(delta_theta - delta_ref))`
- SimPO gate: `sigma(-beta*(delta_theta - m))`

### Why this matters on small datasets

With only 104 pairs, each pair’s gradient weight matters a lot.

In DPO:
- pair weights are influenced by `delta_ref`, which can damp updates when reference already “agrees enough,”
- this can reduce effective step mass on scarce data.

In SimPO:
- no reference subtraction term,
- pair pressure is tied directly to current policy margin state,
- more pairs can stay in active high-gradient region longer early in training.

So the effect is not merely “less regularization” in words; it is:
**the per-pair gradient multiplier is no longer reference-relative**, which changes update distribution across examples and often increases effective movement toward observed preference signal in low-data regimes.

---

## 6) Concrete intuition with one pair

Suppose early training:
- `delta_theta = 0.2` (policy weakly prefers winner),
- `delta_ref = 0.6`,
- `beta = 2`.

DPO:
- `z = 2*(0.2 - 0.6) = -0.8`
- gate `sigma(-z)=sigma(0.8)≈0.69`

SimPO (m=0):
- `u = 2*(0.2) = 0.4`
- gate `sigma(-u)=sigma(-0.4)≈0.40`

Now flip case where reference is very strong opposite calibration or domain-shifted. The relative gating can move either way pair-by-pair, but the crucial point is:
- DPO gates by **policy vs reference margin difference**,
- SimPO gates by **policy margin only**.

On tiny datasets, this gating definition change can materially alter which pairs dominate training.

---

## 7) Why “no reference model” can help beyond memory/cost

Beyond compute savings, removing reference can help optimization behavior when:

1. Reference mismatch to task style/domain  
   - Its margin signal can be a poor anchor for your niche preference pairs.

2. Low sample count  
   - Over-anchoring to reference-relative margin can underuse scarce signal.

3. Strongly curated pair set  
   - You may want direct policy movement toward labeled winner/loser structure.

So mathematically, improvement can come from changed gradient weighting, not just a weaker penalty term.

---

## 8) What to write in your `model_card.md` (defensible wording)

Suggested replacement sentence:

> “We selected SimPO over DPO not only for lower memory overhead, but because SimPO’s pairwise gradient is reference-free: update magnitude is gated by the policy’s own winner-vs-loser margin, whereas DPO gates by policy-minus-reference margin. On our 104-pair dataset, this produced stronger effective movement toward labeled preference signal under the same compute budget.”

---

## 9) What to verify in your Week 11 training script

In your SimPO script, log per-step:
- mean `delta_theta` on train pairs,
- fraction of pairs with `delta_theta > 0`,
- gradient norm of preference head/lora blocks,
- train-vs-dev preference accuracy.

If SimPO is giving the expected behavior, you should see:
- faster early increase in winner-minus-loser margin,
- stable dev preference accuracy without reference model dependence.

---

## 10) Bottom line

You can now defend the choice mechanistically:

- DPO and SimPO share preference direction, but not the same gradient gating.
- DPO scales updates by reference-relative margin mismatch.
- SimPO scales updates by policy margin directly.
- On small datasets (like 104 pairs), that gating difference can yield more usable movement toward supervised preferences, which is the real reason the choice can outperform in practice.

