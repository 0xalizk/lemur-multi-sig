# Review of `subagent_2_paper_feedback.md`

Scope: independently re-verify factual claims, code citations, and arithmetic
in the audit. Verdicts per spot-check, then what was right, what was wrong,
what was missed.

---

## Per-claim verdicts

### C1. `lemur sizes` reproduces Table 2 / Table 5 N=2^10 row

**VERIFIED.** Ran `./target/release/lemur sizes`. Output: `aggregated sig
(N=1024, ~201.2 KB)  206065  (201.2 KB)`. Matches paper Table 2 (201 KB
Lemur Multi-Sig at τ=20, N=2^10) and Table 5 (239 KB worst-case total at
the same cell — exactly the audit's "239 KB bound (Table 5)" referent).

But: the **component breakdown the prompt cites** (`Z 4.2 KB + path
50.8 KB + sib 116.8 KB + u 28.0 KB = 201.2 KB`) is **WRONG ARITHMETICALLY**:
4.2 + 50.8 + 116.8 + 28.0 = 199.8, not 201.2. The actual `lemur sizes`
output reports `Z_agg 5.6 KB + path 50.8 KB + sib 116.8 KB + u 28.0 KB
= 201.2 KB` (Z_agg is 5761 B = 5.6 KB, *not* 4.2 KB; 4.2 KB is the
non-aggregated individual KOTS sig Z). The audit's own §1.1 table is
silent on the aggregated breakdown (just gives the total) but its §7
"CLAUDE.md verification" paragraph repeats the incorrect breakdown
("Z 4.2 KB + path 50.8 KB + sib 116.8 KB + u 28.0 KB = 199.8 KB before
the 1-byte attempt tag, totaling 201.2 KB"). That's a transcription error
— the individual KOTS Z (4.2 KB) is being confused with the aggregated
Z_agg (5.6 KB), and there is no "1-byte attempt tag" reconciling the
1.4 KB gap.

**Corrected breakdown:** `Z_agg 5761 B + path 52000 B + sib 119600 B
+ u 28704 B = 206065 B = 201.2 KB`. Aggregated sig contains Z_agg
(at bit-width tuned to β_agg), not the original Z.

### C2. `parameter/rice_sizes.py` reproduces 201.2 / 283.5 / 394.4 KB

**VERIFIED.** Re-ran. Output matches audit exactly:
```
20,1024,239,201.2
20,32768,331,283.5
20,1048576,457,394.4
```
The worst-case 239 / 331 / 457 numbers match Table 5 (τ=20 row, last
column, d=256 k=4 family); the Rice-encoded 201.2 / 283.5 / 394.4 match
Table 2 (Lemur Multi-Sig column).

### C3. 52 tests passing in `cargo test --release`

**VERIFIED** by source-tally (not re-running). Integration:
`bds_stateful.rs:8`, `gauss_ctx.rs:5`, `materialized_tree.rs:5`,
`profile_pipeline.rs:4`, `robustness.rs:5` = 27. Unit:
`src/{kots,ntt,profile,aux_ntt,hvc,codec}.rs` = 1+4+3+11+5+1 = 25.
Total 52. Matches the audit.

### C4. `kots.py:_build_H` line 95 has `H[i,i,0]=1`

**VERIFIED.** Line 95 reads exactly `H[i, i, 0] = 1`. The constant-1
polynomial encoding (not `1 + x + ... + x^{d-1}`) is correct per the
Fig. 3 identity-block definition.

### C5. `kots.py:vrfy` lines 163-170 have the three β bounds

**PARTIALLY CORRECT (mis-labeled but content right).** Lines 163-170
are the three *wrappers* `ivrfy / svrfy / wvrfy` calling `self.vrfy`
with `beta_z`, `beta_sigma`, and `2*beta_sigma` respectively. The
underlying `vrfy(...)` itself is at line 154. The audit's text in §3.2b
correctly identifies these as wrappers ("vrfy(β) and its three wrappers
(lines 163–170)"), so the content is right. The prompt's phrasing
"vrfy lines 163–170 has the three β bounds" is technically the wrappers
calling vrfy, not vrfy itself.

### C6. `hvc.py:_internal_label` lines 408-412 reduces mod `self.q`

**VERIFIED.** Lines 408-412 read:
```python
def _internal_label(self, left, right, A0, A1):
    contrib = (
        self.ring.mat_vec(A0, left) + self.ring.mat_vec(A1, right)
    ) % self.q
```
Reduction is mod `self.q` (HVC modulus), not `self.q_prime`. The
CLAUDE.md foot-gun is absent. The audit's slight syntactic claim that
the reduction is on one line is fine — the modulo lives at line 411.

### C7. `lemur.py:aggregate` line 294 breaks only via `avrfy`

**VERIFIED.** Read lines 264-297. Line 294 is exactly
`if self.avrfy(pp, pks, t, m, (Z_agg, d_agg, attempt)): return ...`.
There is no norm-only short-circuit; the loop body always falls through
to `avrfy`, which in turn delegates to `kots.svrfy` (algebraic identity +
norm). No drift.

### C8. §7.1 worked example `(d=128, N=2²⁰, τ=24, α=61, β_z=8185)` ≠ implementation cell `(d=256, k=4, τ=20, N=1024, α=87, β_z=14046)`

**VERIFIED.** §7.1 (pdf line 933) reads: "we describe the parameter-
selection process for a representative setting d = 128, N = 2²⁰, and
τ = 24". It then walks `α_w=31` (Constraint 18 at d=128), `k=5, r=3,
α_H=31, α=61, β_z=8185`. Table 5 caption (pdf line 994) confirms
implementation: "Lemur uses the fixed d=256, k=4 parameter set". Python
profile (`lemur-py/profiles.py:106-113`) and Rust profile
(`lemur-rs/src/profile.rs:300-341`) both encode `d=256, k=4, ℓ=1, m=9,
n=4, α=87, α_H=60, α_w=23, β_z=14046`. Every worked-example value
differs from the shipped implementation.

### C9. `summary.txt` does not contain a (d=128) family

**VERIFIED with one nuance.** The `d` column of `summary.txt` is 256
in every one of its 16 rows. The audit phrasing "summary.txt ships 16
rows: 4 values of τ × 4 values of N, all at (d=256, k=4)" is exactly
right. The single subtle point worth flagging: the *first* column
`secpar` is **128** in every row (that is the security level, λ=128 bits),
not the ring dimension. A careless reader could see `128` in column 1 and
mistake it for d=128. The audit does not make this error.

### C10. Batch-verify 80.8 ms vs paper 30.1 ms ≈ 2.68× slowdown

**ARITHMETIC VERIFIED.** 80.8 / 30.1 = 2.684. CLAUDE.md predicts
"~2.5× longer on 11 threads"; observed 2.68× is within 8% of that
prediction. Naive thread-count scaling would be 24/11 = 2.18×, and the
gap (2.68 vs 2.18) is reasonable for memory-bandwidth and Rayon
overhead. The claim is internally consistent and matches the prior-review
prediction. I cannot re-verify the 80.8 ms measurement on this container
without rerunning, but the ratio argument is sound.

### C11. Online Sign time not reported in paper

**VERIFIED.** Extracted paper text shows Table 2 lists only:
- Signing (BDS08): 4.1 ms
- Signing (Tree-in-memory): 2 ms
- Aggregation: 567 ms / ≈23 s† / ≈12 min†
- Batch verification: 30.1 ms / 812 ms§ / 25.3 s§

No "Online Sign" row. The 303.8 µs "Online Sign" figure exists only in
the `bench` binary's output, not in the paper. The audit's §5.4 caveat
that CLAUDE.md's "Online Sign : Stateful Sign : Full Sign ≈ 1:12:10⁵"
is internal-diagnostic, not paper-stated, is correctly flagged.

### C12. `/workspace/assess/review.md` doesn't exist

**VERIFIED.** `/workspace/assess/` directory does not exist on this
container. The audit's no-parroting claim is accurate.

---

## Things the audit got right (preserve in aggregation)

1. **Worked-example vs implementation cell gap** (§5.1, §5.2). This is
   the biggest pedagogical landmine in the paper and the audit correctly
   identifies it as the readability gap that consumes ~30 min of a
   reviewer's time. The (d=128, α=61) vs (d=256, α=87) divergence is
   real and worth flagging in any aggregated review.

2. **Rice-encoded vs worst-case size distinction** (§5.3 and §6.4).
   Table 2 reports Rice-encoded (201.2, 283.5, 394.4 KB); Table 5
   reports worst-case (239, 331, 457 KB). Different encodings of the
   same parameter cell. Verified by inspection of `rice_sizes.py`
   output and paper Table 5.

3. **Extrapolation footnote interpretation** (§4.3, §6.1, §6.2). Table 2's
   "linear extrapolation from N=8192" footnote (the † marker on
   Aggregation columns ≥ 2^15) and the zero-fixture batch-verify (the §
   marker) are correctly distinguished from measured rows. Verifies via
   the paper Table 2 caption I read.

4. **Four code checkpoints** (§3.2 a-d). Each independently verified above
   (C4, C5, C6, C7). The audit's line numbers are accurate.

5. **Python↔Rust byte-equivalence** (§2). The 10-byte gap from the
   `"implementation"` wrapper key is a sensible explanation; the audit
   correctly distinguishes it from a cryptographic-payload difference.
   (Not re-run here, but the methodology — parse-JSON-then-compare-named-
   keys — is the right approach.)

6. **52 / 0 / 0 test tally with file-by-file breakdown** (§1.3). All
   five integration files identified; counts correct; total 52 confirmed
   by source grep.

7. **Chipmunk 40-bit cross-check** (§5.5). The
   `chipmunk_original_security_summary.txt` rop bits (38.8, 38.8, 29.8)
   indeed sit in the 26-39 range, supporting the "~40 bits" claim. The
   audit correctly notes this is *committed estimator output*, not a
   re-run.

---

## Things the audit got wrong (correct here)

### W1. Aggregated-sig component breakdown arithmetic (§7)

The breakdown `Z 4.2 KB + path 50.8 KB + sib 116.8 KB + u 28.0 KB = 199.8 KB`
in the audit's CLAUDE.md verification subsection conflates the individual
KOTS sig Z (4320 B = 4.2 KB) with Z_agg in the aggregated signature
(5761 B = 5.6 KB). With Z_agg the sum is `5.6 + 50.8 + 116.8 + 28.0 =
201.2 KB`, no "1-byte attempt tag" needed. The audit should report:

> aggregated sig (N=1024) = Z_agg 5.6 KB + Babai path 50.8 KB +
> sibling labels 116.8 KB + u 28.0 KB = 201.2 KB total (206 065 B).

### W2. Audit §3.1 Vrfy "mod q'" wording

The audit (§3.1, line 125) writes "Z·A ≡ H·T (mod q')". The Python
`kots.py:vrfy` line 155 reads `Z*A == H*T (mod q)` where `self.q` inside
KOTS is `q'` (the KOTS modulus, named `q` in the KOTS class but
corresponding to `q'` in paper notation). So the audit's paper-notation
statement is correct; just be aware the Python code uses `self.q` for the
KOTS modulus (the *KOTS* class's `q`, not HVC's). Minor pedagogical
nit, not a factual error.

### W3. Section 5.4 ratio arithmetic

The audit computes `303.8 µs : 4.1 ms ≈ 1 : 13.5` and calls CLAUDE.md's
"≈ 1 : 12" "materially right". 4.1 ms / 303.8 µs = 13.5. The audit's
arithmetic is correct, but its framing — comparing local µs to paper-
machine ms — silently mixes two different machines. The honest local
ratio (303.8 µs : 11 ms scaled) is closer to 1 : 36, as the audit itself
states. Worth flagging that the "≈ 1 : 12" is a *paper-machine ratio*
extrapolated from local online-sign and paper-stated BDS08, and the audit
is correct on the order of magnitude but should not present the cross-
machine ratio as a clean reproduction.

---

## Things the audit missed (empirical claims not exercised)

### M1. CLAUDE.md "BDS state 134.4 KB" rounded vs raw 134.375 KB

Audit (§7) says BDS state "matches `lemur sizes` exactly (137 600 bytes)".
137600 B / 1024 = 134.375 KB. `lemur sizes` prints `(134.4 KB)`, which
is the same number to one decimal. Fine, but the audit could note that
the printed "134.4 KB" rounds 134.375 by half-up — not a discrepancy.

### M2. No verification of `bench_breakdown` 1.69× rayon scaling

Audit §4.2 cites "rayon scaling: 1.69× vs single-thread model" from
bench_breakdown but does not actually run it or quote the line. If
this number was measured, it should be sourced from the run log; if
inferred, it should be labeled as a prior-review claim. The current
text reads as a measurement but I cannot find that it was made in
the same session.

### M3. `tree_sign_matches_seed_path_every_slot` test — what τ does it use?

Audit §3.3 cites this test as confirming BDS-equals-seed equivalence but
does not state the τ value at which the test runs. If the test uses
τ=4 or τ=8 (which it does — small τ for test wall time), the audit
should note that the assertion is at small scale and the τ=20 production
path is not tested end-to-end. Audit alludes to this in §7 line 405 but
doesn't tie the two.

### M4. Aggregation 567 ms not reproduced or sanity-checked

The N=1024 aggregation row of Table 2 (567 ms) is the most reproducible
timing claim (it's the only measured aggregation cell — the others are
extrapolated). The audit cancels the run before this number is produced
("the run was cancelled before the aggregation stage produced output",
§4.2 bullet 4). A reviewer would have wanted at least a rough sanity-
check that aggregation completes in the order of seconds, not minutes,
at N=1024 on 11 threads. This is a measurement gap, not an audit error,
but it is missed.

### M5. RHF ≤ 1.0045 claim

Table 5 caption says "Lemur uses the fixed d=256, k=4 parameter set
and maintains RHF ≤ 1.0045 throughout." `summary.txt` rows have
`RHF_LWE_KOTS, RHF_SIS_KOTS, RHF_SIS_HVC` columns, all bounded by
1.0045 (max in d=256, k=4 family is 1.0045, exactly the bound). The
audit does not cross-reference this. Worth a one-line check that
the committed estimator output backs the RHF claim.

### M6. Paper claim "~10× smaller KOTS"

The lemur-prior-work-survey skill mentions "Dual Hint-MLWE assumption →
~10× smaller KOTS". The audit does not check this. The KOTS individual
sig at d=256, k=4 is 4.2 KB (4320 B per `lemur sizes`); Chipmunk's
individual KOTS sig at comparable parameters would be the comparison
target. Audit doesn't pull this out.

---

## Bottom-line meta

The audit is **substantively correct on every load-bearing claim** I
spot-checked (C1-C12). Its only material error is the aggregated-sig
component breakdown arithmetic, which is a transcription confusion
between Z (individual) and Z_aggregated. The §7.1-vs-implementation-cell
gap, the Rice-vs-worst-case distinction, the four code checkpoints, and
the Python↔Rust byte-equivalence are all correctly identified and
verified.

For aggregation: preserve the worked-example gap, the four checkpoint
verdicts, the Rice/raw distinction, and the test/parameter cross-refs.
Correct the Z/Z_agg conflation. Note the measurement gaps (M2, M3, M4).
