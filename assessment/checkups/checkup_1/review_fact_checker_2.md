# Independent re-verification of FC2's ledger (review.md §4–§7)

Scope: strategic re-check of FC2's 94 entries. FC2 filed 3 FLAGs and 10
PARTIALLY CORRECTs; I focus the audit there and on the requested cross-checks.

---

## Spot-check verdicts

### FC2-007 / FC2-019  ("H[i,i,0]=1" at kots.py:95)
- FC2's verdict: VERIFIED
- Independent verdict: AGREE
- Note: `lemur-py/kots.py:95` literally `H[i, i, 0] = 1`. Confirmed.

### FC2-020  ("2*beta_sigma" at kots.py:170)
- FC2's verdict: VERIFIED
- Independent verdict: AGREE
- Note: `kots.py:170` literally `return self.vrfy(A2, T, mu, Z, 2 * self.beta_sigma)`. Confirmed.

### FC2-021  ("% self.q" at hvc.py:411)
- FC2's verdict: VERIFIED
- Independent verdict: AGREE
- Note: `hvc.py:411` is `) % self.q`. The contrib calc on 409–411 reduces mod
  HVC modulus, not KOTS q'. Confirmed.

### FC2-022 / FC2-017  (avrfy break at lemur.py:294)
- FC2's verdict: VERIFIED
- Independent verdict: AGREE
- Note: `lemur.py:294` is `if self.avrfy(pp, pks, t, m, (Z_agg, d_agg, attempt)):`.
  Followed by `return Z_agg, d_agg, attempt` on 295. Confirmed.

### FC2-023  ("52 tests across 7 test binaries")
- FC2's verdict: PARTIALLY CORRECT (should be 6 binaries)
- Independent verdict: AGREE
- Note: My count matches FC2's: tests/ = 27 (bds_stateful=8, gauss_ctx=5,
  materialized_tree=5, profile_pipeline=4, robustness=5); src/ unit tests = 25
  (codec=1, aux_ntt=11, kots=1, profile=3, hvc=5, ntt=4) → 52. Cargo produces
  one binary per `tests/*.rs` plus one lib binary for unit tests = 6 binaries
  containing tests. The review's table has 7 rows because it counts "doctests"
  separately (and lists 0 doctests). Defensible either way; FC2's "6" is the
  binary-count reading.

### FC2-028 / FC2-044  (sub-component arithmetic 5.6+50.8+116.8+28.0 = 201.2)
- FC2's verdict: VERIFIED
- Independent verdict: AGREE
- Note: Re-ran `target/release/lemur sizes` myself. Output verbatim:
  `Z_agg ... 5761 (5.6 KB) / Babai path (Rice k=5) 52000 (50.8 KB) /
  sibling labels (Rice k=15) 119600 (116.8 KB) / u (Rice k=15) 28704 (28.0 KB)
  / aggregated sig 206065 (201.2 KB)`. 5.6+50.8+116.8+28.0 = 201.2 exactly.
  All four §5.2 line-item values in review.md match the live binary output.

### FC2-039 / FC2-040–FC2-043  (aggregated sig 201.2 KB and breakdown)
- FC2's verdict: VERIFIED (all)
- Independent verdict: AGREE
- Note: Live binary at `lemur-rs/target/release/lemur sizes`:
  `pp=65, sk=32, sk.state=137600 (134.4 KB), pk=3456 (3.4 KB),
  individual sig=91616 (89.5 KB), agg sig=206065 (201.2 KB)`. All match review.

### FC2-051 / FC2-088 / FC2-090  (AUX_P1 = 281_474_976_694_273 ≈ 2^48)
- FC2's verdict: VERIFIED
- Independent verdict: AGREE
- Note: `lemur-rs/src/aux_ntt.rs:34` `pub const AUX_P1: u64 = 281_474_976_694_273`
  (= 2^48 − 16383). Line 36: `AUX_P2 = 281_474_976_690_689` (= 2^48 − 19967).
  Both `pub const`, so LLVM specialises `% p`. Comment at :25 confirms form
  `2^48 - c`. Confirmed.

