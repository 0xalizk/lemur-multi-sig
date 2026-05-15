# Review of `subagent_2_skills_feedback.md`

Scope: per-proposed-edit verdict on each skill change, plus gaps the
audit missed and any proposed edits that would degrade the skills.

The two skills under audit:
- `/workspace/.claude/skills/lemur-paper-analysis/SKILL.md`
- `/workspace/.claude/skills/lemur-prior-work-survey/SKILL.md`

---

## Section 1 — `lemur-paper-analysis` proposed edits

### E1. Add to Step 1: "§7.1 worked example is *not* the implementation cell"

**KEEP.** This is the highest-value addition the audit proposes. Verified
independently: §7.1 walks `(d=128, N=2²⁰, τ=24, α_w=31, k=5, r=3, α=61,
β_z=8185)`; the implementation in `lemur-py/profiles.py:106-113` and
`lemur-rs/src/profile.rs:300-341` is `(d=256, k=4, τ=20, N=1024, α=87,
β_z=14046, α_w=23)`. No overlap on any parameter. The proposed paragraph
correctly cites `parameter/summary.txt` line 11 and the Python profile
location. **Recommend incorporating verbatim.**

Minor tightening: the proposed text says "see `parameter/summary.txt`
line 11, also baked into `lemur-py/profiles.py:106-113`" — the Rust
counterpart `lemur-rs/src/profile.rs:300` should also be cited so a
reader doing Rust-side audit lands on the right line.

### E2. Add to Step 3: 4th category — "Encoding-noise size claims"

**KEEP.** Verified Table 5 reports worst-case bounds (239/331/457 KB
at τ=20) and Table 2 reports Rice-encoded (201/284/394 KB at the same
cell), differing by ~14%. The skill's current Step 3 has no explicit
discussion of two-column size reporting; the proposed addition fills
a real gap. **Recommend incorporating.**

One refinement the audit missed: the 14% gap is *averaged* over the
three N values; per-cell it is 15.8 / 14.4 / 13.7 % (as the audit's
own §1.2 computes). The skill addition could say "~14% (15.8/14.4/13.7%
across N=2^{10,15,20})" for precision.

### E3. Add to Step 1 "Understand the engineering": one-liner on §7.2

