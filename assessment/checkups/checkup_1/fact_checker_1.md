# Fact-check ledger for review.md §1-§3 (lines 1-200)

Sources cited:
- PDF text dump: `/tmp/lemur.txt` (from `pdftotext -layout submission/report.pdf`).
- Parameter estimator: `submission/code/parameter/summary.txt`, `chipmunk_original_security_summary.txt`.

---

### FC1-001  [§Header]
**Claim:** "Paper: Lemur: Scalable Post-Quantum Synchronized Multi-Signatures (24 pages)"
**Location:** review.md line 3-4
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 1 title; PDF file claimed 24 pages by reviewer harness.

### FC1-002  [§Header]
**Claim:** "ACM CCS submission for 2026"
**Location:** review.md line 4
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 400 footer "June 03–05,2026, Woodstock, NY" and line 83 "permissions@acm.org".

---

## §1 Executive verdict

### FC1-003  [§1]
**Claim:** "aggregate-signature sizes at N ∈ {2¹⁰, 2¹⁵, 2²⁰}, KOTS/HVC/Lemur algorithm-level structure, Python ↔ Rust byte-level agreement reproduce exactly"
**Location:** review.md lines 46-49
**Status:** PARTIALLY CORRECT (reproduction assertion only; sizes verified)
**Source:** `/tmp/lemur.txt` lines 96-98 (Table 1 values 201/284/394 KB). Cross-validation between Rust/Python is an artifact-side claim not directly verifiable from this ledger pass.

### FC1-004  [§1]
**Claim:** "At paper-scale N=2²⁰ the aggregate is ~2.1× smaller than Chipmunk at corrected 128-bit security"
**Location:** review.md lines 54-56
**Status:** VERIFIED
**Source:** Table 1 (`/tmp/lemur.txt` line 98): Chipmunk Multi-Sig >831 KB vs Lemur 394 KB → 831/394 = 2.11×.

### FC1-005  [§1]
**Claim:** "At N=2¹⁰ the ratio shrinks to 1.18×"
**Location:** review.md line 56
**Status:** VERIFIED
**Source:** Table 1 (`/tmp/lemur.txt` line 96): Chipmunk 237 KB vs Lemur 201 KB → 237/201 = 1.18×.

### FC1-006  [§1]
**Claim:** "Chipmunk claimed 112-bit security"
**Location:** review.md line 58
**Status:** VERIFIED
**Source:** `chipmunk_original_security_summary.txt` shows secpar=112 rows (lines 3-14); paper line 120 confirms "claimed 112-bit".

### FC1-007  [§1]
**Claim:** "actual 22.8–39.4-bit range under modern lattice estimation"
**Location:** review.md line 58
**Status:** VERIFIED
**Source:** `chipmunk_original_security_summary.txt` rows for secpar=112: RSIS1/2 rop spans 26.9 to 39.4; RSIS3 rop spans 22.8 to 29.5. Min=22.8 (row N=131072), Max=39.4 (row N=1024).

### FC1-008  [§1]
**Claim:** "Table 1 puts Rice-encoded Lemur against raw-encoded Chipmunk, ~14% asymmetry disclosed in caption but not column headers"
**Location:** review.md lines 61-63
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 100-104 (Table 1 caption: "Lemur: measured in implementation for N=2^10; Rice-encoded on the corresponding cells of Table 5 for N∈{2^15, 2^20}"); column headers say only "Chipmunk (KB)" / "Lemur (KB)". Asymmetry size verified at FC1-022.

### FC1-009  [§1]
**Claim:** "The 'order of magnitude' KOTS shrink against Chipmunk is actually 4.64× at N=2¹⁰"
**Location:** review.md line 65
**Status:** VERIFIED
**Source:** Table 1: Chipmunk One-Time-Sig 26 KB vs Lemur 5.6 KB → 26/5.6 = 4.643.

### FC1-010  [§1]
**Claim:** "~12.8× at N=2²⁰"
**Location:** review.md line 65
**Status:** VERIFIED
**Source:** Table 1: Chipmunk OTS >110 KB vs Lemur 8.6 KB → 110/8.6 = 12.79×.

