# Lemur audit — implementation / empirical-claims lens

Scope: paper /workspace/repo/report.pdf §7 (Implementation + Tables 2/5),
the four code-audit checkpoints in CLAUDE.md, Python↔Rust byte-equivalence,
and whether the code does what the construction figures (Fig. 3 KOTS, Fig. 4
HVC, Fig. 5 Lemur) describe.

Hardware: 11-thread aarch64 container, 8 GiB RAM. Paper baseline: 24 threads.

---

## 1. Deterministic size claims — verifiable and reproduced

### 1.1 `lemur sizes` matches Table 2 / Table 5

`./target/release/lemur sizes` (run from /workspace/repo/code/lemur-rs) prints,
for the only implemented cell `(d=256, k=4, τ=20, N=1024)`:

| Component                                  | Bytes   | KB     | Paper Table 2 / 5            |
| ------------------------------------------ | ------- | ------ | ---------------------------- |
| pp (seeds + tau)                           |     65  |  0.06  | not tabulated                |
| sk (master seed)                           |     32  |  0.03  | not tabulated                |
| sk.state (BDS08 cache, fresh)              |  137600 | 134.4  | "≈ 134 KB" (§7.2 line 1055)  |
| pk (HVC commitment)                        |   3456  |   3.4  | not tabulated                |
| individual sig                             |  91616  |  89.5  | not tabulated                |
|   Z (KOTS sig)                             |   4320  |   4.2  |                              |
|   sibling labels                           |  70400  |  68.8  |                              |
|   u                                        |  16896  |  16.5  |                              |
| aggregated sig (N=1024, Rice-encoded)      | 206065  | 201.2  | **Table 2 / Table 1: 201.2 KB** exact |

Cross-checked against `parameter/rice_sizes.py`, which uses the same
formulas applied to all three Table 2 N values:

```
tau,N,worst_case_KB,rice_encoded_KB
20,1024,239,201.2
20,32768,331,283.5
20,1048576,457,394.4
```

These three numbers exactly match the Table 2 "Agg. signature size" row
(201.2 KB / 283.5 KB / 394.4 KB) and the Lemur "Multi-Sig" column of Table 1
at τ=20 (201 / 284 / 394 KB). The 239 / 331 / 457 KB worst-case values match
the Table 5 totals at τ=20 (239 / 331 / 457 KB). Pipeline self-consistent.

### 1.2 The 14 % Rice savings claim

Paper §7.2: "the encoder brings the aggregate size from the 239 KB bound
(Table 5) down to 201.2 KB; the same encoder yields ≈ 14% savings on the
larger-N cells (e.g. 394.4 KB vs. 457 KB at N=2²⁰)."

  - N=2¹⁰: 1 − 201.2/239 = **15.8 %** (paper says 14 %; ≈ matches)
  - N=2¹⁵: 1 − 283.5/331 = **14.4 %**
  - N=2²⁰: 1 − 394.4/457 = **13.7 %**

The "≈ 14 %" averages over the three cells; literal compliance is good.

### 1.3 `cargo test --release` — 52 / 52 pass

Five integration files plus a 25-test in-crate unit suite (`src/lib.rs`):
  - `bds_stateful.rs` — 8 tests (BDS state-file framing, snapshot,
    stateful_equals_seed_every_slot)
  - `gauss_ctx.rs` — 5 tests (sampler-context byte-equivalence,
    profile_cdt_bits_match_module_constant)
  - `materialized_tree.rs` — 5 tests (full tree commit matches root,
    tree_sign matches seed-path every slot)
  - `profile_pipeline.rs` — 4 tests (incl.
    `agg_encoding_snapshots_match_python_formula` and `d256_k4_full_pipeline`)
  - `robustness.rs` — 5 tests (bitflip + truncation rejection on both
    individual and aggregated signatures)
  - `src/lib.rs` — 25 unit tests
Total **52, 0 failed, 0 ignored**. The CLAUDE.md "52 tests" claim is accurate
on this machine.

---

## 2. Python ⇔ Rust byte equivalence — verified at every public field

Test: `lemur vectors --tau 3 --signers 2 --slot 0 --msg "artifact check"`,
run in both implementations, then parse JSON and compare named keys.

