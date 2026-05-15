# Review of `subagent_3_paper_feedback.md` (field placement / comparison fairness lens)

Independent verification against `/workspace/repo/report.pdf`,
`/workspace/repo/code/parameter/chipmunk_original_security_summary.txt`,
`/workspace/tmp/notes-related-work.md`, and live WebSearch checks for
the claimed-missing 2024-2025 references.

---

## Per-claim verdicts

### Claim A — "[15] MOR '01 is never cited in body; bibliography filler"
**VERIFIED.** Grep over `pdftotext -layout report.pdf` shows `[15]`
appears exactly twice: once as the pair `[10, 15]` in the §1 opening
sentence ("...by producing a single, compact aggregated signature
[10, 15]"), and once as the bibliography entry. There is no other
body-text use. The audit's secondary point — that MOR is about
*accountable-subgroup* multisigs, a strict refinement, not a generic
origin of multi-signatures — is also correct (MOR title is literally
"Accountable-subgroup multisignatures").

### Claim B — "[2] Boneh-Kim is mis-placed as KOTS ancestor"
**VERIFIED.** §1.2 says verbatim: "Chipmunk's KOTS closely follows the
line of work of Lyubashevsky-Micciancio [13] **and Boneh-Kim [2]**, and
is conceptually very simple." The paper never tells the reader that
Boneh-Kim is non-synchronized or that its one-time variant has
log-size aggregate. Audit's framing — "buried as ancestor so the
reader will not learn this" — is fair. **However** the audit overreaches
slightly by saying Lemur "avoids ever explaining why Lemur shouldn't
be compared to it"; in fact §1 already establishes the synchronized
framing, which implicitly excludes Boneh-Kim's non-synchronized
variant. The mis-placement is real but less rhetorically loaded than
the audit suggests.

### Claim C — "Table 1 silently mixes Rice-coded Lemur with raw Chipmunk"
**PARTIALLY CORRECT.** Verified the Table 1 caption literally says:
"Chipmunk: theoretical, from their scripts. Lemur: measured in
implementation for N=2¹⁰; Rice-encoded on the corresponding cells of
Table 5 for N ∈ {2¹⁵, 2²⁰}." So the asymmetry **is** disclosed in the
caption, not silently. Audit's word "silently" is too strong; the
correct characterization is "disclosed in the caption only, where a
hurried reader can miss it." Audit later softens this to
"caption-as-only-disclosure is sketchy" — that softer version is fair.
The ~14% number is confirmed by the paper itself ("394.4 KB vs.
457 KB at N=2²⁰", §7.1).

### Claim D — "Stripping Rice gives 449 KB Lemur" (arithmetic check)
**WRONG on the number.** Audit says 449 KB; the paper text (§7.1)
explicitly gives the raw bound as **457 KB at N=2²⁰**. The audit's
calculation 394 × 1.14 ≈ 449 is close but the actual raw bound from
the artifact is 457 KB. The 1.85× ratio against Chipmunk's 831 KB is
still roughly right (457/831 ≈ 1.82×, audit said 1.85×). Net effect
on the argument: nil. Net effect on credibility: minor.

### Claim E — "Size ratios 1.18× / 1.47× / 2.1× at N=2¹⁰/2¹⁵/2²⁰"
**VERIFIED.** Table 1 cells:
- N=2¹⁰: 237 / 201 = 1.179 → 1.18× ✓
- N=2¹⁵: >418 / 284 = 1.472 → 1.47× ✓
- N=2²⁰: >831 / 394 = 2.109 → 2.1× ✓

Audit's observation that "up to 2×" is at the favorable corner is
correct and well-anchored.

### Claim F — "RSIS rop range is [22.8, 39.4] in `chipmunk_original_security_summary.txt`"
**VERIFIED.** File inspection confirms:
- Minimum across all 24 rows × 3 RSIS columns: 22.8 (RSIS3, at
  secpar=112, τ∈{21,23,24,26}, N=131072).
- Maximum: 39.4 (RSIS1/RSIS2 at secpar=112, τ=21, N=1024).
- The (secpar=112, τ=21, N=8192) min = 26.0 ✓
- The (secpar=128, τ=21, N=8192) min = 27.2 ✓

### Claim G — "Lemur's '~40 bit' headline is generous; worst cell is ~23 bits"
**VERIFIED in spirit, slightly mis-stated.** The worst RSIS3 value
across all Chipmunk cells in the file is 22.8 (not 23). The audit's
"23-40 bits" range is a reasonable rounding of the actual [22.8, 39.4]
RSIS range. Audit is correct that calling it "40-bit" is the median /
generous-rounding interpretation rather than the worst-case. **Caveat:**
the paper's own text (§1.1) is qualified — "approximately 40-bit
*instead of* the claimed 112-bit" — which can defensibly mean
"approximately 40 in the best cell, worse elsewhere". Whether this
qualifies as rhetorical choice or honest rounding depends on the
reader's charity. Audit's framing is defensible, not damning.

### Claim H — "Aardal et al. CRYPTO 2024 (Falcon+LaBRADOR) exists and is a missing paradigm"
**VERIFIED.** WebSearch confirms: Aardal, Aranha, Boudgoust, Kolby,
Takahashi, "Aggregating Falcon Signatures with LaBRADOR", eprint
2024/311, CRYPTO 2024 (Springer LNCS, August 2024). The paradigm
description (standard PQ signature + lattice proof system,
non-synchronized, public aggregation) is consistent with the audit's
characterization. **The audit's claim that this paradigm has no slot
in Lemur's §1 trichotomy is correct** — Lemur's §1 only names BLS,
hash+SNARK ([7]), and lattice-synchronized. **One nuance:** the audit
calls Boudgoust an author but doesn't list her by name; trivial.

### Claim I — "Anada-Fukumitsu-Hasegawa '25 tightly-secure lattice synchronized AS in standard model exists"
**PARTIALLY CORRECT — venue is wrong.** The paper exists: "Tightly
Secure Lattice-Based Synchronized Aggregate Signature in Standard
Model" by Anada, Fukumitsu, Hasegawa. **However:** WebSearch shows
it appeared in **ICISC 2024** (Springer LNCS vol. 15596), not ProvSec
2025 as the audit asserts (both in §6.3 and §7). This is a venue
error in the audit. The substantive claim — "first lattice-based
synchronized AS secure in the standard model, direct competitor to
Lemur, missing from Lemur's references" — is still valid; the venue
mis-attribution is a sloppy detail the audit should have checked.

### Claim J — "LeanSig (eprint 2025/1332) and HAPPIER (LightSec 2025) exist"
**VERIFIED.**
- LeanSig: eprint 2025/1332, "Technical Note: LeanSig for Post-Quantum
  Ethereum" by Justin Drake et al. Confirmed; explicit successor-line
  to Drake et al. [7].
- HAPPIER: Saygan, Gündoğan, Arslan, Gönen, "Hash-Based, Aggregatable,
  Practical Post-quantum Signatures Implemented Efficiently with
  Risc0", LightSec 2025, Istanbul, Sep 2025. Confirmed.

Both are real, both are post-Drake hash+SNARK / SNARK-friendly
successors. The audit's characterization is supported.

### Claim K — "the 'order of magnitude' KOTS shrink is actually 5×, the paper's wording is favorable"
**VERIFIED.** Table 1: Chipmunk 26 KB vs Lemur 5.6 KB at N=2¹⁰ →
4.64×. "Order of magnitude" overstates. At higher N it widens:
Chipmunk >110 KB vs Lemur 8.6 KB → ~12.8× at N=2²⁰. So the audit's
"4-13× depending on cell" is correct; the paper's "an order of
magnitude" is on the favorable end. Minor framing rather than a fault.

### Claim L — "Lemur not given Chipmunk an opportunity to choose new parameters"
**VERIFIED.** Table 5 caption (paper p. 12) states the Chipmunk
scripts "could not find feasible parameters" at the higher RHF.
Audit's "hostile reviewer" framing is fair: Lemur compares against
Chipmunk's published parameters re-fit, not against a Chipmunk-team
re-parameterization. This is the standard practice for security
recomputation papers but worth flagging.

---

## Validated findings worth preserving

1. **MOR [15] is bibliography filler** — accurate and verifiable;
   recommend the authors either use it (e.g., in §6 to clarify Lemur
   is *not* accountable) or remove it.
2. **Boneh-Kim [2] mis-placement in §1.2 as KOTS ancestor** — accurate;
   sentence-level fix would resolve.
3. **Rice/raw asymmetry in Table 1** — accurate; caption discloses
   but a top-line table footnote would improve clarity.
4. **"40-bit" is rounded toward the median, not the worst-case** —
   accurate; the actual worst RSIS3 bit-security cell is ~23 bits.
5. **Aardal et al. CRYPTO 2024 is a genuinely missing paradigm
   bucket** — verified; this is the strongest single omission the
   audit identifies.
6. **LeanSig + HAPPIER + ecosystem post-date and supersede [7]
   in deployment momentum** — real and citable.
7. **N=2²⁰ "2×" headline is the favorable corner** — accurate;
   ratios are 1.18× / 1.47× / 2.1× across N=2¹⁰/2¹⁵/2²⁰.
8. **Drake et al. [7] single-sentence dismissal** — accurate
   characterization of what §1 actually says (verified by grep).

---

## Errors to correct

1. **"449 KB" raw-Lemur calculation** (§2 of audit, para after
   "Stripping Lemur's Rice coding"): actual raw figure from paper
   §7.1 is **457 KB**, not 449. The audit reverse-engineered from
   1.14× factor instead of reading the explicit number.
2. **Anada-Fukumitsu-Hasegawa venue**: audit says ProvSec 2025;
   WebSearch confirms ICISC 2024 (Springer LNCS 15596). Audit should
   correct the venue or remove the citation if it can't pin down
   the venue.
3. **"silently mixes Rice-coded Lemur with raw Chipmunk"** —
   "silently" is wrong; the asymmetry is disclosed in the Table 1
   caption. "Discloses only in the caption" is the accurate phrasing.
4. **§6.5 "Identity-based interactive aggregate (ICISC '22)"** is
   asserted with no citation, no author, no eprint number. Either
   provide the actual reference or drop the bullet. As-is this is
   unverifiable and weakens the audit's overall credibility.
5. **The "5.6× smaller than Squirrel" claim that the audit
   attributes to Chipmunk's parameters** — not directly verified
   against Squirrel paper in this audit; treat as restated, not
   independently checked.

---

## Missed angles the audit should have caught

1. **The paper's own §1.1 admits "we are unable to provide a
   meaningful runtime comparison against Chipmunk"** (verified at
   `/tmp/report.txt:139`). This is a structural concession that
   undercuts the audit's "comparison story" framing somewhat —
   the paper itself flags that timing is not compared. The audit
   focuses on size comparison fairness but doesn't note that the
   timing comparison is **absent**, which is arguably a larger gap
   (a reviewer asking "how fast is verification at scale?" gets no
   Chipmunk number to compare against).
2. **No mention of EUF-RK vs EUF-CMA** for Anada et al. The audit
   says Anada is "standard model variant", but the audit doesn't
   check whether Anada targets EUF-RK (the standard for synchronized
   multi-sigs) or a weaker model. This affects whether it's actually
   "in the same column" as Lemur.
3. **The "RHF ≤ 1.0045" choice itself**. The paper sets this and
   notes Chipmunk scripts "could not find feasible parameters" at
   this RHF. The audit accepts the RHF threshold as given without
   asking whether it is itself a choice that disadvantages Chipmunk.
   This would be a "hostile reviewer" question worth flagging.
4. **Boudgoust-Takahashi '23 sequential half-aggregation** is
   mentioned in §6.4 of the audit but with no follow-through —
   no claim about why Lemur should compare to it. The bullet is
   dead weight; either expand or remove.
5. **The "Hint-MLWE for KOTS unforgeability" usage is a novel
   application of an assumption originally introduced for ZK
   proofs.** The audit does not assess whether Lemur's Dual
   Hint-MLWE assumption is well-motivated outside its use in
   Lemur. A field-placement audit should ask: is Dual Hint-MLWE a
   reasonable assumption to publish in isolation, or does it look
   ad-hoc? (Spot-checking §3 and Theorem 3.1: the reduction to
   standard MLWE looks rigorous; ad-hocness concern is mitigated.
   But the audit should have at least mentioned the question.)
6. **The paper claims aggregation in 12 minutes at N=2²⁰** (Table 2,
   verified at `/tmp/report.txt:96`). The audit's claim that
   "Drake et al. hash+SNARK takes seconds to minutes per aggregation"
   may not actually beat Lemur on the latency axis if Lemur also
   takes minutes at this N. The audit should have noted this
   self-undercut.

---

## Overall assessment

The audit's substantive findings (1-8 in "validated findings") are
defensible and well-anchored to source data. The factual errors
(449 vs 457 KB; ProvSec 2025 vs ICISC 2024; unverifiable ICISC '22
identity-based citation) are minor but show the audit was written
quickly and not fact-checked. The "missed angles" listed above are
real gaps a deeper audit would catch but do not invalidate the
existing findings.

**Net verdict:** keep the audit's findings; correct the three factual
errors; treat the recommendation section (§8 of the audit) as solid.
The audit is more credible than not; the rhetorical framing
("manufactures more daylight than the underlying facts support") is
defensible at the field-placement level but should be softened with
respect to the size-comparison level, where the numbers do support
Lemur's claim.
