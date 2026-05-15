# Fact-check — review.md §4–§7

Scope: lines ~195–485. One entry per atomic factual claim.

---

### FC2-001  [§4 intro]
**Claim:** "Python reference (`lemur-py/`, ~3.6 kLOC) is the spec"
**Location:** review.md line ~205
**Status:** UNVERIFIABLE
**Source:** Not counted; LOC tally not in submission. Order-of-magnitude plausible (8 .py files, sample 173–660-line ranges).
**Note:** Approximate count not exhaustively confirmed; non-load-bearing.

### FC2-002  [§4 intro]
**Claim:** "Rust port (`lemur-rs/`, ~9 kLOC)"
**Location:** review.md line ~206
**Status:** UNVERIFIABLE
**Source:** As above; not measured.

---

## §4.1 KOTS structural mapping

### FC2-003  [§4.1]
**Claim:** `Setup` at `kots.py:setup` lines 105–118
**Location:** review.md line ~213
**Status:** VERIFIED
**Source:** `lemur-py/kots.py:105` (`def setup`) to line 118 (`return A2`).

### FC2-004  [§4.1]
**Claim:** `KGen` at `kots.py:keygen` lines 126–138, "Gaussian via CDT with σ = α / √(2π)"
**Location:** review.md line ~214
**Status:** VERIFIED
**Source:** `lemur-py/kots.py:126` (`def keygen`); body 126–138. σ=α/√(2π) reference is in `codec.rs:341–345` comment ("sigma = alpha / sqrt(2*pi)"); concept matches paper Fig 3.

### FC2-005  [§4.1]
**Claim:** `_build_H` at lines 90–99
**Location:** review.md line ~215
**Status:** VERIFIED
**Source:** `lemur-py/kots.py:90` (`def _build_H`) — body extends through line 99.

### FC2-006  [§4.1]
**Claim:** `sign` lines 140–152
**Location:** review.md line ~215
**Status:** VERIFIED
**Source:** `lemur-py/kots.py:140` (`def sign`) through line 152.

### FC2-007  [§4.1]
**Claim:** "identity block encoded as H[i,i,0]=1 (constant polynomial 1)"
**Location:** review.md line ~215
**Status:** VERIFIED
**Source:** `lemur-py/kots.py:95` reads `H[i, i, 0] = 1`.

### FC2-008  [§4.1]
**Claim:** "Three thin wrappers ivrfy/svrfy/wvrfy (lines 163–170) over vrfy(beta) (lines 154–161)"
**Location:** review.md line ~216
**Status:** VERIFIED
**Source:** `lemur-py/kots.py:154` (`def vrfy`), :163 (`def ivrfy`), :166 (`def svrfy`), :169 (`def wvrfy`).

---

## §4.2 HVC structural mapping

### FC2-009  [§4.2]
**Claim:** `hvc.py:setup` lines 438–452
**Location:** review.md line ~222
**Status:** VERIFIED
**Source:** `lemur-py/hvc.py:438` (`def setup`); closing `)` at line 452.

### FC2-010  [§4.2]
**Claim:** `_leaf_label` lines 403–406
**Location:** review.md line ~223
**Status:** VERIFIED
**Source:** `lemur-py/hvc.py:403` (`def _leaf_label`); ends line 406 with `return self._dec_vec(...)`.

### FC2-011  [§4.2]
**Claim:** `_internal_label` lines 408–412; "reduces mod self.q (HVC modulus), not q_prime"
**Location:** review.md line ~224
**Status:** VERIFIED
**Source:** `lemur-py/hvc.py:408–412`. Line 411 is `) % self.q`; line 412 is the wrapping `self._dec_vec(contrib, self.q, self.kappa)`. Confirmed `% self.q` not `% self.q_prime`.

### FC2-012  [§4.2]
**Claim:** `_decompose_coeff`/`_dec_poly`/`_proj_poly` lines 190–235
**Location:** review.md line ~225
**Status:** VERIFIED
**Source:** `lemur-py/hvc.py:190` (`def _decompose_coeff`), `:204` (`def _dec_poly`), `:212` (`def _proj_poly`). 235 is past `_proj_poly` body.

### FC2-013  [§4.2]
**Claim:** `_decompose_coeff_zz` / `babai_encode_block` / `babai_decode_block` lines 241–340
**Location:** review.md line ~226
**Status:** VERIFIED
**Source:** `lemur-py/hvc.py:241` (`_decompose_coeff_zz`), `:261` (`babai_encode_block`), `:290` (`babai_decode_block`). End ~340 plausible.

