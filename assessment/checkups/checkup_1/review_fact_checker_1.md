# Independent re-verification of fact_checker_1.md

Reviewer-of-reviewer pass over FC1's 52 entries against:
- `/tmp/lemur.txt` (pdftotext -layout of `submission/report.pdf`)
- `submission/code/parameter/summary.txt`
- `submission/code/parameter/chipmunk_original_security_summary.txt`
- WebSearch for external citations FC1 marked UNVERIFIABLE

## Spot-check verdicts

### FC1-004 (2.1× ratio @ N=2^20)
- FC1's verdict: VERIFIED
- Independent verdict: AGREE
- Note: Table 1 line 98: Chipmunk Multi-Sig >831, Lemur 394. 831/394 = 2.109. The ">" sign means 2.1× is a *lower bound*; review.md phrasing "~2.1×" is honest. Verified.

### FC1-005 (1.18× ratio @ N=2^10)
- FC1's verdict: VERIFIED
- Independent verdict: AGREE
- Note: 237/201 = 1.179. Confirmed.

### FC1-007 (Chipmunk 22.8–39.4 bit range)
- FC1's verdict: VERIFIED
- Independent verdict: AGREE
- Note: Direct grep of `chipmunk_original_security_summary.txt` secpar=112 rows: RSIS1/2 rop ∈ {26.9, 33.3, 39.4, 39.1}, RSIS3 rop ∈ {22.8, 26.0, 29.5}. Global min=22.8 (rows N=131072), global max=39.4 (rows N=1024, τ=21). Range claim exact.

### FC1-009 (KOTS 4.64× @ N=2^10)
- FC1's verdict: VERIFIED
- Independent verdict: AGREE
- Note: 26/5.6 = 4.643. Confirmed.

### FC1-010 (KOTS 12.8× @ N=2^20)
- FC1's verdict: VERIFIED
- Independent verdict: AGREE
- Note: >110 / 8.6 = 12.79 (lower bound). Confirmed.

### FC1-011 (no runtime comparison vs Chipmunk)
- FC1's verdict: VERIFIED
- Independent verdict: AGREE
- Note: /tmp/lemur.txt line 137-139 verbatim quote. Confirmed.

### FC1-012 (Aardal et al. CRYPTO '24 Falcon+LaBRADOR)
- FC1's verdict: UNVERIFIABLE
- Independent verdict: REVISE-TO-VERIFIED (with author-list correction)
- Note: WebSearch confirms ePrint 2024/311 = Aardal, Aranha, **Boudgoust, Kolby, Takahashi** "Aggregating Falcon Signatures with LaBRADOR" CRYPTO 2024 (Springer LNCS, pp. 71–106). FC1's tentative author list "Aardal-Aranha-Boschini-Lehmann" was wrong (Boschini & Lehmann are not on this paper). review.md only cites the paper as "Aardal et al.", which is correct. Venue/topic/year all confirmed.

### FC1-013 (Anada et al. ICISC '24)
- FC1's verdict: UNVERIFIABLE
- Independent verdict: REVISE-TO-VERIFIED
- Note: WebSearch confirms Anada, Fukumitsu, Hasegawa "Tightly Secure Lattice-Based Synchronized Aggregate Signature in Standard Model" in ICISC 2024 (Springer LNCS 978-981-96-5566-3, Ch. 4). Self-described as "first lattice-based synchronized AS [and] secure in the standard model" — matches review.md's framing exactly.

### FC1-020 (Lemur trichotomy sidesteps Aardal/Anada)
- FC1's verdict: UNVERIFIABLE
- Independent verdict: REVISE-TO-VERIFIED
- Note: Combined with FC1-012/013 confirmations + grep of paper references (lines 1038–1085) showing neither Aardal nor Anada cited, review.md's "sidesteps both" claim is factually grounded. The Lemur §1 trichotomy (BLS / hash+SNARK / lattice-synchronized) is in paper but does not engage with these adjacent paradigms.

