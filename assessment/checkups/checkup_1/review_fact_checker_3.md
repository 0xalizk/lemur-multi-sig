# Review of fact_checker_3.md (§§8-11, 51 entries)

Independent re-verification of a strategic sample. Sources: PDF dump
`/tmp/lemur.txt`, parameter regen artifacts, WebSearch.

## Spot-check verdicts

### FC3-005 / FC3-006 (shipped cell vs summary.txt row 11)
- FC3's verdict: VERIFIED
- Independent verdict: AGREE
- Note: `summary.txt` file-line 11 reads
  `128  20  1024  256 … 23 … 14046 … 87 … 60 …`, which exactly maps to
  `lemur-py/profiles.py:106-113` (`d=256, tau=20, n_signers=1024, k=4,
  ell=1, m=9, alpha=87, alpha_h=60, beta_z=14046, alpha_w=23`).

### FC3-004 (§7.1 worked example tuple)
- FC3's verdict: VERIFIED
- Independent verdict: AGREE
- Note: PDF lines 933 (`d=128, N=2²⁰, τ=24`), 935 (`α_w=31`), 939
  (`ℓ=1, k=5, r=3`), 947 (`α=61`), 949 (`β_z=8185`). Every field in
  the review's worked-example tuple is present verbatim in §7.1.

### FC3-009 / FC3-011 / FC3-012 / FC3-015 / FC3-022 (eprint numbers)
- FC3's verdict: VERIFIED
- Independent verdict: AGREE
- Note: Three were re-checked at random via WebSearch:
  - 2022/694 = Squirrel (Fleischhacker, Simkin, Zhang). CONFIRMED.
  - 2023/623 = Hint-MLWE (Kim, Lee, Seo, Song), CRYPTO 2023 LNCS
    14085. CONFIRMED.
  - 2025/055 = Drake, Khovratovich, Kudinov, Wagner,
    "Hash-Based Multi-Signatures for Post-Quantum Ethereum", IACR CiC.
    CONFIRMED.

### FC3-010 / FC3-026 (HAPPIER LightSec 2025)
- FC3's verdict: PARTIALLY CORRECT / UNVERIFIABLE
- Independent verdict: REVISE-TO-VERIFIED
- Note: WebSearch returns Springer chapter
  `10.1007/978-3-032-15541-2_1` titled
  "HAPPIER: Hash-Based, Aggregatable, Practical Post-quantum
  Signatures Implemented Efficiently with Risc0", LightSec 2025
  proceedings. The paper exists and matches the framing
  (post-quantum aggregate alternative to BLS, hash-based, 2-3 MB sigs,
  scales to 2^16 signers on a laptop). FC3's hedge was unnecessary.

### FC3-019 (Lemur §1.2 KOTS-ancestor placement)
- FC3's verdict: UNVERIFIABLE
- Independent verdict: REVISE-TO-VERIFIED
- Note: §1.2 "Technical Overview" exists at PDF line 189. The exact
  sentence is at lines 211-213: "Chipmunk's KOTS closely follows the
  line of work of Lyubashevsky-Micciancio [13] and Boneh-Kim [2]".
  FC3 missed it because it grep'd for "1.2" but the heading is
  laid out as "1.2    Technical Overview" with a tab/whitespace
  separator that may have eluded a literal-string search. Review.md's
  framing — that the paper places [2] in the KOTS-ancestor line —
  is directly supported.

### FC3-024 / FC3-025 (Anada-Fukumitsu-Hasegawa, ICISC 2024 LNCS 15596)
- FC3's verdict: VERIFIED
- Independent verdict: AGREE
- Note: Springer DOI `10.1007/978-981-96-5566-3_4` confirmed,
  LNCS vol. 15596 "Information Security and Cryptology – ICISC 2024".
  Title and "standard model" framing match.

### FC3-033 (8m·ε₂ from Theorem 3.1 vs Lemma 3.3)
- FC3's verdict: PARTIALLY CORRECT (says it's a Lemma 3.3 figure, not
  Theorem 3.1's)
- Independent verdict: REVISE-TO-VERIFIED — FC3's correction is wrong.
- Note: PDF lines 423-434 are the **statement of Theorem 3.1**, and
  the conclusion (line 434) reads literally:
  `Adv^{DualHintMLWE}_{q',m,n,k,ℓ,α,T_{α_H}}(A) ≤ Adv^{MLWE}_{q',m,n,k-ℓ,α₀}(B) + 8mε₂`.
  The `8m·ε₂` term is in Theorem 3.1's bound. Lemma 3.3 is one of the
  hybrid-distance lemmas used inside the proof and contributes the
  `8m·ε₂` summand, but the **bound itself appears in the theorem
  statement**, so review.md's attribution is correct. FC3 over-corrected.

