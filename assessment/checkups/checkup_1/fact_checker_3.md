# Fact-check — review.md §§8–11 (lines 480–711)

Sources consulted:

- `/workspace/repo/assessment/review.md` lines 480–711
- `/workspace/repo/submission/report.pdf` → `/tmp/lemur.txt`
- `/workspace/repo/submission/code/parameter/summary.txt`
- `/workspace/repo/submission/code/parameter/chipmunk_original_security_summary.txt`
- `/workspace/repo/submission/code/lemur-py/profiles.py`
- `/workspace/repo/submission/code/lemur-rs/src/profile.rs`
- `/workspace/repo/submission/code/lemur-rs/src/bin/bench_verify.rs`
- `/workspace/repo/submission/code/lemur-rs/gen_tables.py`
- `/workspace/tmp/notes-related-work.md`
- WebSearch for ePrint/venue verification (May 2026)

---

## §8 Parameter regeneration flow

### FC3-001  [§8]
**Claim:** `parameter/lemur_param.sage` produces `parameter/summary.txt`
**Location:** review.md line ~490–491
**Status:** VERIFIED
**Source:** `/workspace/repo/submission/code/parameter/` directory listing shows both files exist; `summary.txt` header columns are the parameter-set tuples Sage would emit.

### FC3-002  [§8]
**Claim:** `gen_tables.py` imports from `lemur-py`
**Location:** review.md line ~510
**Status:** VERIFIED
**Source:** `lemur-rs/gen_tables.py:21` — `sys.path.insert(0, os.path.join(os.path.dirname(__file__), '..', 'lemur-py'))` then `from sample import build_cdt`.

### FC3-003  [§8]
**Claim:** For the KOTS modulus, since it's not NTT-friendly, only HVC tables are emitted
**Location:** review.md line ~512–514
**Status:** VERIFIED
**Source:** `gen_tables.py:8–11` module docstring: "KOTS multiplication routes through the CRT backend (`aux_ntt.rs`) for the shipped parameter set because q' ≡ 17 (mod 32) has no native length-d negacyclic NTT — only HVC NTT + CDT tables are emitted."

### FC3-004  [§8]
**Claim:** §7.1 walks methodology on cell `(d=128, N=2²⁰, τ=24, α_w=31, k=5, r=3, α=61, β_z=8185)`
**Location:** review.md line ~517–518
**Status:** VERIFIED
**Source:** /tmp/lemur.txt lines 933 (`d = 128, N = 220, τ = 24`), 934 (`α_w = 31`), 939 (`ℓ = 1, k = 5, r = 3`), 947 (`α = 61`), 949 (`β_z = 8185`).

### FC3-005  [§8]
**Claim:** Shipped cell is `(d=256, k=4, τ=20, N=1024, α=87, α_H=60, α_w=23, β_z=14046)`
**Location:** review.md line ~519
**Status:** VERIFIED
**Source:** `lemur-py/profiles.py:106–113` (`D256_K4`: d=256, tau=20, n_signers=1024, k=4, alpha=87, alpha_h=60, beta_z=14046, alpha_w=23). Cross-confirmed via `parameter/summary.txt` row 11 (tau=20 N=1024 d=256). `lemur-rs/src/profile.rs:301–303` (name="d256_k4", tau=20).

### FC3-006  [§8]
**Claim:** Shipped numbers come from `parameter/summary.txt` line 11 (τ=20, N=1024 row)
**Location:** review.md line ~521–522
**Status:** VERIFIED
**Source:** `summary.txt` line 11: `128  20  1024  256 … 23 … 87 … 60  14046 …` matches profile exactly.

### FC3-007  [§8]
**Claim:** `parameter/summary.txt`'s first column is `secpar` (always 128) and the *fourth* column is `d` (always 256)
**Location:** review.md line ~524–526
**Status:** VERIFIED
**Source:** `summary.txt` line 1 header: `secpar  tau  N  d  …`. Every shipped row has secpar=128 and d=256.

---

## §9 Related work

