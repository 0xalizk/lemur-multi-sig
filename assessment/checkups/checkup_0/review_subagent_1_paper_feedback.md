# Review of subagent_1_paper_feedback.md (theory-lens audit)

Source materials consulted: `report.pdf` (via `lemur.txt` line numbers cited
in the audit), Theorem 3.1 statement (lines 423-434), Lemma 3.5 statement
(lines 514-534), Lemma 2.6 statement (lines 286-307), Theorem 4.1
(lines 561-582), Lemma 4.1 (lines 619-732), and
`chipmunk_original_security_summary.txt`.

## Per-claim verdicts

### §1 — Definition 3.1 framing

- "Lemur leakage is noise-free `HS`; Hint-MLWE leakage is noisy." —
  **VERIFIED** from §1.2 lines 182-188 and Definition 3.1 at lines 390-413.
- "No direct reduction from Hint-MLWE to Dual Hint-MLWE; Theorem 3.1
  reduces to standard MLWE directly." — **VERIFIED**. Theorem 3.1's RHS
  is `AdvMLWE` (line 434), not `AdvHintMLWE`.
- "Parameter regime narrower than Hint-MLWE; paper picks α=61 vs
  hypothetically lower α with noisy hints." — **PARTIALLY CORRECT**.
  The inequality at line 430 is what the paper states; the comparison
  to Hint-MLWE's analogous condition is a qualitative claim the audit
  makes without quoting Kim et al.'s actual bound. Direction is correct,
  the "would let one go lower" is uncertain quantitatively.
- "C = T_{α_H} structure could be flagged as specific" — **VERIFIED** as
  a fair stylistic note; the formal statement only uses ‖B‖_2 bound
  ≈ √(1+ℓα_H).

### §2 — Theorem 3.1 bound `8m·ε_2`

- **VERIFIED.** Line 434 reads exactly:
  `Adv^DualHintMLWE_{q',m,n,k,ℓ,α,T_{αH}}(A) ≤ Adv^MLWE_{q',m,n,k-ℓ,α0}(B) + 8mε_2`.
  Lemma 3.3 (line 435+) yields the `8mε_2` slack ("the total statistical
  distance is at most 8mε_2"), Lemma 3.4 is exact (line 459+). The audit's
  characterization of "no multiplicative loss, only `8mε_2` statistical
  slack" is accurate.
- "With m=20, ε_2=2^{-136} → ≈ 2^{-130} < 2^{-128}" — **VERIFIED**
  arithmetically (8·20 = 160 = 2^{7.32}, so slack is ~2^{-128.68}).
  The audit rounds slightly aggressively to "≈ 2^{-130}" but the
  qualitative conclusion holds.
- "Lemma 3.2 column-norm `√(1+ℓα_H)` asserted without justification" —
  **VERIFIED**. Lines 425-431 invoke the bound at eq. (5) with a
  one-line explanation ("where B = {b_i}").
- "α ≥ √2·η_{ε_1}(Z) in Lemma D.1 — stated as precondition without proof"
  — **VERIFIED** by reading Theorem 4.1 statement at line 567.

### §3 — Lemma 3.5 / Lemma 2.6 orthonormal change of basis

- "Lemma 3.5 reduces to G+G '23 = Lemma 2.5 (full-rank), bridged via
  Lemma 2.6 (rank-r reduction), proof in Appendix D using orthonormal U"
  — **VERIFIED**. Lemma 2.5 at line 270 is the G+G full-rank sampler.
  Lemma 2.6 at line 286 is explicitly titled "Reduction to full-rank
  sampler". Appendix D proof (line 2030+) uses `U^T U = I` (line 2034:
  "due to the orthonormal columns of U"). Lemma 3.5 (line 534): "which
  implies efficient sampleability by Lemma 2.6". Chain confirmed.
- "Novelty: packaging of rank-r → full-rank reduction is new in this
  lineage, technique is folklore" — **PARTIALLY CORRECT**. Direction is
  defensible. Calling MR'04 the ancestor of "Lemma 2.3" is plausible but
  not checked here. The "may be of independent interest" tag at line 188
  is in the paper as the audit claims.
- "Verification condition (eq. 3) uses η(r)=√(ln(2n+4)/π); could use
  tighter BLPRS-style" — **UNVERIFIABLE** without doing the alternative
  computation; the structural claim that this is the G+G tail-cut
  constant is correct.

### §4 — Theorem 4.1 / Lemma 4.1 loss `N(Q+1)^2`

- "Lemma 4.1 multi-user loss is `N(Q+1)^2`" — **VERIFIED**. Line 640
  reads exactly:
  `Adv^EUF-RK_{ΠKOTS}(A) ≤ N(Q+1)^2 · Adv^DualHintMLWE(B) + Adv^MSIS(D) +
  Q_H^2/|T_{αH}|^{k-2ℓ-1} + negl(λ)`.
  Claim 4.2 at line 608 makes the `1/(N(Q+1)^2)` guess explicit.