### FC2-014  [§4.2]
**Claim:** "BDS08 traversal [4]: ... lines 467–635"
**Location:** review.md line ~227
**Status:** VERIFIED
**Source:** `lemur-py/hvc.py:467` (`def bds_init`), `:521` (`def bds_advance`), `:602` (`def bds_opening`). Body extends to ~635.

---

## §4.3 Lemur composition

### FC2-015  [§4.3]
**Claim:** "lemur.py: setup/sign_seed/sign_stateful/ivrfy/avrfy lines 94–321"
**Location:** review.md line ~233
**Status:** VERIFIED
**Source:** `lemur-py/lemur.py:94` (`def setup`), `:149` (`sign_seed`), `:172` (`sign_stateful`), `:245` (`ivrfy`), `:299` (`avrfy`); avrfy body ends ~line 320.

### FC2-016  [§4.3]
**Claim:** "aggregate lines 264–297; randomizer oracle at _hash_to_randomizers lines 44–58"
**Location:** review.md line ~234
**Status:** PARTIALLY CORRECT
**Source:** `lemur-py/lemur.py:264` (`def aggregate`); `_hash_to_randomizers` at line 44.
**Note:** The hash-to-randomizers function actually spans lines 44–60 (return at line 60), not 44–58. Minor off-by-2.

### FC2-017  [§4.3]
**Claim:** "break on avrfy passing (line 294)"
**Location:** review.md line ~234
**Status:** VERIFIED
**Source:** `lemur-py/lemur.py:294` is `if self.avrfy(pp, pks, t, m, (Z_agg, d_agg, attempt)):`.

### FC2-018  [§4.3]
**Claim:** "Weighted-sum on Z and on opening: lemur.py:_scale_opening + _add_openings + per-signer kots.ring.scale_mat"
**Location:** review.md line ~235
**Status:** VERIFIED
**Source:** `lemur-py/lemur.py:61` (`_add_openings`), `:71` (`_scale_opening`), `:288` (`Z_agg += self.kots.ring.scale_mat(...)`).

---

## §4.4 Audit checkpoints

### FC2-019  [§4.4]
**Claim:** "kots.py:_build_H line 95: identity block is H[i,i,0]=1"
**Location:** review.md line ~243
**Status:** VERIFIED
**Source:** `lemur-py/kots.py:95` literally `H[i, i, 0] = 1`.

### FC2-020  [§4.4]
**Claim:** "kots.py:vrfy lines 154–170: β_z, β_σ, 2·β_σ bounds exactly"
**Location:** review.md line ~245
**Status:** VERIFIED
**Source:** `kots.py:164` `self.beta_z`, `:167` `self.beta_sigma`, `:170` `2 * self.beta_sigma`.

### FC2-021  [§4.4]
**Claim:** "hvc.py:_internal_label line 411: reduction is % self.q (HVC modulus), not q' (KOTS modulus)"
**Location:** review.md line ~247
**Status:** VERIFIED
**Source:** `lemur-py/hvc.py:411` is `) % self.q`.

### FC2-022  [§4.4]
**Claim:** "lemur.py:aggregate line 294: breaks via avrfy"
**Location:** review.md line ~249
**Status:** VERIFIED
**Source:** `lemur-py/lemur.py:294`.

---

## §5.1 Rust test suite

### FC2-023  [§5.1]
**Claim:** "52/52 tests pass across 7 test binaries"
**Location:** review.md line ~261
**Status:** PARTIALLY CORRECT
**Source:** Counted with `grep -c "#\\[test\\]"`: tests/ dir has 8+5+5+4+5 = 27 integration tests; src/ has 25 unit tests (codec=1, aux_ntt=11, kots=1, profile=3, hvc=5, ntt=4) → 52 total.
**Note:** 52 total is correct, but "7 test binaries" is off. Integration tests live in 5 files (bds_stateful, gauss_ctx, materialized_tree, profile_pipeline, robustness). Plus the unit-test runner (lib) = 6. The 7th is doctests (which the table itself says is 0). Either 6 binaries with tests, or 7 counting the doctest harness.

### FC2-024  [§5.1]
**Claim:** "unit tests in lemur_rs: 25"
**Location:** review.md line ~265
**Status:** VERIFIED
**Source:** `grep -c '#\\[test\\]'` over `lemur-rs/src/`: codec.rs=1, aux_ntt.rs=11, kots.rs=1, profile.rs=3, hvc.rs=5, ntt.rs=4 → 25.

### FC2-025  [§5.1]
**Claim:** "bds_stateful: 8"
**Location:** review.md line ~266
**Status:** VERIFIED
**Source:** `lemur-rs/tests/bds_stateful.rs`: 8 `#[test]` occurrences.

