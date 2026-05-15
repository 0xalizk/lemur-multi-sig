# Open issues — gaps the reviewers flagged but the audits/fact-checks didn't pursue

This file aggregates "Things the audit missed", "Things FC missed",
and "Missed angles" sections from the reviewer files of both rounds:

- `checkup_0/` — original audit round (3 lens-specific audits + 3 reviewers).
- `checkup_1/` — fact-check round (3 fact-checkers + 3 reviewers).

These are **higher-level issues** that need investigation, not atomic
fact checks. (Atomic UNVERIFIABLE items live in `unresolved.md`.)

Each item below was flagged by exactly one reviewer; the original audit
or fact-checker didn't pursue it. Most are not load-bearing for
review.md's central claims, but they're the natural starting points
for a third audit round if anyone wants one.

---

## A. Theory-lens gaps (from checkup_0/review_subagent_1)

### A1. Theorem 4.1 vs Lemma 4.1 ℓ-restriction asymmetry
Theorem 4.1 (PDF line 561) does not state ℓ=1; Lemma 4.1 (PDF line 620) does. The audit noted "stated for general ℓ, proved for ℓ=1" but didn't pin this to the load-bearing link inside the paper's presentation. A theory reviewer would also want to ask whether Lemmas D.1, D.2, D.3 (which chain off Theorem 4.1) similarly assume ℓ=1.

### A2. The `Q := Q_H + N` aggregation
The audit treats `Q+1` as a single quantity in the `N(Q+1)²` loss. The paper defines `Q = Q_H + N` (PDF line 647). For `N=2²⁰, Q_H=2⁶⁰`, `Q ≈ Q_H`, so the audit's numerical estimate is fine — but the dependency means the loss has a hidden `N³` term when `Q_H ≪ N`. Not the operative regime, conceptually important.

### A3. Constraint 9's role as the proof-binding parameter constraint
Lemma 4.2 imposes `r ≥ 2ℓ` (PDF line 734) and uses the prime-p factorization condition. The audit mentions Lemma A.2 but doesn't note that Constraint 9 (`k ≥ 4ℓ`, line 898) is precisely what makes `r = k - 2ℓ ≥ 2ℓ`. Missing this chain leaves the audit short of explaining *why* `k ≥ 4ℓ` is the proof-binding constraint.

### A4. The `2⁻¹⁶²` tail-cut magic number in Lemma D.1
Individual-correctness uses `α ≥ √2·η_{ε₁}(Z)`, but the per-coefficient ε-bound `2⁻¹⁶² + 2(k-ℓ)α_H ε₁` (PDF line 2042) is unexplained. `2⁻¹⁶²` is the `6α` sub-Gaussian tail-cut from Lemma A.5 (i.e. `exp(-π·36) ≈ 2⁻¹⁶²`). Cosmetic, but a theory reviewer would have noted the magic number.

### A5. Theorem 3.1's m=20 origin
The audit's slack arithmetic uses `m=20` for the d=128 illustration without pinning it to Table 4 / PDF line 1021 where the value enters. Cosmetic. (review.md §3 now distinguishes the d=128 illustration from the shipped d=256 case.)

---

## B. Implementation-lens gaps (from checkup_0/review_subagent_2)

### B1. BDS state 134.4 KB rounding
Audit (and review.md) reports the BDS state at exactly 134.4 KB. The raw figure is `137600 B / 1024 = 134.375 KB`; the printed "134.4 KB" rounds half-up. Not a discrepancy, but worth flagging if anyone re-derives from the raw byte count.

### B2. `bench_breakdown` 1.69× rayon scaling — measurement provenance
Audit cites a 1.69× rayon-scaling number from `bench_breakdown`. It's unclear whether this was measured in the current session or quoted from an earlier review. If measured, the source line in the bench output should be cited; if inferred, it should be labeled.

### B3. `tree_sign_matches_seed_path_every_slot` test scope
The integration test confirms BDS-path-equals-seed-path *at small τ* (the test files use τ ∈ {4, 8} for runtime reasons). The audit cites the test as evidence of correctness but doesn't note that the τ=20 production path is not exercised end-to-end. The test's value is to confirm the *algorithm* matches, not the production parameters.