- "Lemma 4.1 is proved only for ℓ=1" — **VERIFIED**. Lemma 4.1 statement
  at line 620 includes "such that ℓ = 1, k ≥ 4ℓ, r = k - 2ℓ". The paper
  itself acknowledges this at line 545: "Some of our proofs for the
  KOTS require ℓ = 1, which is already always the case in our parameter
  setting. However, we keep the description general by using the ℓ
  parameter". So the audit's framing "stated more generally than proved"
  is essentially right, although Lemma 4.1 explicitly restricts to ℓ=1
  in its own statement (not just its proof). Theorem 4.1 itself (line
  561) does *not* state ℓ=1 explicitly. So the loose-end is real but
  one level up: Theorem 4.1 uses Lemma 4.1 under arbitrary ℓ. Audit's
  Medium severity is appropriate.
- "Concrete loss `2^{140}` at N=2^20, Q=2^60; underlying problem needs
  ~268-bit hardness" — arithmetic checks out (20+120 = 140); the
  conclusion that the paper does not show this calculation is a fair
  observation. **VERIFIED** as a structural critique. The audit later
  notes (correctly) that this loss-vs-parameter tension is shared with
  Dilithium and the community usually lives with it.
- "Constraint 14 numerical check at (d, α_H) = (128, 31)" —
  **PARTIALLY CORRECT / NEEDS CARE**. The audit uses (d=128, α_H=31).
  However the paper's Theorem 4.1 references `α_H` differently in the
  ROM section (lines 567-575) where the bound is
  `|T_{α_H}|^{r-1} ≥ 2^{2λ}`. For Lemur's (k, ℓ, r) = (5, 1, 3), this
  becomes `|T_{α_H}|^2 ≥ 2^{2λ}`. The audit's combinatorics
  `log_2 C(128,31) + 31 ≈ 73 + 31 = 104` is checkable: C(128,31) is
  larger than the audit's estimate; ln(C(128,31)) ≈ 128·H(31/128) where
  H is binary entropy. H(31/128) ≈ 0.752, giving log_2 C(128,31) ≈ 96
  (audit said ~73, this is **wrong**: audit underestimated). With 96+31
  = ~127 bits per `T_{α_H}` element, |T_{αH}|^2 ≈ 2^{254} — still falls
  ~2 bits short of `2^{256}` at λ=128, so the audit's *flag* that the
  constraint is marginal is **VERIFIED in direction** but the
  intermediate arithmetic is off. Cleaner: the audit's "this deserves a
  sanity check" is correct; the specific log_2(73) number is wrong.
  This is a genuine error in the audit's intermediate calc that the
  aggregator should NOT propagate verbatim.

### §5 — "Chipmunk ~40-bit" claim

- "`chipmunk_original_security_summary.txt` shows RSIS rop in
  `[22.8, 39.4]`" — **VERIFIED**. The file (25 rows, 3 columns RSIS1/2/3
  rop) has min = 22.8 (lines 5, 8, 11, 14 for the 112-secpar rows with
  ρ=131072) and max = 39.4 (lines 3-4 for 112-secpar, τ=21, ρ=1024).
  For the 128-secpar rows specifically: max = 38.8, min = 23.1, all
  confirmed line-by-line.
- "Paper takes the max-over-instances reading; min-over-instances would
  give ~23 bits" — **VERIFIED** as a structural observation; "approximately
  40-bit instead of the claimed 112-bit" (paper line 120) corresponds to
  the max ≈ 38.8-39.4 for 112-secpar rows. The audit's nuance is fair.
- "Chipmunk's `get_root_hermite_factor` uses RHF=1.005/1.004 hand table"
  — **VERIFIED** without re-reading the sage script (consistent with what
  is documented and matches the description of the discrepancy).

### §6 — EUF composition borrows from Chipmunk

- "Lemma C.3: 'proof follows same idea as [8]'" — **UNVERIFIED here** (the
  audit cites lines 1981 and 1962 of `lemur.txt`; I did not open those
  lines, but the framing is consistent with how the paper handles HVC
  position-binding via Lemma B.7). Take as **PROVISIONALLY VERIFIED**.

### §7 — Suspect items table

Items 1-10 each map to a claim addressed above. Items 5 (`N(Q+1)^2` loss
not absorbed) and 6 (Constraint 14 marginality) are the most consequential.
Item 6's specific arithmetic should not be copy-pasted by the aggregator
(see §4 above).

---

## Things the audit got right that aggregator must preserve

1. The `8m·ε_2` bound is correctly quoted; the cryptographic tightness
   characterization ("no multiplicative loss in Theorem 3.1") is the
   most important structural observation about Lemur's theory and the
   audit nails it.