### FC3-048 (~9 GiB memory at --n 1048576)
- FC3's verdict: VERIFIED
- Independent verdict: AGREE
- Note: `bench_verify.rs:8-15` module docstring explicitly states
  "Peak working set ≈ 9 GiB; budget for ~12 GiB free RAM" with line-
  item breakdown (4 GiB pks + 4 GiB concat_pk_bytes + 8 MiB chunks +
  per-thread scratch). Exact match for the review claim.

### FC3-049 / FC3-050 (bench_verify flag spellings)
- FC3's verdict: VERIFIED
- Independent verdict: AGREE
- Note: `bench_verify.rs:98-149` (parse_args) accepts exactly
  `--n`, `--tau`, `--unique`, `--slot`, `--reps`, `--zero-fixture`.
  The recipe commands `--zero-fixture --n 32768 --reps 3` and
  `--zero-fixture --n 1048576 --reps 1` are both syntactically valid;
  the `--slot < 2^tau` guard is skipped under `--zero-fixture`
  (lines 141-148), so `--slot 0` is also unconditionally fine.

### FC3-036 ("around 40 bits" quote from §1.1)
- FC3's verdict: VERIFIED
- Independent verdict: AGREE
- Note: PDF lines 136-139 contain verbatim: "Since Chipmunk's
  implemented parameters are at a substantially lower security level
  (around 40 bits) than ours (128 bits), we are unable to provide a
  meaningful runtime comparison against Chipmunk." Exact match to the
  block-quote in review.md §10.

### FC3-038 / FC3-039 (Lemma 4.1 ℓ=1 vs Theorem 4.1 general ℓ)
- FC3's verdict: VERIFIED
- Independent verdict: AGREE
- Note: Verified by Grep on PDF dump. Lemma 4.1 statement requires
  `ℓ = 1`; Theorem 4.1 statement omits that constraint. Shipped
  profile has `ell=1` (`profiles.py:109`).

### FC3-013 / FC3-014 (Chipmunk 22.8-39.4 range, 40-bit max)
- FC3's verdict: VERIFIED
- Independent verdict: AGREE
- Note: `chipmunk_original_security_summary.txt` columns RSIS1/2/3 rop
  inspected directly; min 22.8 (rows at N=131072), max 39.4 (row 3,
  τ=21 N=1024). 39.4 ≈ 40 → max-over-instances claim corroborated.

### FC3-027 / FC3-028 (size-ratio arithmetic)
- FC3's verdict: VERIFIED
- Independent verdict: AGREE
- Note: From Table 1 in /tmp/lemur.txt: 26/5.6 ≈ 4.64, 110/8.6 ≈ 12.79
  (KOTS); 237/201 ≈ 1.18, 418/284 ≈ 1.47, 831/394 ≈ 2.11 (aggregate).
  Arithmetic checks pass.

---

## Solid FC3 verdicts worth preserving

- FC3-001..007 (parameter regen flow) — file existence and column
  layout claims are tight and accurate.
- FC3-013/014/032 (Chipmunk 22.8-39.4 bits, ~40-bit headline is the
  maximum, ~70-bit median deficit) — the central field-level claim
  of the paper, properly anchored to
  `chipmunk_original_security_summary.txt`.
- FC3-027/028/029 (size-ratio arithmetic and Rice-vs-raw asymmetry) —
  all checked directly against Table 1 and §7.2.
- FC3-034/035 (Lemma 4.1 `N(Q+1)²` loss, ≈ 2^140 at N=2^20, Q_H=2^60).
- FC3-038/039/041 (Lemma 4.1 ℓ=1 restriction, Theorem 4.1 general ℓ,
  ~12-min aggregation extrapolation footnote).
- FC3-048 (bench_verify ~9 GiB budget): module docstring is explicit.
- FC3-049/050 (bench_verify argparse): flag names verified verbatim.

## Corrections to FC3's verdicts

1. **FC3-019**: REVISE UNVERIFIABLE → VERIFIED. §1.2 exists (PDF
   line 189); the Boneh-Kim sentence is at lines 211-213. FC3
   should have found it.

2. **FC3-033**: REVISE PARTIALLY CORRECT → VERIFIED. The `8m·ε₂`
   bound is in Theorem 3.1's statement (PDF line 434). Lemma 3.3
   contributes one summand, but review.md is right to attribute the
   tight bound to the theorem. FC3 misread the relationship between
   theorem-statement and supporting lemma.

3. **FC3-010 / FC3-026**: REVISE PARTIALLY CORRECT/UNVERIFIABLE →
   VERIFIED. HAPPIER LightSec 2025 exists with Springer DOI
   `10.1007/978-3-032-15541-2_1`; title "HAPPIER: Hash-Based,
   Aggregatable, Practical Post-quantum Signatures Implemented
   Efficiently with Risc0".

4. **FC3-008** (BLS deployment-figures caveat): FC3's PARTIAL is a
   pedantic over-classification. The review.md statement is a
   parenthetical taxonomy entry, not a factual claim about the
   BLS '01 paper's contents — the numbers describe what BLS-12-381
   deployments produce in practice. Mark VERIFIED with the noted
   nuance, not PARTIAL.