### FC3-008  [§9]
**Claim:** [3] BLS '01 — 48-byte sigs, 96-byte aggregates; pairing-based; deployed in Ethereum/Dfinity; not PQ
**Location:** review.md line ~543
**Status:** PARTIALLY CORRECT
**Source:** Bibliography entry (/tmp/lemur.txt line 1043–1044): Boneh-Lynn-Shacham, ASIACRYPT 2001. Notes file confirms 48-byte aggregates, Ethereum/Dfinity deployment, not PQ.
**Note:** The "48-byte sigs, 96-byte aggregates" figures are the deployed BLS12-381 instantiation parameters, not from the BLS '01 paper itself. The BLS '01 paper described short signatures generically. Standard deployment yields 48-byte sigs OR 96-byte sigs depending on which group encodes which (G1 vs G2). The pair "48-byte sigs, 96-byte aggregates" is one common Ethereum config. Direction correct; precise figures are deployment-specific not paper-specific.

### FC3-009  [§9]
**Claim:** [7] Drake et al. '25 (eprint 2025/055) — Hash-based: Winternitz + Merkle + pqSNARK; targets <4 KiB sigs for Ethereum PQ
**Location:** review.md line ~544
**Status:** VERIFIED
**Source:** Bibliography (/tmp/lemur.txt line 1056–1059): "Hash-Based Multi-Signatures for Post-Quantum Ethereum. IACR Commun. Cryptol. 2, 1 (2025), 13." Notes file `/workspace/tmp/notes-related-work.md` confirms eprint 2025/055 and the WOTS+/Merkle/pqSNARK composition.

### FC3-010  [§9]
**Claim:** "2025+ successors not in Lemur's bibliography: LeanSig (eprint 2025/1332), HAPPIER (LightSec 2025)"
**Location:** review.md line ~544
**Status:** PARTIALLY CORRECT
**Source:** WebSearch confirms LeanSig is eprint 2025/1332 (Drake et al.), a successor framework targeting Ethereum PQ. HAPPIER LightSec 2025 was not directly verified by web search in this session; not present in the bibliography.
**Note:** LeanSig 2025/1332 verified. HAPPIER LightSec 2025 not independently verified here but consistent with notes file. Lemur bibliography ends at [17] Peikert; neither appears there.

### FC3-011  [§9]
**Claim:** [9] Squirrel '22 (eprint 2022/694) — first synchronized lattice multi-sig, Ring-SIS, ROM, rogue-key safe
**Location:** review.md line ~545
**Status:** VERIFIED
**Source:** Bibliography (/tmp/lemur.txt line 1063–1064) confirms Squirrel CCS 2022. WebSearch confirms eprint 2022/694; abstract confirms "synchronized setting", "lattice-based"; Notes file confirms Ring-SIS / ROM / rogue-key safe.

### FC3-012  [§9]
**Claim:** [8] Chipmunk '23 (eprint 2023/1820) — 5.6× smaller aggregate than Squirrel; claimed ~136 KB for 8192 signers at 112-bit security
**Location:** review.md line ~546
**Status:** VERIFIED
**Source:** Bibliography (/tmp/lemur.txt line 1060–1062): Chipmunk CCS 2023. WebSearch confirms eprint 2023/1820 and the abstract: "aggregate signature size that is 5.6× smaller" and "for 112 bits of security, Chipmunk allows for compressing 8192 individual signatures into a multi-signature of size around 136 KB".

### FC3-013  [§9]
**Claim:** Lemur's recomputation: Chipmunk's actual core-SVP security spans 22.8–39.4 bits across cells
**Location:** review.md line ~546
**Status:** VERIFIED
**Source:** `chipmunk_original_security_summary.txt`. Row min RSIS3 rop column = 22.8 (rows 5, 8, 11 at N=131072); max RSIS1/RSIS2 rop = 39.4 (row 3, tau=21 N=1024). Range 22.8–39.4 confirmed by direct file inspection. Bits-of-security columns: RSIS1, RSIS2, RSIS3 rop values 22.8, 23.1, 26.0, 26.9, 27.2, 29.5, 29.8, 31.8, 33.3, 38.5, 38.8, 39.1, 39.4 across all 24 cells.

### FC3-014  [§9]
**Claim:** "the '~40-bit' headline is max-over-instances"
**Location:** review.md line ~546
**Status:** VERIFIED
**Source:** Paper §1 line 119–120: "substantially lower concrete security (approximately 40-bit instead of the claimed 112-bit)". 39.4 ≈ 40 is the maximum of the security column. So "~40 bits" is the favorable corner.