**KEEP, BUT TRIM.** The proposed insertion ("§7.2 is essential — it is
the only place the 'two interoperable implementations' and the
'byte-equivalent test vectors' claim is explicit. The CLAUDE.md 'four
checkpoints' list is the empirical version of §7.2's correctness
story.") is accurate (verified §7.2 contains the "fully interoperable
open-source reference implementations" and "cross-validated against
shared byte-level test vectors" language, pdf lines 1011-1014).

But it pushes a forward-reference to CLAUDE.md content that the skill
explicitly disclaims ("Implementation-side guidance lives in the
project's CLAUDE.md", line 8). Better wording:

> §7.2 is the only place the "two interoperable implementations"
> claim and the "byte-equivalent test vectors" claim are made
> explicitly. Reference it whenever a reviewer asks how the
> correctness story is empirically grounded.

### E4. Add to Step 5: "Same parameter cell?" bullet

**KEEP.** The proposed bullet correctly captures the
"shipped-cell-vs-extrapolated-cell" gotcha. Verified: `bench --fast`
runs at N ∈ {1024, 8192} (bench.rs:616 per the audit's claim, which I
did not independently re-check but is consistent with code/README.md);
larger-N sizes go through `rice_sizes.py`. The bullet correctly
distinguishes "measured N=1024" from "extrapolated N=2²⁰".
**Recommend incorporating verbatim.**

### E5. New (proposed in summary table) Step 3.5: bench time budgets

**KEEP, BUT NEEDS DISCIPLINE.** The skill explicitly disclaims
implementation guidance ("implementation, toolchain, ... and
benchmarking guidance are deliberately *not* here"). A "Step 3.5 — bench
time budgets" violates that contract. The right home is CLAUDE.md, not
the skill.

That said, a one-line **pointer** in Step 3 of the skill saying
"specific benchmark timings and budget guidance are in CLAUDE.md;
defining factor is that `bench --fast` keygen alone is ~2 min on
11 threads — do not cancel before 30 min" would help without bloating
the skill. The audit's prose budget list (`lemur sizes` — instant;
`cargo test --release` — 1 min; etc.) belongs in CLAUDE.md, where the
audit's section 3.3 in the skills-feedback file says it currently
lives.

**Modified verdict: route the budget table to CLAUDE.md, not the skill.**
The skill should only carry the one-line "don't cancel before 30 min"
discipline.

---

## Section 2 — `lemur-prior-work-survey` proposed edits

### E6. Step 1: "this skill is over-scoped for empirical verification"

**KEEP.** The proposed paragraph is accurate. The prior-work skill is
genuinely the wrong tool for code-audit / size-reproduction tasks. The
proposed paragraph correctly redirects to `lemur sizes`,
`parameter/rice_sizes.py`, and the byte-equivalence vectors test.

One refinement: the audit's phrase "skip Steps 5–6 and go straight to
the comparison column" is too prescriptive — Step 4 (the KOTS+HVC
blueprint) is useful even for code audits (the audit itself uses it in
§3 of paper-feedback to triage files). Better:

> If your task is **empirical verification**, only Steps 2 and 4
> (KOTS+HVC blueprint) are load-bearing. Steps 5-6 (abstract
> reading, paradigm comparisons) are over-scoped — Chipmunk's
> numerical comparison is reproduced inside the Lemur PDF
> (Table 5), so external Chipmunk-paper fetches are unnecessary.

### E7. Step 5: estimator-output > Chipmunk abstract

**KEEP.** Verified
`parameter/chipmunk_original_security_summary.txt` exists on the
container and contains rop columns that, per the audit's §5.5, sit in
26-39 bits for `λ_claim=128`. The skill currently mentions this file
only in the "Files this skill uses" list at the bottom; promoting it to
Step 5 (where the security recomputation narrative actually lives) is
the right move. **Recommend incorporating.**

### E8. Step 7 #1: add the 26-39 bit rop range

**KEEP.** Currently the skill says "Chipmunk's published 112-bit
parameters actually realize ~40 bits under modern estimation." The
proposed enrichment with "RSIS rop in 26-39 range" and a concrete
cell ("38.8-bit rop for (τ=21, N=1024, λ_claimed=128)") is more
specific and verifiable. **Recommend incorporating, with one factual
caveat:** the audit cites `38.8 / 38.8 / 29.8 bits` for the
`τ=21, N=1024, λ_claim=128` row. I did not independently re-read
`chipmunk_original_security_summary.txt` row-by-row; I trust the
audit's citation here since it has been internally consistent on
every other source claim, but the editor should re-verify the row
exists.

### E9. Step 8: "When *not* to apply this skill" paragraph

**KEEP.** Cleanly delimits the skill's scope. The three negative-
scope bullets (code audit / Table reproduction / re-running estimator)
are exactly right for the prior-work skill, which is explicitly for
"surveying the field for a related-work paragraph".

---

## Section 3 — Cross-cutting (§3 of audit)

### E10. §3.1 — Rice-encoding gap

**REDUNDANT WITH E2.** The audit already proposes adding this to Step 3
of paper-analysis (as E2). Restating it in the cross-cutting section
adds nothing; recommend dropping E10 and keeping E2.

### E11. §3.2 — "What the artifact doesn't ship"

**KEEP.** The "binaries only run at N=1024; larger-N rows are
extrapolations" message is the single biggest reviewer trap. The audit
already proposes this in E4 (Step 5 of paper-analysis) but the
cross-cutting note adds value by emphasizing it should be in **Step 3**
(the trap-claims-to-scrutinize section), not only in Step 5
(scheme-comparison). Recommend incorporating into Step 3 as well:

> Size-claim trap: `lemur sizes` reproduces only the N=1024 cell. The
> N∈{2^15, 2^20} numbers in Table 2 are derived from formulas in
> `rice_sizes.py`. They are reproducible deterministically (the
> formulas are exact) but no full pipeline runs at those N values
> in either implementation.

### E12. §3.3 — Bench-time guidance

**SEE E5.** Same verdict: route to CLAUDE.md, not the skill. The audit's
"do not cancel before 30 min" line is the only sentence worth keeping
in the skill itself.

### E13. §3.4 — Sage availability one-liner

**KEEP.** Both skills currently list `*.sage` files without noting that
the typical container lacks SageMath. A single sentence in each skill's
"Files this skill uses" section ("SageMath is rarely installed; the
committed `.txt` summaries are the practical source of truth") is a
low-cost, high-utility addition. Recommend incorporating.

---

## Section 4 — Missed edits (audit didn't propose, should have)

### M1. Notation trap: `secpar` vs `d` in summary.txt

`parameter/summary.txt`'s first column is `secpar` (security parameter,
always 128) and the *fourth* column is `d` (ring dimension, always 256
in the shipped family). A reader scanning the file may see `128` in
column 1 and conflate it with `d=128`. The paper-analysis skill's Step
4 (notation traps) should add:

> `secpar` (column 1 of `summary.txt`) is the *security parameter*
> λ, not the ring dimension `d`. Both happen to be small powers of
> 2 (λ=128, d=256). Eyeballing the file without reading the header
> can produce a wrong `d=128` inference.

The audit's paper-feedback file does not propose this even though it
correctly identifies the same trap in passing.

### M2. The Z_agg vs Z size distinction

The audit's own paper-feedback file confuses Z (individual KOTS sig)
with Z_agg (aggregated KOTS sig) in one place (see review of
paper-feedback §7). The paper-analysis skill's Step 3 trap-claims
section could call this out:

> The individual KOTS sig Z is ~4.2 KB; the aggregated Z_agg is
> ~5.6 KB. Distinct objects with similar names. When matching
> `lemur sizes` against the paper, line up "individual sig" with
> Z and "aggregated sig" with Z_agg — and note that the aggregated
> sig also includes the Babai path and HVC siblings.

### M3. β_σ vs β_z

Three β quantities live in the codebase: `beta_z` (KOTS individual),
`beta_sigma` (KOTS aggregated), and `2*beta_sigma` (weak-vrfy bound used
in proofs). The Step 4 notation-trap section could note these alongside
the "three α's" entry. Currently it has no β coverage at all.

### M4. The aggregation γ=10 attempt counter

The skill doesn't mention that aggregation retries up to γ=10 times,
each time mixing the attempt number into the randomizer hash. This is
worth one line in Step 3 because it's how the paper's "ε_hom^γ ≪ 2^{-λ}"
correctness argument actually unfolds in code.

### M5. The Rust CRT vs Python schoolbook divergence

The skill mentions that Rust uses Montgomery NTT for HVC and CRT-via-aux-
primes for KOTS, but does not flag that the **Python reference uses
schoolbook for `q'`** while the **Rust uses CRT**, and yet they agree
byte-for-byte on `Z`. That's a non-obvious correctness property: the
schoolbook and CRT implementations of the same KOTS multiplication
agree on signed Z coefficients. Worth one line in Step 7 of
paper-analysis (or Step 2 KOTS+HVC blueprint) for any future reviewer
auditing the Rust port.

---

## Section 5 — Proposed edits that would degrade the skills

### B1. Step 3.5 bench-time table (E5)

As discussed, the paper-analysis skill explicitly says implementation
guidance lives in CLAUDE.md. Adding a full bench-cost table re-imports
implementation content. **Move to CLAUDE.md**, keep only the
"don't cancel before 30 min" line in the skill.

### B2. "Skip Steps 5-6" in prior-work skill (E6 as written)

The audit's text "Skip Steps 5–6 and go straight to the comparison
column" is too aggressive — Step 4 (KOTS+HVC blueprint) is essential
even for empirical audits. The right framing is "Steps 5-6 are
optional for empirical audits", not "skip them".

### B3. Promoting prior-work skill to be a code-audit skill

If the audit's E9 ("When not to apply this skill") gets read as a
recommendation to expand prior-work to cover code audits, that would
damage both skills. Keep the prior-work skill scoped to its name. The
"when not to apply" paragraph is fine; do not let it bleed into a
"how to apply" paragraph for the wrong tasks.

---

## Section 6 — Audit's own scope errors

### S1. Audit critiques `lemur-prior-work-survey` for being over-scoped for empirical verification, but the audit was *told to read it* by the user

The audit (skills-feedback §2 opening: "I read it because the user
asked me to") acknowledges this. The criticism is still useful — the
skill is genuinely the wrong tool for empirical audits and should say
so explicitly. But the audit could be clearer that this is a
"document the scope boundary" recommendation, not a "the skill is
broken" one. The audit's E9 does land the right framing.

### S2. The audit cites "CLAUDE.md says X" multiple times but `/workspace/CLAUDE.md` is not directly verified

The audit references CLAUDE.md claims (e.g. "CLAUDE.md predicts paper's
24-thread timings drop to ~2.5× longer on 11 threads", "Online Sign 347
µs vs Stateful Sign 4.1 ms ≈ 12× ratio"). I did not re-read CLAUDE.md
in this verification. If a follow-up review checks those CLAUDE.md
quotes line-by-line, it may find the audit's quotations are
approximate. Not a blocking issue but worth flagging.

---

## Summary of verdicts

| Audit edit | Section | Skill | Verdict | Notes |
| --- | --- | --- | --- | --- |
| E1 §7.1 ≠ implementation cell | Step 1 | paper-analysis | **KEEP** | Add Rust profile path too |
| E2 Encoding-noise size category | Step 3 | paper-analysis | **KEEP** | Add per-cell numbers |
| E3 §7.2 one-liner | Step 1 | paper-analysis | **KEEP (trim)** | Don't import CLAUDE.md content |
| E4 "Same parameter cell?" bullet | Step 5 | paper-analysis | **KEEP** | As-is |
| E5 Bench time-budget Step 3.5 | new | paper-analysis | **REJECT (mostly)** | Route to CLAUDE.md; keep only "30 min" line |
| E6 "Over-scoped for empirical" | Step 1 | prior-work | **KEEP (soften)** | Don't say "skip 5-6" outright |
| E7 Estimator output > Chipmunk abstract | Step 5 | prior-work | **KEEP** | As-is |
| E8 Add 26-39 bit rop range | Step 7 #1 | prior-work | **KEEP** | Re-verify the cell citation |
| E9 "When not to apply this skill" | Step 8 | prior-work | **KEEP** | As-is |
| E10 Rice gap in cross-cutting | — | both | **DROP** | Redundant with E2 |
| E11 "Artifact doesn't ship N>1024" | Step 3 | paper-analysis | **KEEP** | Strengthens E4 |
| E12 Bench guidance cross-cutting | — | both | **REJECT** | Same as E5 |
| E13 Sage one-liner | Files used | both | **KEEP** | As-is |

Missed:
- M1 (secpar vs d column)
- M2 (Z vs Z_agg distinction)
- M3 (three β's in Step 4)
- M4 (γ=10 retry mechanic)
- M5 (Rust CRT vs Python schoolbook agreement)

Net: most proposed edits are sound and verifiable. The two that should
be rejected (E5, E12) both push implementation/bench content into the
paper-analysis skill, violating its explicit scope. The biggest single
edit worth keeping is E1 (the §7.1 worked-example gap), which fills a
real reviewer trap.