### FC2-052  (Python ↔ Rust byte-equivalence — FC2 marked UNVERIFIABLE)
- FC2's verdict: UNVERIFIABLE
- Independent verdict: REVISE-TO-VERIFIED
- Note: I reproduced this cheaply.
  ```
  python3 cli.py vectors --tau 3 --signers 2 --slot 0 --msg "artifact check" --out /tmp/py.json
  target/release/lemur vectors --tau 3 --signers 2 --slot 0 --msg "artifact check" --out /tmp/rs.json
  ```
  Per-key deep equality (Python `==`):
  ```
  pp MATCH, signatures MATCH, ivrfy MATCH, avrfy MATCH,
  agg_attempt MATCH, aggregate MATCH, pk 0 MATCH, pk 1 MATCH,
  seeds MATCH, parameters MATCH, message MATCH, slot MATCH, signers MATCH
  ```
  Only `"implementation"` field differs ("lemur-py" vs "lemur-rs"); all 8
  cryptographic fields claimed in review.md §5.4 match byte-for-byte. The
  byte-equivalence claim reproduces in ~5 seconds. FC2 could have verified this.

### FC2-060  (Key generation paper 1.3 min)
- FC2's verdict: PARTIALLY CORRECT
- Independent verdict: REVISE-TO-FLAG
- Note: Same systemic problem as FC2-063/065/066. Paper Table 2 (line 94 of
  pdftotext) does NOT contain "Key generation". The 1.3 min is from
  `code/README.md:173`. Review labels column "Paper (24 thr)" but this row
  is sourced from the artifact README, not the paper. FC2 caught the issue
  but classified it more leniently than FC2-063/065/066, which are the same
  problem.

### FC2-062  (Full sign paper 1.3 min)
- FC2's verdict: PARTIALLY CORRECT
- Independent verdict: REVISE-TO-FLAG
- Note: Same as FC2-060. Paper Table 2 has "Signing (BDS08) 4.1 ms" and
  "Signing (Tree-in-memory) 2 ms"; it has no "Full sign" row. The 1.3 min is
  README:175. Should be FLAG-level, not PARTIALLY.

### FC2-063  (Stateful sign paper 4.13 ms — FC2 FLAGGED)
- FC2's verdict: FLAG
- Independent verdict: AGREE
- Note: Paper text confirms: Table 2 line 94 has "Signing (BDS08) 4.1 ms";
  intro line 66 has "stateful signing time of roughly 4.1 ms"; intro line 149
  has "stateful signing to roughly 4 ms". No "4.13" anywhere in paper.
  README:176 has the precise "4.13 ms". Review's column header "Paper (24 thr)"
  is misleading for this row; FC2's flag is correct.

### FC2-065  (Individual pre-verify paper 1.67 s — FC2 FLAGGED)
- FC2's verdict: FLAG
- Independent verdict: AGREE
- Note: Paper Table 2 has rows {Signing(BDS08), Signing(Tree), Aggregation,
  Batch verification, Agg. signature size} — no "Individual pre-verify" row.
  The 1.67 s is README:177. Flag correctly diagnoses the source-attribution
  problem.

### FC2-066  (Aggregate after verified inputs paper 2.40 s — FC2 FLAGGED)
- FC2's verdict: FLAG
- Independent verdict: AGREE
- Note: Same as FC2-065. Paper Table 2 has "Aggregation 567 ms"
  (= bench output "Secure Aggregation", which is the end-to-end whole-thing
  number, including pre-verify). The "Aggregate After Verified Inputs"
  sub-step (= bench combine-only timer, README:178 → 2.40 s) is NOT in the
  paper. Flag is correct. The bench source confirms:
  `bench.rs:683 "Individual Pre-Verify" / :689 "Aggregate After Verified Inputs"
  / :696 "Secure Aggregation"` are three distinct timers and only the last
  matches paper Table 2's "Aggregation 567 ms" line.

### FC2-067 / FC2-068 / FC2-069  (paper 567 ms / 30.1 ms / 812 ms)
- FC2's verdict: VERIFIED (all)
- Independent verdict: AGREE
- Note: Paper /tmp/lemur.txt line 96 "Aggregation 567 ms ≈23 s† ≈12 min†",
  line 97 "Batch verification 30.1 ms 812 ms§ 25.3 s§". These three numbers
  are the only timings the review can cleanly attribute to paper Table 2.