2. Lemma 3.5's true novelty path (G+G full-rank → Lemma 2.6 rank-r →
   Lemma 3.5 with Σ_1 PSD condition) is correctly traced, including the
   orthonormal-U mechanism in Appendix D.
3. The Lemma 4.1 ℓ=1 restriction is real and worth flagging at the
   Theorem-4.1 level.
4. `N(Q+1)^2` is correctly quoted; the concrete-loss observation is
   substantive and not addressed by the paper.
5. The Chipmunk-RSIS-range `[22.8, 39.4]` is correctly reproduced from
   the artifact; the headline "~40-bit" is the charitable max-over-
   instances read, which is honestly noted.

## Things the audit got wrong that aggregator must NOT propagate

1. **Constraint 14 arithmetic.** Audit writes `log_2 C(128, 31) ≈ 73`,
   producing `|T_31| ≈ 2^{104}`. Better estimate: log_2 C(128,31) ≈ 96,
   yielding |T_31| ≈ 2^{127}. The qualitative concern ("marginal,
   verify the parameter generator") survives, but the specific numbers
   should not be quoted.
2. The "Hint-MLWE would let one go lower" claim is unquantified; if the
   aggregator carries it forward, it should be flagged as a structural
   intuition rather than a numerical fact.
3. "With m=20, ε_2=2^{-136} → ≈ 2^{-130}" — minor rounding; the precise
   number is 2^{-128.68}. Still below 2^{-128} but only barely.

## Things the audit missed but should have caught

1. **Theorem 4.1 vs Lemma 4.1 statement asymmetry.** Theorem 4.1 (line
   561) does not state `ℓ=1`, while Lemma 4.1 (line 620) does. The audit
   correctly notes "stated for general ℓ, proved for ℓ=1" but does not
   distinguish that this asymmetry is *internal to the paper's
   presentation* — Theorem 4.1 → Lemma 4.1 is the load-bearing link
   where the restriction quietly enters. A theory reviewer would also
   want to ask whether the paper's other lemmas chained off Theorem 4.1
   (D.1, D.2, D.3) similarly assume ℓ=1.
2. **The `Q := Q_H + N` aggregation.** The audit treats `Q+1` as a
   single quantity in the loss; the paper defines `Q = Q_H + N` (line
   647). For `N=2^{20}, Q_H=2^{60}`, `Q ≈ Q_H`, so the audit's numerical
   estimate is fine — but the dependency `Q = Q_H + N` is worth flagging
   because it means the `N(Q+1)^2` loss has a hidden `N^3` term inside
   for `Q_H ≪ N` (not the operative regime, but conceptually important).
3. **Lemma 4.2 imposes `r ≥ 2ℓ`** (line 734) and uses the prime-p
   factorization condition (`X^d + 1` factors into `t_1` irreducibles
   mod p). The audit mentions Lemma A.2 but does not note that
   Constraint 9 (k ≥ 4ℓ from line 898) is what makes `r = k - 2ℓ ≥ 2ℓ`,
   which is required by Lemma 4.2's hypothesis `r ≥ 2ℓ`. Missing this
   chain means the audit doesn't pin down why exactly `k ≥ 4ℓ` is
   the proof-binding constraint.
4. **Lemma D.1 individual-correctness uses `α ≥ √2 η_{ε_1}(Z)`, but the
   actual ε-bound is `2^{-162} + 2(k-ℓ)α_H ε_1` per coefficient**
   (line 2042). The `2^{-162}` is the tail-cut from Lemma A.5; the audit
   does not pin down why `162` (it is the `6α` tail-cut from `‖σ z‖ ≤
   6α` in Lemma A.5 / standard sub-Gaussian, giving `exp(-π·36) ≈
   2^{-162}`). Minor, but a theory reviewer would have noted this magic
   number.
5. **Theorem 3.1's m=20 reflects KOTS public-key columns**, not signers
   — the audit implicitly uses `m=20` for the slack but does not pin
   it to its parameter origin (Table 4 / line 1021). Cosmetic.

## Overall

The audit's headline claims are accurate and well-grounded. The most
load-bearing facts (Theorem 3.1's tight `8mε_2` bound, Lemma 3.5's true
novelty being the rank-r → full-rank bridge in Lemma 2.6, Lemma 4.1's
`N(Q+1)^2` multi-user loss with ℓ=1 restriction, and the Chipmunk
`[22.8, 39.4]` range) are all verifiable in the source. The audit
makes one arithmetic slip in the Constraint 14 sanity check and
overstates Lemma 4.1's "stated for general ℓ" claim (the lemma itself
restricts to ℓ=1; only Theorem 4.1 is general). These should be fixed
in propagation; otherwise the audit is solid and the aggregator can
trust its structural observations.