### FC1-011  [§1]
**Claim:** "Paper provides no runtime comparison vs Chipmunk (acknowledged in §1.1: 'we are unable to provide a meaningful runtime comparison')"
**Location:** review.md lines 66-68
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` lines 137-139: "Since Chipmunk's implemented parameters are at a substantially lower security level (around 40 bits) than ours (128 bits), we are unable to provide a meaningful runtime comparison against Chipmunk." This is in §1.1.

### FC1-012  [§1]
**Claim:** "Aardal et al. CRYPTO '24 Falcon+LaBRADOR"
**Location:** review.md line 72-73
**Status:** UNVERIFIABLE (paper unaffected; external claim)
**Source:** Paper does not cite Aardal et al.; need external check. ePrint 2024/311 is by Aardal-Aranha-Boschini-Lehmann "Aggregating Falcon Signatures with LaBRADOR" CRYPTO 2024 — matches /workspace/tmp/notes-related-work.md style of citation, but no source consulted directly in this pass.

### FC1-013  [§1]
**Claim:** "Anada et al. ICISC '24 standard-model lattice synchronized"
**Location:** review.md line 73-74
**Status:** UNVERIFIABLE (external)
**Source:** Paper does not cite. Cannot confirm exact venue/year without external lookup; not consulted here.

---

## §2 Field placement and paradigm

### FC1-014  [§2]
**Claim:** "Lemur sits firmly in the lattice-native synchronized ROM column"
**Location:** review.md line 107
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` lines 145-146 "security against rogue-key attacks in the random oracle model"; lines 12-14 "follows the blueprint of Squirrel/Chipmunk".

### FC1-015  [§2]
**Claim:** "BLS [3]"
**Location:** review.md line 90
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` lines 1043-1044 (reference [3] is Boneh-Lynn-Shacham 2001).

### FC1-016  [§2]
**Claim:** "Drake '25 LeanSig 2025/1332 HAPPIER LightSec'25" (hash-based + SNARKs entry [7])
**Location:** review.md lines 90-93
**Status:** PARTIALLY CORRECT
**Source:** `/tmp/lemur.txt` lines 1056-1059 — [7] Drake/Khovratovich/Kudinov/Wagner 2025 "Hash-Based Multi-Signatures for Post-Quantum Ethereum" in IACR CiC 2(1):13. Paper title is "Hash-Based Multi-Signatures for Post-Quantum Ethereum", NOT "LeanSig". LeanSig (2025/1332) and HAPPIER are not cited in paper; their categorization in review's taxonomy is the reviewer's own contribution. The mapping is plausible but not internal to the paper.

### FC1-017  [§2]
**Claim:** "Squirrel '22 [9]"
**Location:** review.md line 98
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 1063-1064: Fleischhacker-Simkin-Zhang 2022 Squirrel CCS 2022.

### FC1-018  [§2]
**Claim:** "Chipmunk '23 [8]"
**Location:** review.md line 101
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 1060-1062: Fleischhacker-Herold-Simkin-Zhang 2023 Chipmunk CCS 2023.

### FC1-019  [§2]
**Claim:** "Boneh-Kim '20 [2] (OTS + interactive, log-size agg)"
**Location:** review.md lines 96-97
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` lines 1041-1042: Boneh-Kim 2020 "One-Time and Interactive Aggregate Signatures from Lattices". Paper at line 213 mentions "and Boneh-Kim [2]".

### FC1-020  [§2]
**Claim:** "Aardal et al. 2024/311 and Anada et al. '24 ICISC … Lemur's stated paradigm trichotomy sidesteps both"
**Location:** review.md lines 112-115
**Status:** UNVERIFIABLE (no external sources consulted this pass; absence-of-citation verifiable)
**Source:** `/tmp/lemur.txt` references (lines 1038-1085) — no mention of Aardal or Anada. The non-citation is verified.