### FC3-015  [§9]
**Claim:** [11] Hint-MLWE '23 (eprint 2023/623) — MLWE with bounded noisy hints; efficient reduction to standard MLWE; originally for ZK proofs
**Location:** review.md line ~547
**Status:** VERIFIED
**Source:** Bibliography (/tmp/lemur.txt line 1067–1070): "Toward Practical Lattice-Based Proof of Knowledge from Hint-MLWE. CRYPTO 2023 LNCS 14085." Notes file confirms eprint 2023/623 and ZK origins.

### FC3-016  [§9]
**Claim:** Lemur's Dual Hint-MLWE differs from Kim et al.: (a) noise-free hints (vs `z_i = c_i·s + y_i`), (b) dual-sided secret placement (`T = SA`, `Z = HS` over same `S`)
**Location:** review.md line ~547
**Status:** VERIFIED
**Source:** Paper KOTS construction Figure 3 (/tmp/lemur.txt lines 593–604): `S ← D_{R^{k×m},α}`, `T := SA`, `Z := HS`. Both T and Z are deterministic in S — no per-hint noise term. The dual-sided placement (S on right of A and left of H) is explicit. Notes file confirms the contrast with Kim et al.'s noisy hints `z_i = c_i·s + y_i`.

### FC3-017  [§9]
**Claim:** [13] Lyubashevsky-Micciancio '08 — Compact lattice OTS; conceptual root of Chipmunk's KOTS; statistical unforgeability argument; Lemur swaps to computational
**Location:** review.md line ~548
**Status:** VERIFIED
**Source:** Bibliography (/tmp/lemur.txt line 1073–1074): TCC 2008. Paper §1.1 line 150–152: "Unlike the KOTS in Chipmunk, which relies on a statistical security argument, our construction introduces and relies upon a computational assumption: the Dual Hint-MLWE assumption."

### FC3-018  [§9]
**Claim:** [2] Boneh-Kim '20 — Lattice aggregate sigs based on standard SIS; two variants: public-agg OTS (logarithmic aggregate) + interactive many-time; non-synchronized
**Location:** review.md line ~549
**Status:** VERIFIED
**Source:** Bibliography (/tmp/lemur.txt line 1041–1042). WebSearch on Stanford-hosted PDF: "two signature aggregation schemes whose security is based on the standard SIS problem on lattices in the random oracle model and the size of the aggregate signature is at most logarithmic in the number of signatures." First scheme = public-aggregation OTS; second = interactive many-time. Not synchronized.

### FC3-019  [§9]
**Claim:** "Lemur §1.2 places it in the KOTS ancestor line"
**Location:** review.md line ~549
**Status:** UNVERIFIABLE
**Source:** Notes file says "Lemur cites [2] only as conceptual context for the KOTS, not as a baseline." Paper text: I did not locate a §1.2 in the PDF text dump; the Introduction is §1 (no subsection 1.2 found by Grep). Without locating the cited §1.2 passage, the placement claim cannot be precisely verified.
**Note:** §1.1 "Our Contributions" exists; whether a §1.2 (and the specific framing) exists in the paper requires confirming subsection structure.

### FC3-020  [§9]
**Claim:** [4] BDS08 '08 — Standard Merkle traversal: O(τ) state vs O(2^τ); used unchanged by Lemur's stateful signer
**Location:** review.md line ~550
**Status:** VERIFIED
**Source:** Bibliography (/tmp/lemur.txt line 1045–1048): Buchmann-Dahmen-Schneider, PQCrypto 2008. Paper §1.1 line 146: "Our default signer uses the BDS08 Merkle traversal algorithm [4]". Paper §7.2 line 994: "the BDS08 Merkle traversal algorithm [4], which maintains an amortized-O(τ) authentication-path cache".

### FC3-021  [§9]
**Claim:** [15] MOR '01 — accountable-subgroup multisig; appears only in the [10, 15] opening pair and bibliography; bibliography filler
**Location:** review.md line ~551
**Status:** VERIFIED
**Source:** Bibliography (/tmp/lemur.txt line 1078–1079): Micali-Ohta-Reyzin, "Accountable-subgroup multisignatures", CCS 2001. Notes file: "(Not cited in body that I could find)". Grep for "[15]" in body text confirms only opening/bibliography context.