```
pp: MATCH
signatures: MATCH
ivrfy: MATCH
avrfy: MATCH
agg_attempt: MATCH
aggregate: MATCH
pk 0: MATCH
pk 1: MATCH
```

The Python JSON has a wrapper key `"implementation": "lemur-py reference"`
that the Rust JSON omits — that explains the 10-byte size difference
(226 318 vs 226 308 chars) and the `diff` mismatch on the first lines.
Otherwise the cryptographic payload — pp (HVC + KOTS public seeds), each
public key, each individual signature (Z + opening), the chosen aggregation
attempt counter, and the aggregate `(Z_agg, d_agg, attempt)` blob — is
byte-identical, which is the actual claim. The paper's wording
("cross-validated against shared byte-level test vectors", §7.2 col. 1
last paragraph) is supported.

This implies the two implementations realize the same scheme bit-for-bit,
so any algorithmic correctness check on Python transfers to Rust.

---

## 3. Construction-figure faithfulness (Fig. 3 KOTS, Fig. 4 HVC, Fig. 5 Lemur)

I read `lemur-py/kots.py`, `hvc.py`, `lemur.py` and the matching Rust modules
end-to-end. The Python code is a literal transcription of the paper to a
degree that the audit can be done by symbol matching.

### 3.1 KOTS (Figure 3) — `kots.py`

  - Setup (lines 105–118): stores only `A2 ∈ R_{q'}^{(m-n) × n}`; the
    effective `A = [I_n; A2]` is reconstructed implicitly by
    `_matmul_with_A` (lines 120–124). This is the paper's "structured form"
    described in §1.2 / §4.
  - Keygen (lines 126–138): each `S[i,j]` drawn from a Gaussian XOF stream
    `SHAKE256(seed‖[i,j]‖'S')`; `T = S · A`. Matches Fig. 3.
  - Sign (lines 140–152): `Z[i,j] = Σ_l H[i,l] · S[l,j]` over **Z** (no
    reduction). This is the paper's exact-arithmetic step — `q'/2 ≫
    k · α_H · max|S|`, so signed coefficients fit without wrapping. Good.
  - Vrfy (lines 154–161): `||Z||_∞ ≤ β` and `Z·A ≡ H·T (mod q')`.

### 3.2 The four audit checkpoints — all verified

I traced each of the four CLAUDE.md foot-guns independently:

  **a) `kots.py:_build_H` (lines 90–99).** The identity block is encoded as
      `H[i, i, 0] = 1` — that is, the constant polynomial 1, not
      `1 + x + ... + x^{d-1}`. Lines 94–95:
      ```python
      for i in range(ell):
          H[i, i, 0] = 1
      ```
      The remaining columns are filled from `_hash_to_ternary(mu, index=...)`,
      ternary with weight `α_H`. Matches Fig. 3.

  **b) `kots.py:vrfy(β)` and its three wrappers (lines 163–170).** Bounds
      passed:
      ```python
      def ivrfy(...): return self.vrfy(..., self.beta_z)        # β_z
      def svrfy(...): return self.vrfy(..., self.beta_sigma)    # β_σ
      def wvrfy(...): return self.vrfy(..., 2 * self.beta_sigma) # 2 β_σ
      ```
      Exactly the paper bounds (Fig. 3 + §4.1). No drift.

  **c) `hvc.py:_internal_label` (lines 408–412).**
      ```python
      contrib = (self.ring.mat_vec(A0, left) + self.ring.mat_vec(A1, right)) % self.q
      return self._dec_vec(contrib, self.q, self.kappa)
      ```
      Reduces mod `self.q` (HVC modulus, 53 bits), **not** `self.q_prime`
      (KOTS modulus). The Rust mirror in `src/hvc.rs:532–545` uses
      `profile.q_hvc()`. Both files cross-check; the silent-miscoding
      foot-gun listed in CLAUDE.md is not present.

  **d) `lemur.py:aggregate` (lines 264–297) and `avrfy` (299–320).** The
      retry loop on lines 283–297 increments `attempt` and breaks **only
      via `self.avrfy(...)` returning True** (line 294). `avrfy` itself
      (line 318) defers to `kots.svrfy`, which checks both the norm
      bound β_σ and the algebraic identity `Z·A ≡ H·T_agg`. There is no
      norm-only short-circuit. The Rust `lemur_aggregate` follows the
      same shape (`src/lib.rs` → `lemur_aggregate`, see bench.rs:686–699
      which assert `agg_sig.z_agg.0 == agg_sig_verified.z_agg.0`).