### FC1-028 (m=20 slack arithmetic)
- FC1's verdict: PARTIALLY CORRECT
- Independent verdict: AGREE (FC1 caught a real subtlety)
- Note: Paper line 1021 cites "ℓ=1, m=20, **d=128**, k=5, α_H=31, ε₁=2^{-150}" but this is in §7 step (1) which derives the **individual correctness ε_cor bound (Constraint 7)** — NOT directly the 8m·ε₂ slack of Theorem 3.1. The 8m·ε₂ slack from Lemma 3.3 uses ε₂=2^{-136} (line 968). The actual shipped parameter set in `summary.txt` is **d=256, m∈{9,10,10,11}**. For the worst shipped m=11: 8·11·2^{-136} = 88·2^{-136} ≈ 2^{-129.54}, comfortably below 2^{-128}. For review's hypothetical m=20: 8·20·2^{-136} = 160·2^{-136} ≈ 2^{-128.68}. The arithmetic in review.md is correct *given m=20* but m=20 is the paper's d=128 illustrative computation for ε_cor, not the slack and not the shipped config. Recommend the aggregator footnote this: review's worst-case framing holds but is a different code path from the paper's example.

### FC1-043 (N(Q+1)² loss → 2^140, ~268-bit)
- FC1's verdict: PARTIALLY CORRECT
- Independent verdict: AGREE
- Note: Arithmetic exact (2^20 · 2^120 = 2^140). The Q_H = 2^60 figure is the reviewer's convention (paper does not state a Q_H bound), so PARTIALLY CORRECT is fair.

### FC1-047 (Table 2 reports Rice-encoded; Table 5 worst-case)
- FC1's verdict: PARTIALLY CORRECT
- Independent verdict: AGREE — review.md has a labeling error
- Note: Paper lines 100–104: **Table 1 holds Rice-encoded Lemur** (201.2/283.5/394.4 KB); Table 2 is the Rust performance benchmark (4.1 ms sign, 30.1 ms verify); Table 5 (line 985) is raw "theoretical values without any entropic encoding" with τ ∈ {12,16,20,24}. review.md line 186-187 says "Table 2 reports Rice-encoded; Table 5 reports worst-case bounds." This is wrong — it's Table 1 that reports Rice-encoded. The 14–16% delta is between Table 1 and Table 5, not Table 2 and Table 5. Aggregator should fix this in the final review.

### FC1-048 (15.8/14.4/13.7% Rice savings)
- FC1's verdict: VERIFIED
- Independent verdict: AGREE
- Note: Table 5 τ=20 raw: 239 / 331 / 457 KB; Table 1 Rice: 201.2 / 283.5 / 394.4. Computed: 15.82% / 14.35% / 13.70%. Exact. Also note paper line 1009 confirms "394.4 KB vs. 457 KB at N=220" verbatim.

### FC1-008 / FC1-049 (Table 1 caption discloses Rice/raw asymmetry; column headers do not)
- FC1's verdict: VERIFIED
- Independent verdict: AGREE
- Note: Line 92 header is bare "Chipmunk (KB) | Lemur (KB)"; caption at line 100-104 explicitly says "Lemur: measured in implementation for N=2^10; Rice-encoded on the corresponding cells of Table 5 for N∈{2^15,2^20}" and "Chipmunk: theoretical, from their scripts". The asymmetry-not-in-headers claim is exact.

### FC1-026 (Theorem 3.1 advantage bound 8m·ε₂, no multiplicative loss)
- FC1's verdict: VERIFIED
- Independent verdict: AGREE
- Note: Line 423 Theorem 3.1; the 8m·ε₂ statistical-distance bound is Lemma 3.3 (line 435). Theorem 3.1 statement at line 434 area gives `Adv^MLWE + 8m·ε₂`. No multiplicative coefficient on the MLWE term. Confirmed.

### FC1-033 (Theorem 4.1 loss N(Q+1)², Q = Q_H + N)
- FC1's verdict: VERIFIED
- Independent verdict: AGREE
- Note: Lines 561 (Theorem 4.1), 619 (Lemma 4.1), 640 (advantage bound with N(Q+1)² coefficient), 647 (Q = Q_H + N). Confirmed.