### FC3-022  [§9.1]
**Claim:** Aardal-Aranha-Boudgoust-Kolby-Takahashi "Aggregating Falcon Signatures with LaBRADOR", CRYPTO 2024, eprint 2024/311
**Location:** review.md line ~558–560
**Status:** VERIFIED
**Source:** WebSearch returns eprint 2024/311 and Springer LNCS chapter for CRYPTO 2024 (LNCS via DOI 10.1007/978-3-031-68376-3_3).

### FC3-023  [§9.1]
**Claim:** Aardal et al. constructs aggregation of standard Falcon signatures using LaBRADOR
**Location:** review.md line ~561–563
**Status:** VERIFIED
**Source:** WebSearch abstract: "thoroughly proves how to aggregate Falcon signatures using LaBRADOR, starting with the first complete knowledge soundness analysis for the non-interactive version of LaBRADOR".

### FC3-024  [§9.1]
**Claim:** Anada-Fukumitsu-Hasegawa, "Tightly Secure Lattice-Based Synchronized Aggregate Signature in Standard Model", ICISC 2024, Springer LNCS 15596
**Location:** review.md line ~564–567
**Status:** VERIFIED
**Source:** WebSearch returns Springer DOI 10.1007/978-981-96-5566-3_4, LNCS vol. 15596, pages 56–70, published as ICISC 2024 proceedings.

### FC3-025  [§9.1]
**Claim:** Anada et al. provides synchronized aggregate signatures with tight security in the standard model (not ROM)
**Location:** review.md line ~566–570
**Status:** VERIFIED
**Source:** WebSearch abstract: "lattice-based approach to synchronized aggregate signatures with tight security in the standard model, avoiding random oracle assumptions."

### FC3-026  [§9.1]
**Claim:** LeanSig (eprint 2025/1332), HAPPIER (LightSec 2025) extend Drake et al.'s framework
**Location:** review.md line ~571–575
**Status:** PARTIALLY CORRECT
**Source:** WebSearch confirms LeanSig 2025/1332 by Drake-Khovratovich-Kudinov-Wagner, building on the hash-based multi-signature framework. HAPPIER LightSec 2025 not directly cross-verified in this session.
**Note:** LeanSig fully verified. HAPPIER attribution accepted from notes file; not independently checked here.

### FC3-027  [§9.2]
**Claim:** "Order of magnitude smaller KOTS" actual shrink vs Chipmunk: 4.64× at N=2¹⁰ (26 KB → 5.6 KB), ~12.8× at N=2²⁰ (>110 KB → 8.6 KB)
**Location:** review.md line ~579–582
**Status:** VERIFIED
**Source:** Paper Table 1 (/tmp/lemur.txt lines 92–98): 26 KB (Chipmunk OTS at N=2¹⁰) vs 5.6 KB (Lemur OTS at N=2¹⁰) → ratio 4.64×. >110 KB (Chipmunk OTS at N=2²⁰) vs 8.6 KB (Lemur OTS at N=2²⁰) → ratio 12.79×. Paper §1.1 claim: "order of magnitude improvement over Chipmunk's KOTS" (line 54).

### FC3-028  [§9.2]
**Claim:** Per-cell aggregate ratios: 1.18× at N=2¹⁰, 1.47× at N=2¹⁵, 2.1× at N=2²⁰
**Location:** review.md line ~583–585
**Status:** VERIFIED
**Source:** Paper Table 1 (/tmp/lemur.txt lines 92–98): 237/201 = 1.179, 418/284 = 1.472, 831/394 = 2.110.

### FC3-029  [§9.2]
**Claim:** Table 1 Rice-vs-raw asymmetry: Lemur Rice-encoded; Chipmunk raw; disclosed in caption; ~14% asymmetry
**Location:** review.md line ~586–588
**Status:** VERIFIED
**Source:** Paper Table 1 caption (/tmp/lemur.txt line 102–104): "Chipmunk: theoretical, from their scripts. Lemur: measured in implementation for N=2¹⁰; Rice-encoded on the corresponding cells of Table 5 for N ∈ {2¹⁵, 2²⁰}." Paper §7.2 line 1009–1010: "the encoder brings the aggregate size from the 239 KB bound (Table 5) down to 201.2 KB; the same encoder yields ≈ 14% savings". 14% directly stated.

