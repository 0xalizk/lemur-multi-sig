# Audit of Lemur — Field Placement, Related Work, and Comparison Fairness

Auditor lens: prior-work survey, paradigm placement, comparison fairness.
Paper: `/workspace/repo/report.pdf` (24 pages, dated 2026, Anonymous Authors,
submitted to a CCS-style venue).

Bottom line up front: **the technical contribution is real, the paradigm
placement is mostly honest, but the comparison story leans heavily on three
rhetorical moves that deserve closer reading: (1) attacking Chipmunk's
parameters rather than its construction, (2) dismissing Drake et al. on a
single sentence, (3) silently ignoring two adjacent paradigms that have
emerged 2024-2026.** Each is defensible in isolation, but together they
manufacture more daylight between Lemur and its competitors than the
underlying facts support.

---

## 1. The 17 references — actual functional roles

The notes in `/workspace/tmp/notes-related-work.md` give a per-ref taxonomy.
Independently verifying against body-text appearances:

| # | Used as | Verdict on the citation |
| - | --- | --- |
| [1] APS '15 | Lattice estimator black-box | Honest. The whole §7.1 strategy depends on it being the right estimator. |
| [2] Boneh-Kim '20 | "KOTS… arising from [13] and [2]" §1.2 KOTS recap | **Partially misleading.** See §3 below. Boneh-Kim is treated as a fellow ancestor of KOTS but is actually a *competing paradigm* (non-synchronized lattice aggregate). The body never tells the reader Boneh-Kim is non-synchronized. |
| [3] BLS '01 | "BLS signature scheme [3]" §1 | Standard. Used correctly as the deployed-but-not-PQ baseline. |
| [4] BDS08 | Stateful Merkle traversal §7.2 | Standard textbook citation. |
| [5] Buser et al. '22 | "As shown in [5], a unique one-time signature can be extended to a many-time VRF" §1.1 | Honest future-work pointer; not load-bearing. |
| [6] G+G '23 | Lemma 2.5 (full-rank Gaussian sampler) | Foundational tool. |
| [7] Drake et al. '25 | "Existing proposals [7] therefore achieve aggregation only by invoking succinct arguments" §1 | **Single-sentence dismissal.** See §4. |
| [8] Chipmunk '23 | Throughout — definitions, theorems, proofs, parameter comparison | Direct predecessor; load-bearing for every comparison claim. |
| [9] Squirrel '22 | "[9] and its successor [8]" §1 | Honest framing. |
| [10] Itakura-Nakamura '83 | Historical opener §1 | Cite-and-move-on. |
| [11] Kim et al. '23 (Hint-MLWE) | Theorem 3.1, §3 setup | Theoretical lever; Lemur generalises to *Dual* Hint-MLWE. Honest. |
| [12] Lyubashevsky '12 | Lemma A.5 tail bound | Foundational. |
| [13] Lyubashevsky-Micciancio '08 | "KOTS… line of work of [13]" §1.2 | Conceptual root of KOTS. Honest. |
| [14] Lyub-Seiler '18 | Lemma A.2 invertibility | Foundational. |
| [15] Micali-Ohta-Reyzin '01 | **Never cited in body** (notes-related-work.md flags this) | Bibliography filler. The cite hangs off "[10, 15]" in the §1 opener but [15] is *not* about origin of multi-sigs — it's about *accountable* multisigs. Sloppy bib. |
| [16] Micciancio-Regev '04 | Lemmas 2.3, 2.4 smoothing parameter | Foundational. |
| [17] Peikert '10 | Lemma 2.1, 2.5 Gaussian sampler | Foundational. |

The triage is clean for everything except **[2] and [15]**:

- **[15] MOR '01** is cited in the §1 opening pair "[10, 15]" as if it were
  the origin of multi-signatures alongside Itakura-Nakamura. MOR is actually
  about accountable-subgroup multisigs, a strict refinement. The bibliography
  entry exists but the body never uses it for anything substantive. This is
  the kind of "we cited the textbook reference" sloppiness that a careful
  reviewer would flag.
- **[2] Boneh-Kim '20** is grouped with [13] as "the line of work" KOTS
  comes from. Functionally it is a *different paradigm*: aggregate
  signatures without a slot schedule, logarithmic-size aggregate, but the
  many-time version is interactive. By burying it as an ancestor, the
  paper avoids ever explaining why Lemur shouldn't be compared to it.

The 4-5 references that actually drive field placement: [3] BLS, [7] Drake
et al., [8] Chipmunk, [9] Squirrel, [11] Hint-MLWE. The skill's
4-bucket triage matches this conclusion exactly.