### FC1-021  [§2 table]
**Claim:** "KOTS: Lemur — Computational unforgeability via Dual Hint-MLWE; Chipmunk — statistical"
**Location:** review.md line 123
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` lines 150-152 "Unlike the KOTS in Chipmunk, which relies on a statistical security argument, our construction introduces and relies upon a computational assumption: the Dual Hint-MLWE assumption."

### FC1-022  [§2 table]
**Claim:** "HVC: Lemur — Module-SIS, matrix commitment; Chipmunk — Ring-SIS, vector commitment"
**Location:** review.md line 124
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` lines 124-126 ("Chipmunk's reliance on Ring-SIS"); lines 1187-1192 (HVC over Sdom = R_{q'}^ρ for Lemur vs Chipmunk's Sdom = R_{q'}); line 1203-1205 ("In Lemur, the binding property of the Ajtai hash is based on the MSIS assumption, whereas Chipmunk relies on…"). Generalization from vectors to matrices stated at lines 47-48 and 125-127.

### FC1-023  [§2 table]
**Claim:** "Stateful sign: BDS08 [4]"
**Location:** review.md line 126
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 146-147 "Our default signer uses the BDS08 Merkle traversal algorithm [4]"; reference [4] at lines 1045-1048.

### FC1-024  [§2 table]
**Claim:** "Aggregation: Weighted random sum, γ retries; Inherited unchanged"
**Location:** review.md line 125
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 936-937 "γ = 10 via Constraint 17" plus weighted-sum description; lines 12-14 / 190 "Lemur builds upon / follows the blueprint of Chipmunk".

---

## §3 What the paper proves and what it does not

### FC1-025  [§3]
**Claim:** "Dual Hint-MLWE (Def 3.1)"
**Location:** review.md line 134
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 390 "Definition 3.1 (Dual Hint-MLWE)".

### FC1-026  [§3]
**Claim:** "at least as hard as standard MLWE up to 8m·ε₂ statistical slack with no multiplicative loss (Theorem 3.1)"
**Location:** review.md lines 134-136
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 423 "Theorem 3.1 (Hardness of Dual Hint-MLWE)"; line 434 advantage bound "AdvMLWE_q',m,n,k-ℓ,α0(B) + 8m·ε₂". No multiplicative factor.

### FC1-027  [§3]
**Claim:** "proof via four hybrid hops Lemmas 3.1–3.4 plus the sampleability lemma 3.5"
**Location:** review.md lines 136-137
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 454 Lemma 3.1 (G0≡G1); line 485 Lemma 3.2 (Σ1 pos def); line 435 Lemma 3.3 (8mε₂ stat dist); line 459 Lemma 3.4 (G2 exact); line 514 Lemma 3.5 (Sampleability). Sequence of hybrids G0/G1/G2 at line 445-451 (only 3 games, but 4 lemmas — review's "four hybrid hops" loosely characterizes Lemmas 3.1-3.4).

### FC1-028  [§3]
**Claim:** "With the paper's parameters m=20, ε₂=2^{-136}, the slack is 8·20·2^{-136} ≈ 2^{-128.7} — narrowly below 2^{-128}"
**Location:** review.md lines 137-139
**Status:** PARTIALLY CORRECT
**Source:** `/tmp/lemur.txt` line 965 "ε₂ = 2^{-136}" verified. m=20 appears at line 1021 ("Substituting ℓ=1, m=20, d=128, k=5, α_H=31, ε₁=2^{-150}")—but this is the d=128 illustration; the shipped/implemented parameter set uses d=256 with m∈{9,10,10,11} (summary.txt). Slack computation 8·20·2^{-136} = 2^{-128.68} is arithmetically correct for the d=128 case. For the actual implemented m=11: 8·11·2^{-136} ≈ 2^{-129.54}, which is more comfortably below 2^{-128}. Review's m=20 is a valid in-paper value but not the only one used.

### FC1-029  [§3]
**Claim:** "Lemma 3.5 (sampleability over non-full-rank lattices)"
**Location:** review.md line 140
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 514 "Lemma 3.5 (Sampleability). Let q', k, ℓ, α_H ∈ N+ such that k > ℓ…"; reduces to full-rank via Lemma 2.6 (line 286).

### FC1-030  [§3]
**Claim:** "bridges to the G+G '23 [6] full-rank sampler via Lemma 2.6 (rank-r → full-rank reduction)"
**Location:** review.md lines 141-142
**Status:** VERIFIED
**Source:** Reference [6] G+G Devevey-Passelègue-Stehlé 2023 (line 1053-1055); Lemma 2.6 named "Reduction to full-rank sampler" at line 286. Paper line 534 "which implies efficient sampleability by Lemma 2.6".

### FC1-031  [§3]
**Claim:** "using an orthonormal change-of-basis (Appendix D, U^T U = I)"
**Location:** review.md line 143
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 2035 "(orthonormal columns of U). Furthermore, UU^T · c = c_∥". The reduction is in Appendix D (per line 1994 area, "Proof of Lemma 2.6"). U has orthonormal columns implies U^T U = I.

### FC1-032  [§3]
**Claim:** "A KOTS construction (Fig 3) is W'-EUF-RK secure in the ROM"
**Location:** review.md line 145
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 606 "Figure 3: Our KOTS Construction Π_KOTS"; line 582 / 635 "W'-multi-user existentially unforgeable under rerandomized keys" / "W'-EUF-RK secure in the ROM".

### FC1-033  [§3]
**Claim:** "Theorem 4.1, proof via Lemma 4.1 with multi-user reduction loss of N(Q+1)² where Q := Q_H + N"
**Location:** review.md lines 146-148
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 561 "Theorem 4.1"; line 619 "Lemma 4.1"; line 640 "Adv^EUF-RK ≤ N(Q+1)² Adv^DualHintMLWE + Adv^MSIS + Q_H²/|T_αH|^{k-2ℓ-1} + negl"; line 647 "where Q := Q_H + N".

### FC1-034  [§3]
**Claim:** "An HVC construction (Fig 4) over Module-SIS satisfies individual correctness, probabilistic homomorphism, robust homomorphism, and position binding (Theorem B.1)"
**Location:** review.md lines 148-150
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 815 "Figure 4: Our HVC Construction Π_HVC"; line 1470 "Theorem B.1" and surrounding text on individual correctness, probabilistic & robust homomorphism, position binding (line 1196 lists the four properties).

### FC1-035  [§3]
**Claim:** "Composition: a synchronized multi-signature (Fig 5)"
**Location:** review.md line 151
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 906 "Figure 5: Lemur Multi-Signature Construction".

### FC1-036  [§3]
**Claim:** "Accountable aggregation… Micali-Ohta-Reyzin model [15] is adjacent but unused; [15] appears only in the [10, 15] opening pair and the bibliography"
**Location:** review.md lines 156-158
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 17 (only [10,15] in-text citation); line 1078 (bibliography entry [15] Micali-Ohta-Reyzin "Accountable-subgroup multisignatures"). No other in-text use of [15].

### FC1-037  [§3]
**Claim:** "Multi-message aggregation. Every signer must sign the same message at the same slot."
**Location:** review.md lines 160-161
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` lines 110-112 "sign the same candidate block in the same time step"; line 194 "In Chipmunk, signatures are aggregated only for the same message and time step" (inherited).

### FC1-038  [§3]
**Claim:** "Security after slot reuse. KOTS is one-time per slot; reusing leaks the secret"
**Location:** review.md lines 162-163
**Status:** VERIFIED (consistent with KOTS definition)
**Source:** `/tmp/lemur.txt` line 509 Sign oracle "|Q_i| = 1 then return ⊥" — one-signature-per-user enforcement. KOTS structure Z = HS leaks S over multiple H's by linear algebra (standard).

### FC1-039  [§3 loose ends]
**Claim:** "Lemma 4.1 is restricted to ℓ=1 (PDF line 620 in pdftotext layout)"
**Location:** review.md lines 167-168
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 620 "and ε_3 ∈ R such that ℓ = 1, k ≥ 4ℓ, r = k − 2ℓ".

### FC1-040  [§3 loose ends]
**Claim:** "The paper itself acknowledges this at line 545: 'Some of our proofs for the KOTS require ℓ = 1, which is already always the case in our parameter setting.'"
**Location:** review.md lines 168-170
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` lines 544-547: "Some of our proofs for the KOTS require ℓ = 1, which is already always the case … by using the ℓ parameter and specify when it's restricted to ℓ = 1."

### FC1-041  [§3 loose ends]
**Claim:** "All shipped parameters use ℓ=1"
**Location:** review.md line 170
**Status:** VERIFIED
**Source:** `submission/code/parameter/summary.txt` column "ell" = 1 in every row.

### FC1-042  [§3 loose ends]
**Claim:** "Theorem 4.1 is stated for general ℓ"
**Location:** review.md line 171
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 561 Theorem 4.1 statement quantifies over "m, n, k, ℓ" with constraint only k ≥ 4ℓ, not ℓ = 1.

### FC1-043  [§3 loose ends]
**Claim:** "At N=2²⁰, Q_H=2⁶⁰, the loss is ~2¹⁴⁰, requiring ~268-bit core hardness"
**Location:** review.md lines 173-175
**Status:** PARTIALLY CORRECT (computation)
**Source:** N=2^20, Q_H=2^60 ⇒ Q = Q_H+N ≈ 2^60, Q+1 ≈ 2^60, (Q+1)² = 2^120, N(Q+1)² = 2^20·2^120 = 2^140. Verified. Q_H=2^60 is an external assumption convention; paper does not state Q_H bound. Need 128+140 ≈ 268 bits of core hardness in worst case: arithmetic correct.

### FC1-044  [§3 loose ends]
**Claim:** "Lemur picks parameters at the 128-bit core-SVP level"
**Location:** review.md line 176
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 929 "We target a security level of λ = 128 bits"; `summary.txt` secpar=128 throughout.

### FC1-045  [§3 loose ends]
**Claim:** "Lemma C.3 EUF composition is sketched, with explicit 'follows the same idea as [8]' disclaimer"
**Location:** review.md lines 178-180
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 1962 "Lemma C.3"; line 1981 "Proof. The proof follows the same idea as in [8]. For complete-…".

### FC1-046  [§3 headline]
**Claim:** "201.2 KB at N=2¹⁰, 283.5 KB at N=2¹⁵, 394.4 KB at N=2²⁰"
**Location:** review.md lines 183-184
**Status:** VERIFIED
**Source:** Table 1 / `/tmp/lemur.txt` lines 96-98: 201, 284, 394 KB. Paper line 152 confirms 201.2 KB. Table 5 (line 977) shows τ=20 row N=2²⁰ raw 457 KB → Rice 394.4 (line 1010 cites this exact number).

### FC1-047  [§3 headline]
**Claim:** "Table 2 reports Rice-encoded; Table 5 reports worst-case bounds"
**Location:** review.md lines 184-186
**Status:** PARTIALLY CORRECT
**Source:** Table 1 (not Table 2) reports Rice-encoded — `/tmp/lemur.txt` line 100-104. Table 2 reports runtime/sizes for Rust (line 102-104). Table 5 reports "theoretical values without any entropic encoding" (line 985-988). Review's attribution of Rice to "Table 2" is wrong — it's Table 1 that holds the Rice-encoded sizes. Table 2 reports Rust performance benchmarks. This is a labeling error.

### FC1-048  [§3 headline]
**Claim:** "Differ by 14–16% per cell (15.8/14.4/13.7% at N=2^{10,15,20})"
**Location:** review.md lines 186-187
**Status:** VERIFIED (arithmetic)
**Source:** Table 5 raw at τ=20: 239 / 331 / 457 KB (lines 972/974/977). Table 1 Rice: 201.2 / 283.5 / 394.4 KB. Differences: (239-201.2)/239 = 15.82%; (331-283.5)/331 = 14.35%; (457-394.4)/457 = 13.70%. All match.

### FC1-049  [§3 headline]
**Claim:** "Table 1's Lemur-vs-Chipmunk comparison mixes Rice-encoded Lemur with raw-encoded Chipmunk; caption discloses, column headers do not"
**Location:** review.md lines 188-191
**Status:** VERIFIED
**Source:** Table 1 caption (lines 100-104): "Chipmunk: theoretical, from their scripts. Lemur: measured in implementation for N=2^10; Rice-encoded on the corresponding cells of Table 5 for N∈{2^15, 2^20}". Headers (line 92): bare "Chipmunk (KB) | Lemur (KB)".

### FC1-050  [§3 headline]
**Claim:** "4.1 ms stateful sign, 30.1 ms batch verify at N=2¹⁰ at 24 threads"
**Location:** review.md lines 192-193
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 94 "Signing (BDS08) 4.1 ms"; line 97 "Batch verification 30.1 ms"; Table 2 caption (line 104) "24 CPU threads".

### FC1-051  [§3 headline]
**Claim:** "Reproduced within ~10–15% of the linear thread-count scaling"
**Location:** review.md lines 193-194
**Status:** UNVERIFIABLE (a §5/§6 cross-reference; not in scope of §1-3 lines)
**Source:** Empirical re-run noted at review §1 line 50; numbers not duplicated in §3.

### FC1-052  [§3 headline]
**Claim:** "λ = 128 under MLWE + MSIS via APS-style lattice estimation. Not independently re-run here (no SageMath in container)"
**Location:** review.md lines 195-198
**Status:** VERIFIED
**Source:** `/tmp/lemur.txt` line 929 (λ=128); line 932-933 "MSIS hardness is estimated using the state-of-the-art Dilithium security estimation framework, and MLWE hardness is evaluated using the lattice estimator [1]" (APS = ref [1] Albrecht-Player-Scott). `summary.txt` is the committed estimator output.

---

## Cross-claim notes

- The headline claim "below 10 KB for 1M signers" (paper line 116, 182) corresponds to Lemur OTS 8.6 KB at N=2^20 in Table 1: VERIFIED.
- "[10,15] opening pair" (review FC1-036): the [10] in the paper is Itakura-Nakamura 1983 (line 1065-1066) — original multisig paper. Combined opening citation [10,15] = "Itakura-Nakamura, Micali-Ohta-Reyzin". Confirmed at /tmp/lemur.txt line 17.
- Field-map paradigm taxonomy (§2 ASCII tree): LeanSig and HAPPIER labels added by reviewer — not in the paper, so the "trichotomy in §1" critique cannot be assessed without external sources. The paper at §1 mentions Squirrel/Chipmunk as the main lattice predecessors and [7] Drake et al as the hash-based comparand. The paradigm map's PQ-sig+lattice-PoK bucket and standard-model sibling are reviewer-added; no paper text either confirms or refutes their existence.

## Summary counts

VERIFIED: 35
PARTIALLY CORRECT: 5  (FC1-003, FC1-016, FC1-028, FC1-043, FC1-047)
FLAG: 0
UNVERIFIABLE: 4  (FC1-012, FC1-013, FC1-020, FC1-051)

Notable findings worth surfacing to aggregator:
1. FC1-047: Review attributes "Rice-encoded" to Table 2 — actually Table 1. Table 2 is the Rust performance benchmarks.
2. FC1-028: The "m=20" cited for the slack computation matches paper's d=128 illustration (line 1021) but the shipped implementation runs d=256 with m∈{9,10,10,11}. Slack stays ≤2^{-128} either way; review's specific m=20 is paper-grounded but not the implemented case.
3. FC1-016: Review's expanded paradigm map labels [7] as "LeanSig 2025/1332 HAPPIER LightSec'25" — these are reviewer-introduced taxonomic labels, not the actual title of [7], which is "Hash-Based Multi-Signatures for Post-Quantum Ethereum" (Drake-Khovratovich-Kudinov-Wagner, IACR CiC 2025).
4. FC1-012, FC1-013, FC1-020: Aardal et al. and Anada et al. citations are reviewer's own — not in paper. Should be confirmed externally before treating as authoritative.