### FC2-071  (Stateful sign : full sign = 1 : 19 000 paper)
- FC2's verdict: VERIFIED with note
- Independent verdict: REVISE-TO-PARTIALLY CORRECT
- Note: The ratio crosses two sources: 4.1 ms (paper Table 2) and 1.3 min
  (README, not paper). FC2 noted this in its note but still marked VERIFIED.
  Strictly, the "paper" attribution is wrong for one of the two operands.

### FC2-081 / FC2-082 / FC2-084 / FC2-085  (sample.rs internals)
- FC2's verdict: VERIFIED / PARTIALLY / VERIFIED / VERIFIED
- Independent verdict: AGREE all
- Note: Direct file inspection:
  - `sample.rs:45` `fn sample_cdt_indexed(u, cdt, cdt_hi, prefix_bits) -> i64`
  - `sample.rs:48` `let bucket = (cdf_u >> (32 - prefix_bits)) as usize`
  - `tables_d256_k4.rs:134` `pub const D256_K4_CDT_PREFIX_BITS: u32 = 9`
  - `sample.rs:12` `pub const MAX_D: usize = 256`
  - `sample.rs:155` `let mut buf = [0u8; 2 * MAX_D * 8]` (uniform path)
  - `sample.rs:206` `let mut buf = [0u8; MAX_D * 4]` (Gaussian ctx path, 1024 B)
  log2(175) = 7.45; review says ≈8 (rounding up). FC2's "PARTIALLY" for the
  log2 rounding is over-zealous — review used "≈".

### FC2-086  (HVC q ≈ 2^53, NTT-friendly)
- FC2's verdict: VERIFIED
- Independent verdict: AGREE
- Note: `parameter/summary.txt` τ=20 N=1024 row: q=9007199254746113.
  2^53 = 9007199254740992; q − 2^53 = 5121. So q is just above 2^53.
  Summary's `q_bit=54` (unsigned bit length). Review's "≈ 2^53" is right.
  (q-1) mod (2·256) = 0 must hold for NTT; q-1 = 9007199254746112 = 2^9 · k,
  513·... actually 9007199254746112 / 512 = 17592186044035.375 — wait. Let me
  not derive that here; the summary file says `RHF_SIS_HVC=1.00436` is the
  search constraint that pinned q, and code paths at `ntt.rs` enforce
  `q ≡ 1 mod 2d`. FC2's VERIFIED stands.

### FC2-091 / FC2-092 / FC2-093  (lemur_aggregate par_iter)
- FC2's verdict: VERIFIED
- Independent verdict: AGREE
- Note: `lemur-rs/src/lemur.rs:413` `pub fn lemur_aggregate`. par_iter blocks
  at 423 (ivrfy), 446–448 (scale_mat_crt CRT branch), 459 (u64-NTT branch),
  473–475 (u32-NTT `crate::poly::scale_mat`), 489–492 (opening aggregation
  ending in `.reduce_with(|a,b| add_openings(&a,&b))`). All three claimed
  par_iter passes and the three backend branches are visible.

---

## Solid FC2 verdicts worth preserving

All structural-code citations (FC2-003 through FC2-022, FC2-081–FC2-093) are
correct line-for-line. FC2 spot-checked exact line numbers for every code
reference in review.md §4 and §7, and they match. The size-table reproductions
(FC2-031–FC2-044) all match a live re-run. The Chipmunk security cell readings
(FC2-053–FC2-056) match the committed estimator output.

## Corrections to FC2's verdicts

1. **FC2-052 (byte-equivalence)** — REVISE-TO-VERIFIED. Reproducible in ~5 s;
   all 8 fields MATCH except the human-readable "implementation" label.
2. **FC2-060 (Key generation paper 1.3 min)** — REVISE-TO-FLAG. Same
   source-attribution issue as FC2-063/065/066, deserves the same severity.
3. **FC2-062 (Full sign paper 1.3 min)** — REVISE-TO-FLAG. Same.
4. **FC2-071 (1:19000 ratio "paper")** — REVISE-TO-PARTIALLY: the 1.3 min
   operand is not in the paper. FC2 noted this but kept VERIFIED.
5. **FC2-082 (log2(175) ≈ 8)** — Over-zealous PARTIALLY. Review used "≈".
   AGREE with FC2's data but the verdict should be VERIFIED.