### B4. Aggregation 567 ms not reproduced
Paper Table 2's `N=2¹⁰` aggregation cell (567 ms) is the most reproducible timing claim — the only measured aggregation row; larger-N cells are linear extrapolations. The implementation audit cancelled `bench --fast` before this number printed. A reviewer wanting "is aggregation seconds or minutes?" at this cell still doesn't have a local measurement. (review.md §6 does report a measured 1.41 s on 11 threads — but for the same row, via a different run that wasn't tracked in the audit.)

---

## C. Field/comparison gaps (from checkup_0/review_subagent_3)

### C1. No runtime comparison vs Chipmunk, not just no size comparison
The paper itself acknowledges (§1.1, PDF line 137-139) "we are unable to provide a meaningful runtime comparison against Chipmunk." The field-lens audit focused on size-comparison fairness but didn't elevate the *absence of timing comparison* as a structural gap. A reviewer asking "how fast does Lemur verify at scale vs Chipmunk?" gets no answer. (review.md §10 now flags this — but it was the reviewer's catch, not the audit's.)

### C2. EUF-RK vs standard model for Anada et al.
The audit calls Anada et al. (ICISC 2024) a "standard-model variant" of Lemur's setting. But the audit didn't check whether Anada targets EUF-RK specifically or a weaker model. If weaker, the "direct sibling" framing is misleading — Anada might not actually be in the same column. Worth a paper-side fetch.

### C3. The RHF ≤ 1.0045 threshold itself
The paper sets RHF ≤ 1.0045 and notes Chipmunk scripts "could not find feasible parameters" at this RHF. The audit accepts the threshold as given. A hostile reviewer would ask: is the threshold itself a choice that disadvantages Chipmunk? If RHF ≤ 1.005 had been picked (Chipmunk's natural target), would Chipmunk re-fit cleanly under the modern estimator? This is the only place the "Lemur is 2× smaller than Chipmunk" comparison could be argued away.

### C4. Boudgoust-Takahashi '23 sequential half-aggregation
Mentioned in audit §6.4 without follow-through. Either a missing related-work item (in which case it deserves a sentence in review.md §9) or dead weight (in which case it should be cut from the audit). Currently neither.

### C5. Dual Hint-MLWE motivation outside Lemur
The audit doesn't ask whether Dual Hint-MLWE is a well-motivated assumption *outside* its use in Lemur. Is it a generic primitive other schemes might want, or an ad-hoc construction tailored to this proof? Spot-checking Theorem 3.1 the reduction to standard MLWE looks rigorous; the assumption is "real" mathematically. But a field-placement audit should at least flag the question.

### C6. Drake et al. timing comparison self-undercut
Paper Table 2 claims aggregation in **~12 min at N=2²⁰** for Lemur. The audit dismissed Drake et al.'s hash+SNARK paradigm as "seconds to minutes per aggregation" — but Lemur is in that same range at the realistic deployment scale. The audit didn't note the self-undercut. (Discussed in review.md §10, but as a reviewer add, not an audit catch.)

---

## D. Skills-usability gaps (from checkup_0/review_subagent_{1,3}_skills_feedback)

### D1. `lemur.txt` workflow not documented in the skills
The audits all relied on `pdftotext -layout report.pdf | grep` to find line numbers. The skills now document this in their Setup section, but during the audit round neither skill mentioned the workflow.

### D2. Tables 4 and 5 (parameter-constraint tables) absent from skills
Theorem 4.1's preconditions correspond to numbered constraints in paper Tables 4–5. A theory user should be pointed at the constraint table, not have to re-derive what is required of the parameters.

### D3. `parameter/lemur_param.sage` is the source of truth — not flagged in skills
For a theory user checking that picked parameters satisfy theorem preconditions, the Sage script is the load-bearing artifact. The skills mention it in the parameter-regeneration flow but don't call it out as the *re-derivation entry point* for the proof-side numerical inequalities (eq. 4, 5; α=61 derivation in §7).

### D4. Skills' "Files this skill uses" sections referenced ephemeral paths
At the time of the original audit, skill files cited `/workspace/tmp/notes-related-work.md` and similar workspace-relative paths. This was caught in checkup_0 and partially fixed when the skills were converted to repo-relative paths.

### D5. "Triage by submission date" missing from prior-work skill
Catching post-submission supersession is more general than "check citation horizon"; it's also useful when reading any paper dated months or years before the present (e.g. "is this paper still current as of 2026?"). The skill's Step 2.5 handles "could-have-cited-in-revision" vs "post-CR" but doesn't generalise to ongoing currency checks.

---

## E. Fact-check-round-specific reviewer notes (from checkup_1/review_fact_checker_3)

### E1. FC3 did not WebSearch its asserted ePrint numbers
The fact-checker accepted ePrint numbers from `tmp/notes-related-work.md` without re-WebSearching them. Three out of six were spot-checked by reviewer R3 (Squirrel 2022/694, Hint-MLWE 2023/623, Drake 2025/055 all confirmed); the other three (Chipmunk 2023/1820, Aardal 2024/311, LeanSig 2025/1332) were left on FC3's say-so. Confidence is high but not 100%.

### E2. FC3 didn't check whether `bench_verify` actually accepts the flags the reproduction recipe quotes
The recipe says `--zero-fixture --n 1048576 --reps 1`. Reviewer R3 opened `bench_verify.rs:98-149` and confirmed these flag names. Still, a "did anyone actually run this exact command" check is open.

---

## Triage suggestion

By severity:

- **Items worth pursuing for accuracy of review.md:**
  C1 (timing comparison absence — already partly in review.md §10),
  C2 (Anada EUF-RK check — would change the §9.1 "direct sibling" framing if Anada is weaker),
  C3 (RHF threshold choice — the only argument against the "2× smaller" headline),
  C6 (Drake-vs-Lemur timing self-undercut — already partly in review.md §10).

- **Items worth pursuing for skill quality:**
  D2 (point at Tables 4–5 for proof-side preconditions),
  D3 (call out `lemur_param.sage` as the re-derivation entry point).

- **Items worth pursuing only if doing a third audit round:**
  A1–A5 (theory micro-gaps),
  B1 (rounding nit),
  B2 (provenance of one bench number),
  C4 (Boudgoust-Takahashi follow-up).

- **Already resolved or cosmetic:**
  B3 (test-scope caveat could be added but doesn't break anything),
  B4 (the 567 ms cell was eventually measured on 11 threads),
  D1 / D4 / E1 / E2 (already fixed or low-stakes).

Total: ~20 open issues, of which 6 would meaningfully improve review.md's
field-placement section and 2 would improve the skills.