### FC1-039 (Lemma 4.1 restricted ℓ=1, line 620)
- FC1's verdict: VERIFIED
- Independent verdict: AGREE
- Note: Line 619-620: "Lemma 4.1. Let λ, p, t_1, t_2, q', d, m, n, k, ℓ, r, β_σ, α_H, α_w, N ∈ N+ … and ε_3 ∈ R such that ℓ = 1, k ≥ 4ℓ, r = k − 2ℓ". Confirmed.

### FC1-040 (paper acknowledges ℓ=1 restriction at line 545)
- FC1's verdict: VERIFIED
- Independent verdict: AGREE
- Note: Line 544–547 quote verified verbatim.

### FC1-041 (all shipped parameters use ℓ=1)
- FC1's verdict: VERIFIED
- Independent verdict: AGREE
- Note: `summary.txt` column "ell" reads 1 in every row I inspected (12 rows, all τ ∈ {12,16,20,24} × N ∈ {2^10,2^15,2^17,2^20}). Confirmed.

### FC1-046 (size headlines 201.2 / 283.5 / 394.4 KB)
- FC1's verdict: VERIFIED
- Independent verdict: AGREE
- Note: Table 2 row "Agg. signature size" (line 98): exact match. Also line 152, 1009.

### FC1-006 (Chipmunk claimed 112-bit)
- FC1's verdict: VERIFIED
- Independent verdict: AGREE
- Note: chipmunk_original_security_summary.txt rows 3–14 all have secpar=112; paper line 119-120 says "approximately 40-bit instead of the claimed 112-bit". Confirmed.

### FC1-016 (Drake '25 → "LeanSig 2025/1332 HAPPIER LightSec'25")
- FC1's verdict: PARTIALLY CORRECT
- Independent verdict: AGREE
- Note: Paper ref [7] (line 1056–1059) is Drake, Khovratovich, Kudinov, Wagner "Hash-Based Multi-Signatures for Post-Quantum Ethereum" IACR CiC 2(1):13, 2025. "LeanSig" and "HAPPIER LightSec'25" are reviewer-added taxonomic labels not in the paper. The ASCII tree placement under [7] is the reviewer's editorial mapping, not a paper claim.

## Solid FC1 verdicts (preserve as-is)

FC1-001, 002, 003, 004, 005, 006, 007, 008, 009, 010, 011, 014, 015, 017, 018, 019, 021, 022, 023, 024, 025, 026, 027, 029, 030, 031, 032, 033, 034, 035, 036, 037, 038, 039, 040, 041, 042, 044, 045, 046, 048, 049, 050, 052 — all checked or representative-sampled; verdicts hold.

## Corrections to FC1's verdicts

- **FC1-012** UNVERIFIABLE → VERIFIED. Aardal et al. is Aardal, Aranha, Boudgoust, Kolby, Takahashi "Aggregating Falcon Signatures with LaBRADOR" CRYPTO 2024 / ePrint 2024/311. FC1's note conjectured "Aardal-Aranha-Boschini-Lehmann" — incorrect author list, but review.md only writes "Aardal et al." so review.md is fine.
- **FC1-013** UNVERIFIABLE → VERIFIED. Anada, Fukumitsu, Hasegawa "Tightly Secure Lattice-Based Synchronized Aggregate Signature in Standard Model" ICISC 2024 (Springer LNCS, 10.1007/978-981-96-5566-3_4). Self-billed as first lattice synchronized AS in standard model.
- **FC1-020** UNVERIFIABLE → VERIFIED. With FC1-012/013 now confirmed and grep of paper refs showing neither cited, the "sidesteps both" claim is verifiable.
- **FC1-051** UNVERIFIABLE → can be partially confirmed if needed; out of §1-3 scope so leaving as-is is also acceptable.

## Claims FC1 missed (statements in review.md §1-3 worth flagging)