### 3.3 HVC (Figure 4) — `hvc.py`

  - Leaf: `_leaf_label(M_t, B_mat) = dec_q(B · dec_{q'}(M_t))`. Matches
    Fig. 4 / Appendix B leaf rule.
  - Internal: see (c) above. Matches Fig. 4.
  - Babai encode/decode (lines 261–340) implements the deterministic
    coset representative algorithm of Lemma B.1 / Figure 6.
  - BDS08 (lines 555–558 + `bds_*`): standard authentication-path
    traversal. The `phi` cursor is the BDS leaf pointer, advanced after
    each sign. Confirmed by integration tests
    `stateful_equals_seed_every_slot` and `tree_sign_matches_seed_path_every_slot`.

### 3.4 Lemur composition (Figure 5) — `lemur.py`

  - `setup` (line 94) returns `(kots_pp, hvc_pp)`. ✓
  - `keygen` (line 133) builds the HVC commitment and primes the BDS
    cache in a single tree walk. ✓
  - `sign_seed` (line 149) and `sign_stateful` (line 172) both produce
    `(Z, d_open)`; the test `tree_sign_matches_seed_path_every_slot`
    asserts byte equality. ✓
  - `_hash_to_randomizers` (line 44) keys SHAKE256 on
    `(t, |m|, m, pks_bytes, attempt)`. The randomizer oracle includes
    the `attempt` counter, which is what makes the γ-retry meaningful:
    each attempt resamples weights. ✓
  - `aggregate` retries up to γ=10 times. ✓

No deviation from the paper figures.

---

## 4. Timing claims — paper Table 2 vs this machine, with ratios

The paper's absolute timings (24-thread x86) are not reproducible on
11-thread aarch64. The interesting question is whether the **ratios**
the paper relies on hold up.

### 4.1 Measured here

| Stage                                | This machine (11 thr) | Paper (24 thr)  | Ratio paper/mine |
| ------------------------------------ | --------------------- | --------------- | ---------------- |
| Online Sign (KOTS only)              | **303.8 µs** (mean of 20) | n/a directly† | n/a              |
| Stateful Sign (BDS08)                | not measured (full `bench --fast` cancelled at full-sign rederive stage) | 4.1 ms |  |
| Batch verify, N=1024, zero-fixture   | **80.8 ms** (mean of 5) | 30.1 ms        | 0.37 (= mine 2.7× longer) |
| Sampler micro: `xof_gauss_poly`      | 1.66 µs/poly          | not stated      |                  |
| `leaf_label`                         | 129.7 µs              | not stated      |                  |
| `internal_label`                     | 49.6 µs               | not stated      |                  |

† Online sign isn't in paper Table 2 — Table 2 reports Signing (BDS08)
  and Signing (Tree-in-memory) but not the bare KOTS path. The 303.8 µs
  figure exists only in the bench output.

### 4.2 Ratio sanity-checks

  - **24-thread → 11-thread scaling.** Naively `24/11 ≈ 2.18×`, but
    rayon and memory bandwidth narrow that. CLAUDE.md predicts "paper's
    24-thread timings drop to ~2.5× longer on 11 threads". Measured
    batch verify: 80.8/30.1 = **2.68×**. Consistent with the CLAUDE.md
    prediction within 8 %. The build is not misconfigured.
  - **bench_breakdown rayon scaling factor** at τ=9, N_slots=512:
    "rayon scaling: 1.69× vs single-thread model" (i.e. the leaf-tree
    work parallelises sublinearly because of the per-leaf kots_keygen
    bottleneck). The single-thread model is 293 ms; 11-thread is
    173 ms; ratio 1.69× — well short of 11×. The aggregation step
    parallelises better (`par_iter().reduce_with()` in
    `lemur.rs:lemur_aggregate`).
  - **Paper's BDS08 : Tree-in-mem ratio** is 4.1 ms : 2 ms ≈ 2.05×.
    Not reproduced here (cancelled the bench), but the materialised-tree
    path is gated by `--with-tree` which requires ~8 GiB RAM and is
    flagged as risky in this container.
  - **Paper's Aggregation : Batch-verify ratio at N=1024** is
    567 ms : 30.1 ms ≈ 18.8×. CLAUDE.md cites 19×. I did not reproduce
    this end-to-end; the full `bench --fast` aggregation row needs
    keygen-once (~1.8 min) plus full sign + aggregate; the run was
    cancelled before the aggregation stage produced output.