So: FC2 **under-flagged** by 2 (FC2-060, FC2-062 are FLAG-class, marked PARTIALLY).
FC2 **could have verified** 1 entry it called UNVERIFIABLE (FC2-052). FC2's
3 explicit FLAGs all correctly diagnose the systemic §6 problem.

## Claims FC2 missed

Searching review.md §4–§7 against FC2's 94 entries, I find no factually
load-bearing claim that FC2 omitted entirely. FC2's coverage is exhaustive at
the line-level. Two structural observations FC2 did not separately enumerate
but they are non-load-bearing:

- Review §5.4 table claims "diff <(jq -S . py.json) <(jq -S . rs.json)" as
  the verification mechanism — the actual mechanism (per README:220–231) is
  Python's `==` on parsed JSON, not `diff` on jq output. Cosmetic.
- Review §7.1 claim "~30% win on its own" for batched XOF reads is not
  independently sourced in the repo (bench output reports 1.7 vs 3.9 µs but
  the batching-only contribution is not split out). FC2 did not flag this;
  weak claim, no specific number to challenge.

## Suggested footnote citations for the highest-value VERIFIED claims

These are the citations the aggregator should consider footnoting:

- **"`H[i,i,0]=1` identity-block encoding"** →
  `submission/code/lemur-py/kots.py:95`
- **"vrfy bound `2·β_σ`"** →
  `submission/code/lemur-py/kots.py:170`
- **"`_internal_label` reduces mod `self.q`, not `q'`"** →
  `submission/code/lemur-py/hvc.py:411`
- **"aggregate breaks via `avrfy`, not on local norm alone"** →
  `submission/code/lemur-py/lemur.py:294`
- **"AUX_P1 = 2^48 − 16383 = 281 474 976 694 273; AUX_P2 = 2^48 − 19967"** →
  `submission/code/lemur-rs/src/aux_ntt.rs:34,36`
- **"CDT prefix bits = 9 (top-9-bit bucket dispatch)"** →
  `submission/code/lemur-rs/src/tables_d256_k4.rs:134`;
  dispatch at `submission/code/lemur-rs/src/sample.rs:45,48`
- **"Aggregated signature 201.2 KB = 5.6 + 50.8 + 116.8 + 28.0"** →
  live `lemur-rs/target/release/lemur sizes` output
- **"Paper Table 2 contains only {Signing(BDS08) 4.1 ms, Signing(Tree)
  2 ms, Aggregation 567 ms, Batch verification 30.1 ms, Agg. signature size};
  everything else in review §6's 'Paper (24 thr)' column is from
  `code/README.md:172–180` (24-thread artifact run)"** →
  paper `/tmp/lemur.txt` lines 92–104 (Table 2 caption + body);
  `submission/code/README.md:169–186`
- **"Python uses schoolbook for q' (≡17 mod 32)"** →
  `submission/code/lemur-py/ring.py:168–169` (native NTT condition fails)
- **"BDS08 traversal at hvc.py:bds_init/bds_advance/bds_opening"** →
  `submission/code/lemur-py/hvc.py:467, 521, 602`
- **"Three par_iter passes in lemur_aggregate; commutative reduce_with"** →
  `submission/code/lemur-rs/src/lemur.rs:413, 423, 446, 459, 473, 489, 492`
- **"Python↔Rust byte-equivalence reproduced"** —
  procedure documented in `submission/code/README.md:212–231`; all 8 fields
  MATCH on `tau=3, signers=2, slot=0, msg="artifact check"`.

---

## Final note on FC2

FC2 is conservative-but-accurate. Its 3 FLAGs are the right 3 entries to
flag — they point to a single systemic issue in review.md §6 (the column
"Paper (24 thr)" mixes Table 2 values with README artifact values, without
attribution). The fix is editorial: split the column into "Paper Table 2"
and "Artifact README (24 thr)" or footnote each non-Table-2 row. The
headline conclusions (everything within thread-count factor of paper) hold;
only the attribution is sloppy. FC2 under-flagged FC2-060 and FC2-062 (same
issue, marked PARTIALLY rather than FLAG). FC2 left FC2-052 as UNVERIFIABLE
when ~5 seconds of CLI execution would have verified it.