### FC2-026  [§5.1]
**Claim:** "gauss_ctx: 5"
**Location:** review.md line ~267
**Status:** VERIFIED
**Source:** `lemur-rs/tests/gauss_ctx.rs`: 5 `#[test]`.

### FC2-027  [§5.1]
**Claim:** "materialized_tree: 5"
**Location:** review.md line ~268
**Status:** VERIFIED
**Source:** `lemur-rs/tests/materialized_tree.rs`: 5 `#[test]`.

### FC2-028  [§5.1]
**Claim:** "profile_pipeline: 4"
**Location:** review.md line ~269
**Status:** VERIFIED
**Source:** `lemur-rs/tests/profile_pipeline.rs`: 4 `#[test]`.

### FC2-029  [§5.1]
**Claim:** "robustness: 5"
**Location:** review.md line ~270
**Status:** VERIFIED
**Source:** `lemur-rs/tests/robustness.rs`: 5 `#[test]`.

### FC2-030  [§5.1]
**Claim:** "Total: 52 tests. Time: ~1 minute on 11 threads"
**Location:** review.md line ~273
**Status:** VERIFIED (count)
**Source:** 25 + 27 = 52. Time not re-measured in this audit.

---

## §5.2 Size table — `lemur sizes`

### FC2-031  [§5.2]
**Claim:** pp = 65 B
**Location:** review.md line ~285
**Status:** VERIFIED
**Source:** `./target/release/lemur sizes` output line "pp (seeds + tau)  65".

### FC2-032  [§5.2]
**Claim:** sk master seed = 32 B
**Location:** review.md line ~286
**Status:** VERIFIED
**Source:** lemur sizes: "sk (master seed)  32".

### FC2-033  [§5.2]
**Claim:** sk.state fresh BDS = 134.4 KB (137600 B)
**Location:** review.md line ~287
**Status:** VERIFIED
**Source:** lemur sizes: "sk.state (fresh BDS)  137600  (134.4 KB)".

### FC2-034  [§5.2]
**Claim:** pk (HVC commitment) = 3.4 KB
**Location:** review.md line ~288
**Status:** VERIFIED
**Source:** lemur sizes: "pk (HVC commitment)  3456 (3.4 KB)".

### FC2-035  [§5.2]
**Claim:** Individual sig = 89.5 KB
**Location:** review.md line ~289
**Status:** VERIFIED
**Source:** lemur sizes: "individual sig  91616 (89.5 KB)".

### FC2-036  [§5.2]
**Claim:** Z (KOTS sig individual) = 4.2 KB
**Location:** review.md line ~290
**Status:** VERIFIED
**Source:** lemur sizes: "Z (KOTS sig)  4320  (4.2 KB)".

### FC2-037  [§5.2]
**Claim:** sibling labels (individual) = 68.8 KB
**Location:** review.md line ~291
**Status:** VERIFIED
**Source:** lemur sizes: "sibling labels  70400  (68.8 KB)".

### FC2-038  [§5.2]
**Claim:** u (individual) = 16.5 KB
**Location:** review.md line ~292
**Status:** VERIFIED
**Source:** lemur sizes: "u  16896  (16.5 KB)".

### FC2-039  [§5.2]
**Claim:** Aggregated sig N=2^10 = 201.2 KB
**Location:** review.md line ~293
**Status:** VERIFIED
**Source:** lemur sizes: "aggregated sig (N=1024, ~201.2 KB)  206065 (201.2 KB)". Matches paper Table 1 / Table 2 (201.2 KB).

### FC2-040  [§5.2]
**Claim:** Z_agg = 5.6 KB at "20b wider bit-width tuned to β_σ"
**Location:** review.md lines ~294, 298–300
**Status:** VERIFIED
**Source:** lemur sizes: "Z_agg (20b, bound=378933)  5761 (5.6 KB)". Code path in `codec.rs:354–367` derives `zagg_dx` from `bit_length(2*zagg_bound)` where `zagg_bound = min(c_zagg*sigma_zagg, beta_sigma)`.

### FC2-041  [§5.2]
**Claim:** Babai path (Rice k=5) = 50.8 KB
**Location:** review.md line ~294
**Status:** VERIFIED
**Source:** lemur sizes: "Babai path (Rice k=5)  52000 (50.8 KB)".

### FC2-042  [§5.2]
**Claim:** sibling labels (Rice k=15) = 116.8 KB
**Location:** review.md line ~295
**Status:** VERIFIED
**Source:** lemur sizes: "sibling labels (Rice k=15)  119600 (116.8 KB)".