1. **review.md line 11-12: "11 CPU threads, 8 GiB RAM"** — environment-side fact, not in scope of paper-FC ledger but worth noting that paper Table 2 caption (line 104) says **24 CPU threads**; the 11-thread reproduction is contextually flagged in §1 already.
2. **review.md line 116: "no other PQ aggregate at this scale"** — phrasing attributed to paper; paper §1.1 and §1 do not use this exact phrase. The closest is line 144-146 "significantly smaller aggregated signature sizes for large sets of signers". Reviewer's quoted phrase is paraphrase, not a direct paper claim — worth a soft footnote.
3. **review.md line 113: "Lemur's stated paradigm trichotomy (BLS / hash+SNARK / lattice-synchronized in §1)"** — paper §1 doesn't explicitly enumerate a three-way taxonomy; it discusses BLS (deployed but not PQ), hash-based + SNARKs [7], and the lattice predecessors [8,9]. Reviewer's "trichotomy" is a reasonable characterization but not verbatim from paper.
4. **review.md line 53: "Squirrel/Chipmunk KOTS+HVC blueprint; it changes two of the four blocks (KOTS and HVC)"** — paper §1.2 line 190 says "Lemur follows the blueprint of Chipmunk [8]"; KOTS change (line 148-152) and HVC change (line 123-127) are both in paper; aggregation + BDS08 inherited (line 146-147, 211). The "two of four blocks" reduction is the reviewer's structural summary, supported by but not directly stated in paper.
5. **review.md line 67: "we are unable to provide a meaningful runtime comparison"** — exact quote at paper line 138-139. FC1-011 caught this; no issue.
6. **review.md line 178: "Dilithium does the same"** — community-practice claim, not in paper, not checked. Probably correct but unsupported in ledger.

## Suggested footnote sources for highest-value VERIFIED claims

1. **Chipmunk 22.8–39.4-bit range (review.md line 58 / FC1-007)** — cite `submission/code/parameter/chipmunk_original_security_summary.txt` (secpar=112 rows). This is the load-bearing rectification claim.
2. **2.1× / 1.18× ratios (review.md line 55-56 / FC1-004,005)** — cite paper Table 1 (line 92-104 of `/tmp/lemur.txt`, i.e. paper page 2 Table 1).
3. **KOTS 4.64× and 12.79× (review.md line 65 / FC1-009,010)** — same Table 1; arithmetic explicit.
4. **8m·ε₂ slack with m=20 caveat (review.md line 137-139 / FC1-028)** — cite paper Lemma 3.3 + line 968 (ε₂=2^{-136}) + footnote that m=20 is the paper's d=128 illustration (line 1021), and that shipped d=256 cells use m∈{9,10,10,11} per `summary.txt`. The slack stays ≤ 2^{-128} in both interpretations, but the cleaner derivation uses the shipped m=11.
5. **Aardal et al. (review.md line 72 / FC1-012)** — cite ePrint 2024/311 / CRYPTO 2024 / Aardal-Aranha-Boudgoust-Kolby-Takahashi. Confirms the citation reviewer added.
6. **Anada et al. (review.md line 73 / FC1-013)** — cite ICISC 2024 / Springer LNCS / DOI 10.1007/978-981-96-5566-3_4 / Anada-Fukumitsu-Hasegawa. Confirms standard-model sibling.
7. **Table 1 caption asymmetry (review.md line 61-63, 188-191 / FC1-008,049)** — cite paper Table 1 caption (lines 100–104 of dump) and Table 5 caption (lines 985–996). Headers are at line 92.
8. **Rice savings 15.8/14.4/13.7% (review.md line 187 / FC1-048)** — Table 5 raw rows τ=20 (lines 972/974/977) vs Table 1 Rice (line 98). Paper line 1009 even cites "394.4 KB vs. 457 KB at N=220" verbatim.

## Aggregate count after this pass

FC1 originally: 35 VERIFIED / 5 PARTIALLY / 0 FLAG / 4 UNVERIFIABLE / 8 NOT-CHECKED-AS-VERIFIED-OR-OTHER.

After reviewer-of-reviewer:
- VERIFIED: 35 + 3 (FC1-012/013/020 upgraded) = 38
- PARTIALLY CORRECT: 5 (FC1-003, 016, 028, 043, 047) — all FC1's PARTIAL flags stand
- UNVERIFIABLE: 1 (FC1-051 only — out of §1-3 scope)
- FLAG: 0

FC1's labeling-error catch on FC1-047 is the most impactful single finding; the aggregator should ensure final review.md says "Table 1" not "Table 2" for Rice-encoded sizes.