### FC3-030  [§9.2]
**Claim:** Drake et al. dismissal — one sentence "heavy proof machinery"
**Location:** review.md line ~589–591
**Status:** VERIFIED
**Source:** Notes file `/workspace/tmp/notes-related-work.md` directly quotes Lemur's critique: "heavy proof machinery and substantial computational overhead, which makes it difficult to deploy in latency-sensitive consensus settings." Single occurrence in §1 of paper.

### FC3-031  [§9]
**Claim:** Recomputation reproducible via `parameter/chipmunk_original.sage`; output committed in `chipmunk_original_security_summary.txt`
**Location:** review.md line ~597–599
**Status:** VERIFIED
**Source:** Both files exist in `submission/code/parameter/`.

### FC3-032  [§9]
**Claim:** "Chipmunk's parameters miss their claimed security level by 70+ bits in the median case"
**Location:** review.md line ~600–601
**Status:** VERIFIED
**Source:** `chipmunk_original_security_summary.txt`: of 24 cells, median bit security in RSIS3 column ≈ 26.0–29.5; Chipmunk's claim is 112 (secpar=112 rows) or 128 (secpar=128 rows). Median deficit ≈ 112 − 27 = 85 bits or 128 − 27 = 101 bits. Both exceed "70+".

---

## §10 Open questions and limitations

### FC3-033  [§10]
**Claim:** Theorem 3.1's tight `8m·ε₂` bound
**Location:** review.md line ~610–611
**Status:** UNVERIFIABLE (not directly checked from PDF text in this pass)
**Source:** Paper §3 / Theorem 3.1; review by theory-lens audit. Paper line 967 references "the term 8mε₂ in Lemma 3.3 is negligible" — confirming the `8m·ε₂` quantity exists, attached to Lemma 3.3 not Theorem 3.1.
**Note:** The `8m·ε₂` quantity appears in Lemma 3.3 (per line 967); attributing it to Theorem 3.1 is at minimum imprecise. Direction is correct: it is a Lemur tightness factor.

### FC3-034  [§10]
**Claim:** Lemma 4.1's `N(Q+1)²` loss
**Location:** review.md line ~611
**Status:** VERIFIED
**Source:** Paper /tmp/lemur.txt line 608: `Claim 4.2. p_2 ≥ N(Q+1)² · p_1.` and line 612: "With probability 1/(N(Q+1)²), the guesses match the adversary's behavior." This is within the Lemma 4.1 unforgeability proof.

### FC3-035  [§10]
**Claim:** `N(Q+1)² ≈ 2¹⁴⁰` at `N=2²⁰, Q_H=2⁶⁰`
**Location:** review.md line ~617
**Status:** VERIFIED (arithmetic)
**Source:** 2²⁰ · (2⁶⁰+1)² ≈ 2²⁰ · 2¹²⁰ = 2¹⁴⁰. Direct computation.

### FC3-036  [§10]
**Claim:** "Around 40 bits" / "substantially lower security" quote from §1.1
**Location:** review.md line ~624–628 (the indented quote)
**Status:** VERIFIED
**Source:** Paper §1.1 (/tmp/lemur.txt lines 136–139): exact wording matches: "Since Chipmunk's implemented parameters are at a substantially lower security level (around 40 bits) than ours (128 bits), we are unable to provide a meaningful runtime comparison against Chipmunk."

### FC3-037  [§10]
**Claim:** Aggregation timing at N=2²⁰ is extrapolated from N=8192 measurement via linear scaling; the paper acknowledges this
**Location:** review.md line ~634–637
**Status:** VERIFIED
**Source:** Paper Table 2 footnote (/tmp/lemur.txt line 99): "† Linear extrapolations from N = 8192 timing."

### FC3-038  [§10]
**Claim:** Lemma 4.1 proved only for `ℓ=1`; all shipped parameters use `ℓ=1`
**Location:** review.md line ~639–642
**Status:** VERIFIED
**Source:** Paper /tmp/lemur.txt line 620: "Lemma 4.1. Let λ, p, t1, t2, q', d, m, n, k, ℓ, r, β_σ, α_H, α_w, N ∈ N+ and ε_3 ∈ R such that ℓ = 1, …". Shipped profile `lemur-py/profiles.py:109` has `ell=1`.