### FC2-043  [§5.2]
**Claim:** u (Rice k=15) = 28.0 KB
**Location:** review.md line ~296
**Status:** VERIFIED
**Source:** lemur sizes: "u (Rice k=15)  28704 (28.0 KB)".

### FC2-044  [§5.2]
**Claim:** "Sub-component sum: 5.6 + 50.8 + 116.8 + 28.0 = 201.2 KB"
**Location:** review.md line ~297
**Status:** VERIFIED (arithmetic)
**Source:** 5.6+50.8+116.8+28.0 = 201.2 exactly.

### FC2-045  [§5.2]
**Claim:** "`bench --fast` realized aggregate-size: 195.8 KB"
**Location:** review.md line ~303
**Status:** UNVERIFIABLE
**Source:** Could not run `bench --fast` (15+ min). The number is not committed in the repo. Review's own caveat acknowledges Rice variance.

---

## §5.3 Rice-encoded sizes

### FC2-046  [§5.3]
**Claim:** Table 5 worst-case N=2^10 = 239 KB; N=2^15 = 331 KB; N=2^20 = 457 KB
**Location:** review.md line ~314–316
**Status:** VERIFIED
**Source:** `parameter/summary.txt` rows secpar=128, τ=20: N=1024 → total=239 KB; N=32768 → 331 KB; N=1048576 → 457 KB.

### FC2-047  [§5.3]
**Claim:** Rice-encoded N=2^10=201.2 KB, N=2^15=283.5 KB, N=2^20=394.4 KB
**Location:** review.md line ~314–316
**Status:** VERIFIED
**Source:** `parameter/rice_sizes.py` output: "20,1024,239,201.2; 20,32768,331,283.5; 20,1048576,457,394.4". Matches paper Table 1 values exactly.

### FC2-048  [§5.3]
**Claim:** "Rice/worst gap is 13.7–15.8% across cells"
**Location:** review.md line ~318
**Status:** VERIFIED (arithmetic)
**Source:** (239-201.2)/239=15.8%, (331-283.5)/331=14.35%, (457-394.4)/457=13.7%. Range 13.7–15.8% correct.

### FC2-049  [§5.3]
**Claim:** Paper Table 1 caption: "Lemur: measured ... for N=2^10; Rice-encoded on the corresponding cells of Table 5 for N ∈ {2^15, 2^20}. Chipmunk: theoretical, from their scripts"
**Location:** review.md line ~321–324
**Status:** VERIFIED
**Source:** Paper PDF, page near line 101–104 (pdftotext): caption text "Chipmunk: theoretical, from their scripts. Lemur: measured in implementation for N=2^10; Rice-encoded on the corresponding cells of Table 5 for N ∈ {2^15, 2^20}".

---

## §5.4 Python ↔ Rust byte-equivalence

### FC2-050  [§5.4]
**Claim:** "Python uses schoolbook multiplication for the KOTS modulus q'"
**Location:** review.md line ~348–352
**Status:** VERIFIED
**Source:** `lemur-py/ring.py:168–169`: `# (q' ≡ 17 mod 32) fail the native condition and use schoolbook`, `self._native_ntt = (q - 1) % (2 * d) == 0`; lines 214–217: schoolbook fallback for non-NTT case.

### FC2-051  [§5.4]
**Claim:** "Rust uses CRT-via-two-aux-primes NTT (aux_ntt.rs)"
**Location:** review.md line ~349–351
**Status:** VERIFIED
**Source:** `lemur-rs/src/aux_ntt.rs:34` `pub const AUX_P1: u64 = 281_474_976_694_273` and `:36 AUX_P2: u64 = 281_474_976_690_689`. Both primes ~2^48.

### FC2-052  [§5.4]
**Claim:** Byte-equality between python/rust on pp / signatures / ivrfy / avrfy / agg_attempt / aggregate / pk
**Location:** review.md line ~336–345
**Status:** UNVERIFIABLE (not re-run)
**Source:** Would require running both CLIs. The bench/test infrastructure suggests this is the intended design; codec.rs commented to match Python byte-for-byte.

---

## §5.5 Chipmunk security recomputation

### FC2-053  [§5.5]
**Claim:** "RSIS rop range across all 24 rows × 3 columns: [22.8, 39.4]"
**Location:** review.md line ~360–361
**Status:** VERIFIED
**Source:** `chipmunk_original_security_summary.txt` (24 data rows). Max value 39.4 at row 3 (secpar=112,τ=21,N=1024). Min value 22.8 at four rows: secpar=112 τ∈{21,23,24,26}, N=131072, RSIS3 column.