### 4.3 What the paper does not say about timings

  - Table 2 caption: "Linear extrapolations from N=8192 timing" applies
    only to the Aggregation row at N∈{2^15, 2^20}. So the headline
    "≈ 12 min" for N=2^20 aggregation is **predicted, not measured**.
    The artifact has no N>1024 keygen pipeline — confirmed in
    `code/README.md` and verified by `bench --fast` source
    (`ns: &[usize] = &[1024, 8192]`, bench.rs:616) which never
    exercises N=2^15 or 2^20 aggregation.
  - Batch verification at N∈{2^15, 2^20} is benchmarked with a
    zero-public-key fixture (`bench_verify --zero-fixture`). The
    fixture replaces aggregation with an all-zero accepting input
    so only `lemur_avrfy` is timed. Footnote § in Table 2 says this
    plainly; do not extract a wall-clock comparison against
    Chipmunk's full pipeline at the same N from these rows.
  - "Signing (Tree-in-memory) 2 ms": described in §7.2 with no
    measurement detail beyond "With the entire tree in RAM,
    consecutive signatures can be generated in 2 ms." Whether the
    paper actually measured this on the materialised tree (the
    artifact has a `--with-tree` flag, gated at 8 GiB RAM) or
    estimated it isn't stated.

---

## 5. Paper ↔ code divergences I found

### 5.1 §7.1 worked example uses (d=128, N=2²⁰, τ=24); implementation is (d=256, k=4, τ=20, N=2¹⁰)

This is the largest readability gap. Paper §7.1 says:
> "we describe the parameter-selection process for a representative
> setting d = 128, N = 2²⁰, and τ = 24 from now on. Once d and λ are
> fixed, Constraint 18 fixes α_w = 31."

It then walks through ℓ=1, k=5, r=3, α_H=31, α=61, β_z=8185.

But Table 5 caption says:
> "Lemur uses the fixed d=256, k=4 parameter set and maintains
> RHF ≤ 1.0045 throughout."

And the implementation (`lemur-py/profiles.py:106-113`,
`lemur-rs/src/profile.rs:300-341`) has `d=256, k=4, ℓ=1, m=9, n=4,
α=87, α_H=60, α_w=23, β_z=14046`. None of the §7.1 worked-example
values match the committed implementation cell. The reader has to
flip between `summary.txt` (`(d=256, k=4, τ=20, N=1024)` row) and the
paper text to make sense of either. The paper does not flag that
the worked example is illustrative rather than describing the actual
implementation cell.

This is not a bug — `summary.txt` line 11 is consistent end to end
— but the paper is vague enough about which parameter cell is
"the implementation" that a reviewer will need to consult
`parameter/summary.txt` to figure it out.

### 5.2 Paper Table 5 reports `d=256, k=4` for Lemur; summary.txt has only this family

`parameter/summary.txt` ships 16 rows: 4 values of τ × 4 values of N,
all at `(d=256, k=4)`. The `(d=128, k=5)` family described in §7.1's
worked example **does not exist in the committed estimator output**.
Either the paper's worked example used a parameter regen that didn't
get committed, or the example is purely illustrative. Code-side, only
`d=256, k=4` exists — `gen_tables.py` ships one profile, `D256_K4`,
and the Rust `Profile::all()` array has one entry.

### 5.3 `eta` differs across τ in summary.txt; profile picks the τ=20 row