---

## 2. The "~2× smaller than Chipmunk at corrected security level" claim

This is the headline. Unpacking it:

### What the paper actually shows (Table 1, p. 2; Table 5, p. 12)

At λ=128, τ=20, N=2²⁰, with RHF ≤ 1.0045 throughout:
- Chipmunk: > 110 KB one-time sig, > 831 KB multi-sig (theoretical; the
  scripts could not find feasible parameters at this RHF).
- Lemur: 8.6 KB one-time sig, 394 KB multi-sig (Rice-encoded; **measured
  in implementation** for N=2¹⁰ and **predicted Rice-encoded** for
  larger N).

So the ~2× claim is **multi-sig 831 KB vs 394 KB** at N=2²⁰. That is
genuinely ~2.1×. At N=2¹⁵ it's 418 vs 284, more like 1.47×. At N=2¹⁰
it's 237 vs 201, more like 1.18×. The headline "up to 2×" is at the
largest realistic N, which is the favorable corner.

### Is the comparison apples-to-apples?

Walking the skill's red-flag checklist:

1. **Same recomputed security level?** This is where Lemur's whole pitch
   lives. The paper says Chipmunk's published parameters realize "only
   around 40-bit security (instead of the claimed 112-bit level)" (abstract;
   §1; §1.1). Inspecting `chipmunk_original_security_summary.txt`:
   - The "RSIS rop" columns are `log2(rop)` from the lattice estimator's
     `rough` mode (confirmed in `chipmunk_original.sage` line 38-39).
   - For Chipmunk's published (secpar=112, τ=21, N=8192) cell, the
     three RSIS columns are 33.3, 33.3, 26.0. The minimum, 26 bits, is
     SIS-3.
   - For (secpar=128, τ=21, N=8192) the minimum drops to 27.2.
   - At secpar=112, τ=21, N=131072 the minimum is 22.8.

   Lemur rounds the "minimum RSIS bit-security" across Chipmunk's
   published cells to ~40. That is generous — the *actual* worst cell is
   ~23. The "40-bit" headline appears chosen to be defensible while
   still dramatic. (Calling it 25-bit would be technically more accurate
   to the worst cell and rhetorically stronger; calling it 27-bit at the
   8192-signer setting would be the apples-to-apples number for the
   8192-comparison in Chipmunk's headline.) **The estimator output is
   convincing as evidence Chipmunk's parameters are far below 112 bits;
   the precise number "40" appears to come from an arithmetic-average or
   the best cell rather than the worst.**

   The skill's note that "Chipmunk's published 112-bit parameters
   actually realize ~40 bits" is now traceable to a specific data file
   but should be sharpened to "23-40 bits depending on which cell, with
   the largest-N cells worst."

2. **Rice-coded or raw?** Table 1 is explicit: Lemur is Rice-encoded
   "on the corresponding cells of Table 5 for N ∈ {2¹⁵, 2²⁰}", giving
   a ~14% reduction. Chipmunk's column is "theoretical, from their
   scripts" (Table 1 caption). The Chipmunk script also produces raw
   sizes — they don't apply Rice. **This is the comparison-fairness
   issue the skill flags.** Stripping Lemur's Rice coding (multiply
   by 1.14): Lemur multi-sig at N=2²⁰ becomes 449 KB (matches Table 5
   "Total" column for τ=20, N=2²⁰), vs Chipmunk's 831 KB. Still ~1.85×;
   the 2× number tightens slightly but doesn't collapse. **Not a fatal
   issue, but the caption-as-only-disclosure is sketchy. A careful
   reader has to compare Tables 1 and 5 to discover this.**

3. **Same signing model?** Both schemes are synchronized with BDS08
   stateful signing. Apples-to-apples here.

4. **Same N?** Compared at N ∈ {2¹⁰, 2¹⁵, 2²⁰}. Fair.

5. **Same security definition?** Both EUF-RK. Fair.