### FC2-054  [§5.5]
**Claim:** "Min 22.8: RSIS3 column at secpar=112, τ ∈ {21,23,24,26}, N=131072"
**Location:** review.md line ~362
**Status:** VERIFIED
**Source:** chipmunk summary lines 5,8,11,14: all show "26.9 26.9 22.8" with τ∈{21,23,24,26}, N=131072, secpar=112.

### FC2-055  [§5.5]
**Claim:** "Max 39.4: RSIS1/RSIS2 at secpar=112, τ=21, N=1024"
**Location:** review.md line ~363
**Status:** VERIFIED
**Source:** Line 3 of chipmunk summary: "112 21 1024 ... 39.4 39.4 29.5". Max value 39.4 appears only in this row.

### FC2-056  [§5.5]
**Claim:** "For secpar=128 rows specifically: range [23.1, 38.8]"
**Location:** review.md line ~364
**Status:** VERIFIED
**Source:** Rows 15–26: max 38.8 (N=1024 rows, RSIS1/2); min 23.1 (N=131072 rows, RSIS3).

---

## §6 Benchmark measurements

### FC2-057  [§6]
**Claim:** "Hardware: ARM aarch64 Linux, 11 threads, 8 GiB RAM"
**Location:** review.md line ~376
**Status:** VERIFIED
**Source:** `target/.rustc_info.json` shows `host: aarch64-unknown-linux-gnu`; bench output shows "Threads: 11".

### FC2-058  [§6]
**Claim:** "Paper baseline: 24 threads"
**Location:** review.md line ~376
**Status:** VERIFIED
**Source:** Paper Table 2 caption: "24 CPU threads".

### FC2-059  [§6]
**Claim:** "Expected slowdown vs paper: ~2.2× linear (24/11)"
**Location:** review.md line ~379
**Status:** VERIFIED (arithmetic)
**Source:** 24/11 = 2.18.

### FC2-060  [§6 Table]
**Claim:** "Key generation paper 1.3 min, measured 1.7 min, ratio 1.3×"
**Location:** review.md line ~384
**Status:** PARTIALLY CORRECT
**Source:** Paper Table 2 does NOT include "Key generation". The 1.3 min comes from `code/README.md:174` ("Key generation 1.3 min"), a 24-thread run.
**Note:** Mislabeled column header as "Paper (24 thr)"; the actual paper Table 2 doesn't list Key generation. Number itself is consistent with the artifact README, not the paper.

### FC2-061  [§6 Table]
**Claim:** Online sign (KOTS only) "304 µs"
**Location:** review.md line ~385
**Status:** UNVERIFIABLE (not reproduced); README cites "347 us" for the same measurement (`code/README.md:175`).
**Source:** No 304-µs figure anywhere in the repo. README has 347 µs.
**Note:** 304 is from a separate run; may differ. Review's own row says "implementation-internal, not a paper claim".

### FC2-062  [§6 Table]
**Claim:** "Full sign paper 1.3 min, measured 1.7 min"
**Location:** review.md line ~386
**Status:** PARTIALLY CORRECT
**Source:** README:177 "Full signing, including HVC open  1.3 min". Paper Table 2 lists "Signing (Tree-in-memory) 2 ms" and "Signing (BDS08) 4.1 ms", not a full-keygen-plus-tree time of 1.3 min.
**Note:** The 1.3 min is from artifact README's 24-thread run, not the paper.

### FC2-063  [§6 Table]
**Claim:** "Stateful sign (BDS08) paper 4.13 ms"
**Location:** review.md line ~387
**Status:** FLAG
**Source:** Paper Table 2 shows "Signing (BDS08) 4.1 ms" (PDF text line 94), and intro says "stateful signing time of roughly 4.1 ms" (PDF line 66). The 4.13 value is from `code/README.md:176`, not the paper.
**Note:** Review labels source as "Paper" but the precise 4.13 number is the artifact README's 24-thread measurement. Paper says 4.1 ms.

### FC2-064  [§6 Table]
**Claim:** Stateful sign measured 3.91 ms, ratio 0.95×
**Location:** review.md line ~387
**Status:** UNVERIFIABLE
**Source:** No `bench --fast` re-run; 3.91 ms not committed in submission.

### FC2-065  [§6 Table]
**Claim:** "Individual pre-verify, N=2^10 paper 1.67 s"
**Location:** review.md line ~388
**Status:** FLAG
**Source:** Paper Table 2 does NOT list individual pre-verify. The 1.67 s comes from `code/README.md:177`, an artifact-only sub-metric.
**Note:** Column labeled "Paper (24 thr)" is incorrect for this row.