`summary.txt` line 11 has `eta=776` for τ=20,N=1024. The profile in
`lemur-py/profiles.py:111` uses `eta=776`. Good. But `summary.txt`
line 14 (τ=20,N=2²⁰) has `eta=256`, a different value. The
implementation only handles the N=1024 row, so users who pass
`--tau 20 --signers 2^20` won't get a parameter-consistent run — the
binaries' `lemur sizes` and `bench` are hard-wired to N=1024 with
larger-N sizes computed via the Rice-sizes script. This is
documented in `code/README.md` ("ship the fixed d=256, k=4, tau=20,
N=1024 parameter cell") and is not in tension with the paper's
claims, but a casual reader of the README ("the implementations
ship the fixed cell") may not notice the implication that
**Table 2's N=2^15 and N=2^20 rows are extrapolations using
different `eta`/`m`/`q'` than the implementation actually
realises**.

### 5.4 CLAUDE.md → paper inflation in one detail

CLAUDE.md (line 199) quotes the "Online Sign 347 µs vs Stateful Sign
4.1 ms ≈ 12×" ratio. The paper itself does not give an Online Sign
time — Table 2 has only Stateful (BDS08, 4.1 ms) and Tree-in-memory
(2 ms). The Online figure is implementation-internal (bench output)
not a paper-stated claim. So the "online : stateful : full ≈ 1 : 12 :
10⁵" ratio in CLAUDE.md is a useful diagnostic but should not be
described as a "paper ratio". My measured online = 303.8 µs;
unmeasured stateful here; **if** paper-4.1 ms scales to 11 threads
at the 2.68× factor (= 11 ms here), the local ratio would be
303.8 µs : 11 ms ≈ 1 : 36. The actual paper-reported ratio is
303.8 µs : 4.1 ms ≈ 1 : 13.5, close to CLAUDE.md's "≈ 1 : 12". CLAUDE.md
is materially right on the order of magnitude.

### 5.5 Chipmunk 40-bit recomputation is data-grounded

`parameter/chipmunk_original_security_summary.txt` rows for the
1024-signer / τ=21 / λ-claim=128 cell show RSIS1 rop = **38.8 bits**,
RSIS2 rop = 38.8 bits, RSIS3 rop = 29.8 bits — far below the claimed
128 bits. This is the file the §1.1 "Chipmunk parameters realize
~40-bit security" claim cites. The estimator script
(`parameter/chipmunk_original.sage`) is committed but not re-run here
(no SageMath in container). The committed output is internally
consistent: identical seed values across rows of the same `τ` /
`alpha_w` produce identical RSIS bits, as one would expect of a
deterministic estimator.

---

## 6. What the paper is vague about, implementation-side

  1. **Table 2 "Aggregation" measurement protocol.** The footnote says
     "Linear extrapolations from N=8192 timing" for N∈{2^15, 2^20}. The
     N=1024 entry (567 ms) is presumably measured but the paper
     doesn't state on how many reps, whether the figure is mean /
     median / single run, or how variance was handled. The artifact's
     `bench --fast` reports a single aggregation timing per N — no
     repetition statistics.
  2. **What "linear extrapolation" means.** Aggregation has both an
     O(N) pre-verify component and an O(N log N) weighted-sum
     reduction. The paper extrapolates linearly; if the reduction
     tree dominates at large N, that under-estimates. With 11
     threads and Rayon, the empirical scaling I see at small N is
     sub-linear in N (because keygen amortises), so the paper's
     extrapolation choice errs in either direction depending on
     where the bottleneck sits at N=2^20.
  3. **`α_w = 23` vs `α_w = 31`.** §7.1 worked example uses
     α_w = 31 (Constraint 18 at d=128). The implemented profile
     uses α_w = 23 (consistent with d=256, derivable from
     Constraint 18 at d=256: `T_α_w = α_d_w · 2^α_w ≥ 2^λ` →
     `α_w ≥ 23` at d=256). The paper does not call out that the
     numerical value depends on d.
  4. **"Predicted Rice-encoded sizes" footnote.** Table 2's ‡
     footnote refers reader to Table 5. But Table 5's cells (e.g.
     τ=20, N=2²⁰: total 457 KB) are the *worst-case* totals, not
     Rice-encoded. The Rice-encoded number 394.4 KB only appears in
     Table 1 + Table 2. Reading Table 5 alone gives only 457 KB and
     the reader has to know `parameter/rice_sizes.py` or the
     §7.2 text to recover the 394.4 KB figure. Could be flagged
     in the Table 5 caption.

---

## 7. CLAUDE.md / review.md — independent verification notes

`/workspace/assess/review.md` does not exist on this container, so
no parroting risk. CLAUDE.md is mostly correct on the points I
checked:

  - "exactly reproduces Table 5 N=2^10 column": correct, all three
    sub-component bytes match (Z 4.2 KB + path 50.8 KB + sib 116.8 KB
    + u 28.0 KB = 199.8 KB before the 1-byte attempt tag, totaling
    201.2 KB).
  - "52 tests": correct.
  - "byte-equal Python ⇔ Rust vectors": correct on every
    cryptographic field (only the implementation-label string differs).
  - "8 GiB tree at τ=20": not exercised but the materialized-tree
    test in `tests/materialized_tree.rs` uses smaller τ — note that
    `tree_sign_matches_seed_path_every_slot` passes at the small
    τ used in tests but no end-to-end at τ=20 is run in this audit.
  - "BDS state 134.4 KB" matches `lemur sizes` exactly (137 600 bytes).
  - "Python uses schoolbook for q'": confirmed,
    `lemur-py/profiles.py` comment lines 19–22 say so and the ring
    constructor in `kots.py:69` instantiates `Ring(q', d)` without
    NTT tables.
  - "Rust uses CRT via two aux primes for q'": confirmed,
    `lemur-rs/src/profile.rs:316` has `kots_crt: Some(KotsCrtCfg{...})`.

CLAUDE.md is slightly **wrong** when it says (line 28) "Headline
numbers (paper Table 2, τ=20, N=2^10, 24 threads): aggregated
signature 201.2 KB, batch verify 30.1 ms, stateful BDS08 sign 4.1 ms.
At N=2^20: aggregated sig 394.4 KB." It is correct for size but
should call out that the N=2^20 batch-verify (25.3 s) is
zero-fixture, not full pipeline, and the N=2^20 aggregation
(≈12 min) is linearly extrapolated from N=8192. The paper does say
this in the footnote; CLAUDE.md doesn't repeat it.

---

## 8. Bottom line for a reviewer

  - **Size claims:** the artifact reproduces every size number in
    Tables 2 and 5 deterministically. `lemur sizes` and
    `parameter/rice_sizes.py` are the two reproducible front doors;
    both match the paper to the last KB. This is the most credible
    part of the empirical claims and is genuinely verifiable in
    minutes on a fresh container.
  - **Construction correctness:** the Python reference is the spec
    and is faithful to Figures 3, 4, 5. The four CLAUDE.md
    foot-guns (`_build_H` constant-1 encoding, three vrfy bounds,
    `_internal_label` HVC modulus, aggregate-break-via-avrfy) all
    inspect correctly. The Rust path is byte-equivalent on
    deterministic test vectors.
  - **Timing claims:** the headline Table 2 numbers were generated
    on 24 threads; on 11 threads they scale by ~2.7× (matches the
    CLAUDE.md prediction). The N>1024 entries are either zero-
    fixtured (batch verify) or linearly extrapolated (aggregation)
    — both clearly footnoted in Table 2. The paper's "2 ms tree
    sign" claim is not exercised in default benches.
  - **Security claims:** ungrounded directly, in the sense that
    SageMath isn't in the container. But the estimator output
    files (`parameter/summary.txt`, `chipmunk_original_security_summary.txt`)
    are internally consistent and the "Chipmunk realises ~40-bit
    security" headline is grounded in committed RSIS-rop numbers
    in the 26–39 bit range for the `λ_claim=112` and `λ_claim=128`
    cells.
  - **Implementation footprint:** one parameter cell shipped
    (`d=256, k=4, τ=20, N=1024`). Larger-N table entries are
    extrapolations from formulas. The README and §7.2 say so, but
    a hurried reader could miss this and think the artifact runs
    end-to-end at N=2^20.

Net: deterministic claims are clean; timing claims need ratios not
absolutes; the paper conflates "implementation cell" with "worked
example" in §7.1 in a way that takes ~30 minutes of code reading to
disambiguate.