5. **FC3-040** (line 545 acknowledgment): FC3's PARTIAL on a "±1-2
   line" offset is over-strict; line numbers in a pdftotext dump
   are inherently approximate. AGREE the substance is verified; the
   PARTIAL flag is noise.

## Claims FC3 missed in §8-§11

- **FC3 did not verify** the Table 1 raw numbers `7 KB / 152 KB /
  159 KB` for the `(τ=12, N=2^10)` row that's the basis for the
  KOTS-shrink and aggregate-ratio arithmetic. These are present in
  Table 5 (PDF lines 957-984) and `summary.txt`'s "sig size /
  open size / total size" columns (row 3 of `summary.txt` is the
  τ=12 N=1024 cell: `7 KB | 152 KB | 159 KB`). The arithmetic FC3
  verified rests on these — worth a direct anchor.

- **FC3 did not verify** Table 2's "12 min† at N=2²⁰" — the † is
  defined at PDF line 99 (footnote: "Linear extrapolations from
  N = 8192 timing"). FC3 cited line 99 but did not surface the full
  ~12 min figure for the §10 bullet.

- **FC3 did not check** the "bench" binary's `--fast` flag is
  actually defined (FC3 marked VERIFIED-syntactic without opening
  `bench.rs`). Not high-value, but the strict version should
  verify by reading `bench.rs`.

- **FC3 did not engage** with the contrast that the shipped profile
  uses `n=4` (HVC-side), whereas summary.txt row 11 also shows `n=4`
  for that cell — review.md doesn't claim this but a strict reviewer
  could check it.

## Suggested footnote citations (highest-value verified claims)

For the final review:

1. **Shipped cell** `(d=256, k=4, τ=20, N=1024, α=87, α_H=60, α_w=23,
   β_z=14046)`:
   → `submission/code/lemur-py/profiles.py:106-113` and
     `submission/code/parameter/summary.txt` line 11.

2. **§7.1 worked-example cell** `(d=128, N=2^20, τ=24, α_w=31, k=5,
   r=3, α=61, β_z=8185)`:
   → report.pdf §7.1, /tmp/lemur.txt lines 933, 935, 939, 947, 949.

3. **"~40 bits vs 128 bits" runtime-comparison quote**:
   → report.pdf §1.1, /tmp/lemur.txt lines 136-139.

4. **Chipmunk 22.8-39.4 bit core-SVP range**:
   → `submission/code/parameter/chipmunk_original_security_summary.txt`
     columns RSIS1/2/3 rop, 24 rows.

5. **Theorem 3.1 tight 8m·ε₂ bound**:
   → report.pdf Theorem 3.1 conclusion, /tmp/lemur.txt line 434:
     `Adv^DualHintMLWE ≤ Adv^MLWE + 8mε₂`.

6. **Lemma 4.1 N(Q+1)² loss**:
   → report.pdf Claim 4.2, /tmp/lemur.txt lines 608, 612; ℓ=1
     restriction at line 620.

7. **9 GiB memory budget at N=2^20**:
   → `submission/code/lemur-rs/src/bin/bench_verify.rs:8-15`.

8. **bench_verify argparse**:
   → `submission/code/lemur-rs/src/bin/bench_verify.rs:98-149`.

9. **Boneh-Kim placement in KOTS ancestor line (Lemur §1.2)**:
   → report.pdf §1.2 "Key-Homomorphic One-Time Signatures" paragraph,
     /tmp/lemur.txt lines 211-213.

10. **HAPPIER LightSec 2025**:
    → Springer chapter
      `https://doi.org/10.1007/978-3-032-15541-2_1` (LightSec 2025
      proceedings).

11. **Anada-Fukumitsu-Hasegawa ICISC 2024 LNCS 15596**:
    → Springer chapter
      `https://doi.org/10.1007/978-981-96-5566-3_4`.

12. **LeanSig 2025/1332**:
    → `https://eprint.iacr.org/2025/1332` (Drake, Khovratovich,
      Kudinov, Wagner).

13. **Aardal et al. (Falcon+LaBRADOR) CRYPTO 2024**:
    → `https://eprint.iacr.org/2024/311`; Springer LNCS chapter
      DOI `10.1007/978-3-031-68376-3_3`.

---

## Summary

- 51 FC3 entries, 13 spot-checked in depth, plus randomized eprint
  cross-checks.
- **3 corrections**: FC3-019 (§1.2 does exist; missed by literal
  grep), FC3-033 (8m·ε₂ IS in Theorem 3.1's statement), FC3-010/026
  (HAPPIER LightSec 2025 confirmed).
- **2 over-strict PARTIALs** to downgrade: FC3-008 (BLS deployment
  figures), FC3-040 (line-number ±1).
- The substantive verdicts (parameter cells, size ratios, security
  range, runtime quote, memory budget, argparse) are all correct.
- The most load-bearing claim of the §§8-11 review block — the
  Chipmunk 22.8-39.4 bit range — is rock-solid and reproducible from
  the committed artifact.