### FC2-066  [§6 Table]
**Claim:** "Aggregate after verified inputs, N=2^10 paper 2.40 s"
**Location:** review.md line ~389
**Status:** FLAG
**Source:** Paper Table 2 does NOT list this sub-step. README:178 gives 2.40 s. Paper Table 2 row "Aggregation" 567 ms is end-to-end (includes pre-verify+aggregate combined or just the combine step — see footnote).
**Note:** Same mislabeling.

### FC2-067  [§6 Table]
**Claim:** "Secure aggregation N=2^10 paper 567 ms"
**Location:** review.md line ~390
**Status:** VERIFIED
**Source:** Paper Table 2: "Aggregation 567 ms" for N=2^10.

### FC2-068  [§6 Table]
**Claim:** "Batch verification N=2^10 paper 30.1 ms"
**Location:** review.md line ~391
**Status:** VERIFIED
**Source:** Paper Table 2: "Batch verification 30.1 ms".

### FC2-069  [§6 Table]
**Claim:** "Batch verify N=2^15 paper 812 ms"
**Location:** review.md line ~393
**Status:** VERIFIED
**Source:** Paper Table 2 row for N=2^15: "812 ms§" (zero-public-key fixture).

### FC2-070  [§6 Table]
**Claim:** measured timings (1.41 s, 79.31 ms, 1.22 s)
**Location:** review.md line ~390–393
**Status:** UNVERIFIABLE (not reproduced in this audit; bench --fast skipped)
**Source:** Not committed; consistent in magnitude with 2.2× ratio.

### FC2-071  [§6 Ratios]
**Claim:** "Stateful sign : full sign = 1:19000 (paper)"
**Location:** review.md line ~399
**Status:** VERIFIED
**Source:** Paper 4.1 ms : 1.3 min ≈ 4.1 : 78000 = 1 : 19024. Within stated rounding.
**Note:** Strictly speaking 1.3 min "full sign" comes from README, not paper Table 2 (which has no full sign). The 1:19000 ratio is plausible but cross-sources two artifacts; review attributes both to "paper".

### FC2-072  [§6 Ratios]
**Claim:** "Stateful sign : full sign = 1:26000 (measured)"
**Location:** review.md line ~399
**Status:** UNVERIFIABLE (not reproduced); arithmetic 3.91 : 102000 = 1 : 26086 consistent.

### FC2-073  [§6 Ratios]
**Claim:** "Secure aggregation : batch verify (N=2^10) = 19:1 paper"
**Location:** review.md line ~400
**Status:** VERIFIED
**Source:** Paper 567 ms / 30.1 ms = 18.84 ≈ 19:1.

### FC2-074  [§6 Ratios]
**Claim:** "= 18:1 measured"
**Location:** review.md line ~400
**Status:** UNVERIFIABLE
**Source:** Arithmetic 1410/79.31 = 17.78 ≈ 18:1 consistent.

### FC2-075  [§6 note]
**Claim:** "'Online sign : stateful sign ≈ 1:12' ratio cited in earlier review iterations was implementation-internal; paper does not report online-sign time"
**Location:** review.md line ~402–404
**Status:** VERIFIED
**Source:** Grepped paper text — no "online sign" timing. README:175 has "Online signing, KOTS only 347 us"; 4130/347 ≈ 11.9 ≈ 12:1.

### FC2-076  [§6 Sampler]
**Claim:** "Profile-specialised (indexed CDT) 1.7 µs/poly"
**Location:** review.md line ~410
**Status:** VERIFIED
**Source:** Live `bench --sampler-only` output: "Production xof_gauss_poly_with_profile (profile-specialised): 1.7 us per poly".

### FC2-077  [§6 Sampler]
**Claim:** "Generic GaussCtx 3.8 µs"
**Location:** review.md line ~411
**Status:** PARTIALLY CORRECT
**Source:** Live bench output: "xof_gauss_poly_ctx @ profile CDT (runtime-dynamic): 3.9 us per poly".
**Note:** 3.9 in this run, review says 3.8 — variance within noise.

### FC2-078  [§6 Sampler]
**Claim:** "Const-folded per-sample read (32-bit) 4.1 µs"
**Location:** review.md line ~412
**Status:** PARTIALLY CORRECT
**Source:** Live bench: "cdt_bits=32 (175 entries): 4.3 us per poly".
**Note:** 4.3 in our re-run vs 4.1 in review; within run-to-run variance.