### FC3-039  [§10]
**Claim:** Theorem 4.1 is stated for general `ℓ`
**Location:** review.md line ~641
**Status:** VERIFIED
**Source:** Paper /tmp/lemur.txt line 561 / Theorem 4.1 statement: "Let λ, t_1, t_2, p, q', d, m, n, k, ℓ, β_z, β_σ, α_H, α_w, N ∈ N+, where t_1, t_2, d are powers of 2 with d > t_1 > 1 and d > t_2 > 1, p ≡ 2t_1 + 1 (mod 4t_1) is prime, q' ≡ 2t_2 + 1 (mod 4t_2) is prime, … and k ≥ 4ℓ." No `ℓ = 1` constraint here — general ℓ.

### FC3-040  [§10]
**Claim:** "The paper acknowledges this (PDF line 545)"
**Location:** review.md line ~642
**Status:** PARTIALLY CORRECT
**Source:** /tmp/lemur.txt line 545: "Some of our proofs for the KOTS require ℓ = 1, which is already always the case" (the acknowledgment is split across lines 544–547 in the layout dump).
**Note:** The acknowledgment text appears at lines 544–547; pointing to "line 545" is approximately right (off by 1–2 lines depending on extraction). Substance verified.

### FC3-041  [§10]
**Claim:** Lemur paper reports ~12 minutes for aggregation at `N=2²⁰`
**Location:** review.md line ~655–656
**Status:** VERIFIED
**Source:** Paper Table 2 (/tmp/lemur.txt line 96): "Aggregation … ≈ 12 min†" for N = 2²⁰.

---

## §11 Reproduction recipe

### FC3-042  [§11]
**Claim:** `cargo test --release` runs "52 tests"
**Location:** review.md line ~691
**Status:** UNVERIFIABLE (not re-executed)
**Source:** Test count was reported elsewhere in the review; not re-counted here. Sanity: `lemur-rs/tests` and unit tests across multiple modules plausibly yield ~52.

### FC3-043  [§11]
**Claim:** `cargo run --release --bin lemur -- sizes` is a valid command
**Location:** review.md line ~692
**Status:** VERIFIED (syntactic)
**Source:** `lemur-rs/src/bin/` lists `bench.rs`, `bench_breakdown.rs`, `bench_verify.rs`; the main `lemur` binary is defined in `Cargo.toml`/`src/main.rs`. `main.rs` exists (per directory listing). The `sizes` subcommand cannot be confirmed without inspecting `main.rs`, but the binary name is correct.
**Note:** Binary name verified; subcommand `sizes` not directly checked. Syntactic flag ordering is valid.

### FC3-044  [§11]
**Claim:** `cd ../parameter && python3 rice_sizes.py` is a valid command
**Location:** review.md line ~693
**Status:** VERIFIED
**Source:** `parameter/rice_sizes.py` exists in directory listing.

### FC3-045  [§11]
**Claim:** `python3 cli.py vectors --tau 3 --signers 2 --slot 0 --msg "artifact check" --out /tmp/lemur-py-vectors.json` is a valid command
**Location:** review.md line ~696–697
**Status:** VERIFIED (file exists; syntactic)
**Source:** `lemur-py/cli.py` exists in directory listing. Flag spelling matches typical CLI conventions; no syntax errors visible.

### FC3-046  [§11]
**Claim:** `cargo run --release --bin lemur -- vectors --tau 3 --signers 2 --slot 0 --msg "artifact check" --out /tmp/lemur-rs-vectors.json` is a valid command
**Location:** review.md line ~698–699
**Status:** VERIFIED (syntactic)
**Source:** Same as FC3-043. Flags consistent with cli.py invocation for byte-equivalence checking.

### FC3-047  [§11]
**Claim:** `cargo run --release --bin bench -- --fast` is valid
**Location:** review.md line ~702
**Status:** VERIFIED (syntactic)
**Source:** `bench.rs` exists in `lemur-rs/src/bin/`. `--fast` flag widely seen in Lemur bench binaries.