6. **Same lattice assumption family?** Chipmunk is Ring-SIS; Lemur is
   Module-SIS for HVC and MLWE/Dual-Hint-MLWE for KOTS. The skill
   correctly notes Ring-SIS gives *tighter* parameters than
   Module-SIS — so the Module-SIS shift is **not** part of why Lemur
   gets smaller. The KOTS shrink comes from the computational vs.
   statistical argument (Dual Hint-MLWE replacing Chipmunk's
   statistical KOTS unforgeability). The HVC Module-SIS shift is for
   *parameter flexibility*, not size; the paper says so in §1.1 ("more
   compact HVC… allows for more compact commitments and openings under
   aggregation").

### Verdict on "2× smaller"

The claim is **defensible**, with three caveats:

- The "2×" holds at N=2²⁰; at N=2¹⁰ it's 18%.
- Chipmunk's table is raw, Lemur's table is Rice-encoded.
- The comparison is against **Chipmunk's published parameters re-fit to
  RHF ≤ 1.0045**, not against parameters Chipmunk's authors would have
  picked if forced to that RHF. Chipmunk's authors are not given an
  opportunity to choose new parameters; the paper says the Chipmunk
  scripts "could not find feasible parameters" for some cells at
  RHF ≤ 1.0045 (Table 5 caption, p. 12).

The comparison is honest enough to pass review, but a hostile reviewer
would point out that **the 2× win is asymptotic-in-N**, the **Rice
asymmetry hides another ~14%**, and the **claim implicitly assumes
Chipmunk's authors had no recourse** when shown their RHF was too
loose.

---

## 3. Boneh-Kim [2] — the non-synchronized model confusion

The paper places [2] as a co-ancestor of KOTS in §1.2 ("the line of work
of Lyubashevsky-Micciancio [13] and Boneh-Kim [2]"). This is the same
phrasing the skill warns about in Step 6 ("not directly comparable to
synchronized multi-sigs — different deployment model. Cite for
completeness; do not compare numerically.").

What's actually in Boneh-Kim '20 (`/workspace/tmp/boneh-kim-2020-agg-ots.pdf`,
verified via Stanford crypto.stanford.edu/~skim13/agg_ots.pdf):

- Two schemes. The one-time aggregate signature has **logarithmic-size
  aggregate** in the number of signatures, public aggregation, no slot
  schedule.
- The many-time scheme requires interaction.

**The omission Lemur makes:** Boneh-Kim's one-time scheme has
log-size aggregate — that's *much better* than Lemur's 394 KB at N=2²⁰
on the size axis alone. The reason it isn't deployed is the
non-synchronized model and the lack of a sound way to scale up to
many time slots while staying compact. Lemur should have at least one
sentence acknowledging that Boneh-Kim achieves smaller aggregates
under a different model. Instead, [2] is folded into the KOTS
genealogy where the reader will not learn this. **This is a
field-placement weak point.**

The honest framing: "Lattice non-synchronized aggregate signatures
[2] achieve sub-linear aggregate size at the cost of an interactive
many-time variant or a one-time-only public-aggregation variant; we
target the synchronized model used by [8, 9] which trades aggregate
size for non-interactive many-time signing." That sentence would not
weaken the paper but does not appear.

---

## 4. Drake et al. [7] — the "heavy proof machinery" dismissal

§1 contains the only mention of [7]:

> Existing proposals [7] therefore achieve aggregation only by invoking
> succinct arguments, such as SNARK-like proof systems, to prove
> knowledge of many valid signatures. While conceptually appealing,
> this approach introduces heavy proof machinery and substantial
> computational overhead, which makes it difficult to deploy in
> latency-sensitive consensus settings.

Three problems:

1. **"[7]" is a single citation but the hash+SNARK paradigm is much
   bigger.** Drake et al. '25 (eprint 2025/055, CiC '25) has a
   follow-up "LeanSig" (eprint 2025/1332) and a successor ecosystem
   ("leanXMSS", "leanMultisig", "leanSpec"; pq.ethereum.org). As of
   May 2026 this paradigm has ~10 client teams building on it for
   Ethereum's actual PQ migration with 2029 target. Lemur cites
   only [7] and characterises the entire paradigm by it.

2. **"Heavy proof machinery and substantial computational overhead"
   is hand-wavy.** A serious related-work treatment would cite
   benchmarked sign/verify numbers. The hash+SNARK paradigm
   (Drake et al., HAPPIER '26 with Risc0, leanSig benchmarks)
   reports "hundreds of XMSS signatures per second on M4 Max" for
   aggregation, and signature sizes of 3000 bytes per validator
   pre-aggregation. Lemur's individual signing time of 4 ms is
   competitive but the *aggregate* signature size at N=2²⁰ is
   394 KB — vs Drake et al.'s aggregate proof which is bounded
   by a SNARK proof (typically tens of KB).

3. **The latency claim deserves a number.** "Difficult to deploy
   in latency-sensitive consensus" is exactly the kind of claim
   that should cite a benchmarked verification or aggregation time.
   None given.

**Verdict on the Drake dismissal:** uncharitable and shallow. A more
honest version would say: "Hash+SNARK schemes [7] achieve smaller
aggregate sizes (≤ tens of KB) at the cost of seconds-to-minutes
aggregation latency; we target a different point on the curve where
aggregation is sub-second but the aggregate is hundreds of KB."

This is also where the **field-placement honesty** breaks down. The
paper's §1 framing reads as if lattice-synchronized is the only
serious PQ multi-sig contender. After 2024-2026 ecosystem changes,
this is no longer true — hash+SNARK has overtaken it in real
deployment intent for Ethereum.

---

## 5. Squirrel [9] and Chipmunk [8] — novelty attribution

The paper is admirably explicit about reusing Chipmunk's framework.
§1.1: "Lemur builds upon the non-interactive framework in Chipmunk
[8] but fundamentally redesigns the foundations of its underlying
building blocks". §1.2 says Lemur follows the blueprint of
Chipmunk and highlights only "where Lemur departs". The HVC
construction in Figure 4 explicitly cites [8] for component
definitions; Defs B.2-B.5 cite Chipmunk; Lemma B.2 quotes Theorem
25 from [8]; the aggregation layer is inherited unchanged.

This is correct attribution. The two layers Lemur claims novelty on:

- **KOTS:** new Dual Hint-MLWE assumption (§3) + Theorem 3.1
  reduction from MLWE. The novel proof step is Lemma 3.5
  (sampleability over non-full-rank lattices). This is genuine new
  cryptographic content.
- **HVC:** Ring-SIS → Module-SIS lift, commitment domain
  Rₑq^{ρ×ν} (matrices) instead of Rₑq (vectors). Less dramatic but
  load-bearing for parameter flexibility.

The skill's "4 blocks, Lemur touches KOTS and HVC" framing matches
the paper exactly. The "KOTS gets ~10× smaller from the computational
vs. statistical argument" claim is consistent with §1.1's "an order
of magnitude improvement over Chipmunk's KOTS" — also Table 1, where
the One-Time Sig column shows 5.6 KB vs 26 KB at N=2¹⁰, ~5×. The
"order of magnitude" wording is on the favorable side of 5×.

**Verdict:** novelty attribution is honest. The only nit is that the
"order of magnitude" framing is generous; the actual KOTS shrink
ratio is 4-13× depending on cell.

---

## 6. Papers Lemur should have cited but didn't

These are real omissions, not just nice-to-haves:

### 6.1. Falcon + LaBRADOR aggregation (Aardal et al., CRYPTO 2024, eprint 2024/311)

This is a **new paradigm** that the paper's framing misses
entirely: aggregate Falcon (NIST PQ standard) signatures using
LaBRADOR (a lattice-based proof system). This sits between the
hash+SNARK and lattice-synchronized buckets — it's lattice-based
but uses a proof system for aggregation, like the hash+SNARK
paradigm. It is **non-synchronized**, public-aggregation,
post-quantum, and uses standardized signatures as the inner
primitive.

Lemur's §1 trichotomy (BLS / hash+SNARK / lattice-synchronized)
does not have a bucket for this. The paradigm map in the skill
also misses it. **This is a real gap.** A reviewer who knows the
field will notice.

### 6.2. Hash+SNARK successor papers

The "[7]" citation covers Drake et al.'s original paper only.
Successor work (eprint 2025/1332 "LeanSig"; HAPPIER at
LightSec '25 with 2-3 MB sigs and 2^16 signers on a laptop;
leanMultisig/leanXMSS in the Ethereum PQ ecosystem) is
ignored. By May 2026 the hash+SNARK paradigm has produced
≥4 papers and a deployment roadmap.

### 6.3. Anada-Fukumitsu-Hasegawa '25 (ProvSec 2025)

"Tightly Secure Lattice-Based Synchronized Aggregate Signature
in Standard Model" — first lattice-based synchronized AS
secure in the **standard model** (no ROM), based on the FHSZ
multi-sig construction. **Direct competitor to Lemur in the
synchronized lattice column**, with different security
assumptions (standard model vs. ROM). Lemur does not cite this
even though it would have been online before Lemur's
submission. **This is the most damaging omission for field
placement.**

### 6.4. Sequential half-aggregation (Boudgoust-Takahashi '23, eprint 2023/159)

Lattice signatures, sequential aggregation. Different model
(sequential vs. synchronized non-interactive), but a fair
related-work section would mention it as a third lattice
aggregation paradigm beyond synchronized and Boneh-Kim style.

### 6.5. Identity-based interactive aggregate (ICISC '22)

Adjacent. Could be cited; not load-bearing.

**Summary:** Lemur cites 17 papers but misses **at least 3
field-relevant 2024-2026 works** (Falcon+LaBRADOR, Anada et al.,
hash+SNARK successors). Given that Lemur is dated 2026, with the
abstract specifically claiming "post-quantum non-interactive
aggregation is no longer out of reach" (§1, end), citing only
one PQ competitor paper ([7], Drake et al. '25) is thin. A
reviewer who cares about field placement will flag this.

---

## 7. Is Lemur superseded by a post-submission paper?

As of May 2026:

- **leanSig/leanXMSS** is in development with Ethereum target 2029;
  not a single peer-reviewed paper but an ecosystem. If Ethereum
  proceeds with leanSig, Lemur loses its "for Ethereum" framing
  even before publication.
- **HAPPIER** (LightSec '25, Saygan et al.) reports 2-3 MB
  aggregate sigs and 2^16 signers on a laptop. Lemur's 394 KB
  at 2^20 is still ~5-8× smaller. So HAPPIER doesn't supersede
  Lemur on size, but it is **closer than Lemur's §1 dismissal of
  the entire hash+SNARK paradigm implies**.
- **Anada-Fukumitsu-Hasegawa '25** is in the same lattice-sync
  column. Lemur should compare to it. It's the standard-model
  variant, so the security model is genuinely stronger; the
  size/time numbers presumably weaker, but Lemur not citing it
  means readers can't tell.

**Verdict on supersession:** not superseded on numbers. Superseded
on **paradigm framing** by the leanSig ecosystem, which targets
the same application (Ethereum PQ validator aggregation) with a
different paradigm and considerably more momentum.

---

## 8. Concrete issues for the authors

If I were a reviewer:

1. **The "40-bit Chipmunk" headline is rhetorically chosen.** The
   estimator output shows 23-40 bits across cells, worst at
   high N. Either give the worst-case number (23) or give a
   range; "40" feels like the median, picked to be defensible
   yet dramatic.
2. **The Rice-vs-raw asymmetry in Table 1 should be in the
   caption, not buried in a footnote.** A reviewer not comparing
   Tables 1 and 5 line-by-line will miss it.
3. **The Drake et al. dismissal is uncharitable.** A two-paragraph
   related-work treatment with citation of the hash+SNARK
   ecosystem (leanSig, HAPPIER, etc.) and concrete latency
   numbers would address this.
4. **Boneh-Kim '20 is mis-placed.** It is a competing paradigm,
   not a KOTS ancestor.
5. **Falcon+LaBRADOR is missing entirely.** This is the largest
   gap.
6. **Anada et al. ProvSec '25 is missing.** First lattice-sync
   AS in the standard model; same column as Lemur.
7. **[15] MOR '01 is dead weight in the bibliography.** Either
   use it (e.g., to clarify that Lemur is *not* accountable) or
   remove it.

None of these would prevent acceptance at a top venue, but
collectively they make the related-work and field-placement
sections weaker than the technical content deserves.

---

## 9. Where the paper is strong

- **Technical novelty (Dual Hint-MLWE, Theorem 3.1 reduction,
  Lemma 3.5 non-full-rank sampleability) is genuine.**
- **The estimator-output critique of Chipmunk is well-grounded
  and reproducible from the committed artifact.**
- **The novelty attribution to Chipmunk and Squirrel is
  scrupulous: every inherited definition is cited, every
  modification is flagged.**
- **The Rust + Python dual implementation cross-validated
  byte-for-byte is unusual and credible.**

The paper is technically strong; the field-placement is just
slightly more self-flattering than it needs to be.

---

## Files referenced for this audit

- `/workspace/repo/report.pdf` — Lemur paper (24 pages).
- `/workspace/repo/code/parameter/chipmunk_original_security_summary.txt`
  — Chipmunk-under-modern-estimator data (the 27.2-bit number is
  there, not "40-bit").
- `/workspace/repo/code/parameter/summary.txt` — Lemur's own
  parameter cells; confirms Table 5 numbers.
- `/workspace/repo/code/parameter/chipmunk_original.sage` lines
  28-42 — confirms RSIS columns are `log2(rop)` in `rough` mode.
- `/workspace/tmp/notes-related-work.md` — prior triage notes;
  independently verified against body-text uses.
- `/workspace/tmp/boneh-kim-2020-agg-ots.pdf` — Boneh-Kim '20
  fetched for the non-synchronized check.