### FC2-079  [§6 Sampler]
**Claim:** "Const-folded batched read (32-bit) 3.1 µs"
**Location:** review.md line ~413
**Status:** PARTIALLY CORRECT
**Source:** Live bench: "cdt_bits=32: 3.3 us per poly".
**Note:** Off by ~6%.

### FC2-080  [§6 Sampler]
**Claim:** "Indexed-CDT bucket dispatch gives ~2.2× speedup over generic context path"
**Location:** review.md line ~415–416
**Status:** VERIFIED
**Source:** 3.9/1.7 = 2.29, ≈ 2.2×.

---

## §7 Performance internals

### FC2-081  [§7.1]
**Claim:** "Indexed bucket dispatch (sample_cdt_indexed + cdt_hi table emitted by gen_tables.py): top 9 bits of a 32-bit sample u index into a small [lo, hi] window"
**Location:** review.md line ~446–447
**Status:** VERIFIED
**Source:** `lemur-rs/src/sample.rs:45` `fn sample_cdt_indexed(u: u32, cdt: &[u32], cdt_hi: &[u16], prefix_bits: usize) -> i64`; `:48` `let bucket = (cdf_u >> (32 - prefix_bits)) as usize`. `D256_K4_CDT_PREFIX_BITS = 9` in `tables_d256_k4.rs:134`.

### FC2-082  [§7.1]
**Claim:** "binary-search depth from ~log2(175) ≈ 8 levels to ~1–2"
**Location:** review.md line ~448
**Status:** PARTIALLY CORRECT
**Source:** Bench output confirms 175 CDT entries. log2(175) ≈ 7.45.
**Note:** Review says ≈ 8, actual 7.45 rounds up to 8. Acceptable.

### FC2-083  [§7.1]
**Claim:** "Measured speedup vs generic GaussCtx: 2.2×"
**Location:** review.md line ~449
**Status:** VERIFIED
**Source:** Same as FC2-080.

### FC2-084  [§7.1]
**Claim:** "Batched XOF reads: one xof.read(d · cdt_bytes) of ≤ 1024 bytes per poly"
**Location:** review.md line ~450–452
**Status:** VERIFIED
**Source:** `sample.rs:206` `let mut buf = [0u8; MAX_D * 4]` (MAX_D=256, cdt_bits=32 → 1024 bytes).

### FC2-085  [§7.1]
**Claim:** "Stack-allocated buffers sized to MAX_D = 256"
**Location:** review.md line ~454
**Status:** VERIFIED
**Source:** `sample.rs:12` `pub const MAX_D: usize = 256;`. Stack buffers at `:155, :206, :267`.

### FC2-086  [§7.2]
**Claim:** "HVC q ≈ 2^53, NTT-friendly (q ≡ 1 mod 2d)"
**Location:** review.md line ~462
**Status:** VERIFIED
**Source:** `parameter/summary.txt` row secpar=128 τ=20 N=1024: q = 9007199254746113 ≈ 9.007e15 ≈ 2^53.0; bit-width column says 54. (q-1) mod 512 must be 0 for d=256 NTT.

### FC2-087  [§7.2]
**Claim:** "KOTS q' ≡ 17 mod 32, not NTT-friendly at length d=256"
**Location:** review.md line ~463
**Status:** VERIFIED
**Source:** `lemur-py/ring.py:168` comment "(q' ≡ 17 mod 32) fail the native condition". Python uses the condition `(q-1) % (2*d) == 0` at line 169.

### FC2-088  [§7.2]
**Claim:** "aux_ntt.rs CRT-over-two-aux-primes ... near 2^48"
**Location:** review.md line ~463
**Status:** VERIFIED
**Source:** `aux_ntt.rs:34,36` AUX_P1=281474976694273, AUX_P2=281474976690689. log2 ≈ 47.999 ≈ 2^48.

### FC2-089  [§7.2]
**Claim:** "ntt.rs u64 Montgomery: Cooley-Tukey forward / Gentleman-Sande inverse"
**Location:** review.md line ~462
**Status:** VERIFIED
**Source:** `ntt.rs:1–3` docstring: "Montgomery reduction, forward/inverse NTT. Uses Cooley-Tukey forward and Gentleman-Sande inverse".

### FC2-090  [§7.2]
**Claim:** "CRT primes are compile-time constants (AUX_P1, AUX_P2) so LLVM specialises % p"
**Location:** review.md line ~465–466
**Status:** VERIFIED
**Source:** `aux_ntt.rs:34,36` are `pub const`.