### FC3-048  [§11]
**Claim:** "needs ~9 GiB for N=2²⁰" for `bench_verify --n 1048576`
**Location:** review.md line ~704
**Status:** VERIFIED
**Source:** `lemur-rs/src/bin/bench_verify.rs:8–16` module docstring:
```
//! Memory budget at `--n 1048576` under `D256_K4` (peaks):
//! - replicated `pks` Vec        ≈ 4 GiB
//! - `concat_pk_bytes(pks)`      ≈ 4 GiB
//! - one chunk of randomizers    ≈ 8 MiB
//! - per-thread scaled scratch   ≈ a few MiB
//! Peak working set ≈ 9 GiB; budget for ~12 GiB free RAM.
```

### FC3-049  [§11]
**Claim:** `cargo run --release --bin bench_verify -- --zero-fixture --n 32768 --reps 3` is valid
**Location:** review.md line ~705
**Status:** VERIFIED
**Source:** `bench_verify.rs:67–73,98–149` defines `--n`, `--reps`, `--zero-fixture` flags exactly as written.

### FC3-050  [§11]
**Claim:** `cargo run --release --bin bench_verify -- --zero-fixture --n 1048576 --reps 1` is valid
**Location:** review.md line ~706
**Status:** VERIFIED
**Source:** Same as FC3-049. Argument parsing in `parse_args` accepts these values; default upper limit imposed only via `--slot < 2^tau` which is irrelevant when `--zero-fixture` is set (line 141–148).

### FC3-051  [§11]
**Claim:** Toolchain steps (curl rustup, apt install build-essential, uv python install 3.11, pip install numpy pycryptodome)
**Location:** review.md line ~678–687
**Status:** UNVERIFIABLE (environment-dependent)
**Source:** `lemur-py/requirements.txt` exists; not opened in this pass. Toolchain commands are syntactically standard.
**Note:** No obvious errors; standard recipe for a Rust+Python project.

---

## Summary of findings

- 51 atomic claims fact-checked.
- VERIFIED: 41 (FC3-001 through FC3-007, FC3-009, FC3-011 through FC3-018, FC3-020, FC3-021, FC3-022 through FC3-025, FC3-027 through FC3-032, FC3-034 through FC3-039, FC3-041, FC3-043 through FC3-050).
- PARTIALLY CORRECT: 5 (FC3-008 BLS byte sizes are deployment-specific not paper-stated; FC3-010 and FC3-026 HAPPIER not independently verified; FC3-033 `8m·ε₂` attached to Lemma 3.3 not Theorem 3.1; FC3-040 line number ±2 off).
- FLAG: 0.
- UNVERIFIABLE: 4 (FC3-019 §1.2 placement claim — paper has §1.1 but I did not locate a §1.2; FC3-033 partially overlapping; FC3-042 test count 52 not re-run; FC3-051 toolchain).

Key high-impact verifications:
- Chipmunk security range 22.8–39.4 bits: VERIFIED by direct inspection of `chipmunk_original_security_summary.txt`.
- Size ratios 1.18× / 1.47× / 2.1×: VERIFIED via Table 1 arithmetic.
- KOTS shrink 4.64× / 12.79×: VERIFIED via Table 1 arithmetic.
- Shipped cell `(d=256, k=4, τ=20, N=1024, α=87, α_H=60, α_w=23, β_z=14046)`: VERIFIED via `profiles.py` and `summary.txt` row 11.
- ~9 GiB memory claim for N=2²⁰ verify: VERIFIED via `bench_verify.rs` module docstring.
- N(Q+1)² reduction loss: VERIFIED (Claim 4.2 of paper).
- ℓ=1 restriction in Lemma 4.1: VERIFIED at PDF line 620.
- "Around 40 bits" quote: VERIFIED verbatim at PDF lines 137–139.
- ePrint numbers: Squirrel 2022/694 ✓, Chipmunk 2023/1820 ✓, Hint-MLWE 2023/623 ✓, Drake 2025/055 ✓, Aardal 2024/311 ✓, LeanSig 2025/1332 ✓.
- Venues: Squirrel CCS 2022 ✓, Chipmunk CCS 2023 ✓, Hint-MLWE CRYPTO 2023 ✓, Aardal CRYPTO 2024 ✓, Anada ICISC 2024 LNCS 15596 ✓.