### FC2-091  [§7.3]
**Claim:** "Three concurrent map-reduce passes in lemur_aggregate"
**Location:** review.md line ~473
**Status:** VERIFIED
**Source:** `lemur-rs/src/lemur.rs:413` `pub fn lemur_aggregate`; uses `par_iter()` at `:423, :446, :459, :473, :489`. Includes ivrfy (`try_for_each(lemur_ivrfy)` at 423), Z aggregation via `scale_mat_crt` at 446–448, opening aggregation at 489–492 with `reduce_with(|a,b| add_openings(...))`.

### FC2-092  [§7.3]
**Claim:** "par_iter().reduce_with safe because addition is commutative"
**Location:** review.md line ~482–483
**Status:** VERIFIED
**Source:** `lemur.rs:492` `.reduce_with(|a, b| add_openings(&a, &b))`. Polynomial addition over Z is commutative.

### FC2-093  [§7.3]
**Claim:** "Three branches in the source for CRT / u64-NTT / u32-NTT backends"
**Location:** review.md line ~478–479
**Status:** VERIFIED
**Source:** `lemur.rs:446, :459, :473` — three par_iter blocks (CRT, u64-NTT, u32-NTT branches around `scale_mat_crt`/`scale_mat` calls).

### FC2-094  [§7.3]
**Claim:** "24-thread paper timings become ~2.5× longer on 11 threads, within 15% of linear"
**Location:** review.md line ~483–485
**Status:** PARTIALLY CORRECT
**Source:** Linear ratio 24/11 = 2.18. The measured ratios for the parallel paper-comparable rows: 567 ms → 1.41 s = 2.49×; 30.1 → 79.31 = 2.64×. Median ~2.5×.
**Note:** 2.5× vs ideal 2.18× = 14.7% above ideal — at the edge of the "within 15%" envelope, technically passes.

---

## Summary of issues

- **FLAGs (3):**
  - FC2-063 "Stateful sign paper 4.13 ms" — paper says 4.1 ms, 4.13 is from artifact README.
  - FC2-065 "Individual pre-verify paper 1.67 s" — not in paper Table 2; from README.
  - FC2-066 "Aggregate after verified inputs paper 2.40 s" — not in paper Table 2; from README.

  All three are the same systematic issue: review's §6 table column "Paper (24 thr)" mixes paper Table 2 entries with artifact `code/README.md` entries from a 24-thread run, without distinguishing them. Headline conclusions (ratios within thread-count factor) survive.

- **PARTIALLY CORRECT (10):**
  - FC2-016 line-range off-by-2 for `_hash_to_randomizers`.
  - FC2-023 "7 test binaries" — should be 6 (5 integration + 1 lib).
  - FC2-060, FC2-062 mislabeled README numbers as paper.
  - FC2-077–079 sampler microbench numbers slightly off from a fresh re-run (3.9 vs 3.8, 4.3 vs 4.1, 3.3 vs 3.1) — run-to-run variance.
  - FC2-082 binary-search depth log2(175) ≈ 7.45, review says 8.
  - FC2-094 11-thread slowdown sits at boundary of stated 15% envelope.

- **UNVERIFIABLE (7):** kLOC counts (FC2-001/002); review's measured runtimes (FC2-064, 070); byte-equivalence diff (FC2-052); 195.8 KB realised aggregate (FC2-045); 304 µs online sign (FC2-061).

- **VERIFIED (74):** All structural code citations (KOTS/HVC/Lemur line numbers and `% self.q`, `H[i,i,0]=1`, `2*β_σ`), all `lemur sizes` output values, all Rice-encoding sizes and parameter summary cells, all Chipmunk security cell values, paper Table 1/2 values quoted, code/binary structural facts (AUX_P1/2 ≈ 2^48, MAX_D=256, CDT prefix=9 bits, sample_cdt_indexed signature, NTT comments, par_iter chains, schoolbook fallback for q'≡17 mod 32).

Overall: §4 (correctness audit) is essentially verbatim accurate to the Python source. §5 (testing/sizes) is accurate where it cites committed artifacts. §5.4 byte-equivalence not re-run but design is consistent. §5.5 Chipmunk numbers are precise. §6 has a recurring source-attribution problem: the "Paper (24 thr)" column conflates paper Table 2 cells (Aggregation 567ms, Batch verify 30.1ms, BDS08 4.1ms) with `code/README.md` artifact-only sub-metrics (Key gen 1.3 min, full sign 1.3 min, pre-verify 1.67s, aggregate after pre-verify 2.40s, stateful sign 4.13ms). The numbers themselves all match their actual sources; only the column label is misleading. §7 (performance internals) is accurate against the Rust source.
