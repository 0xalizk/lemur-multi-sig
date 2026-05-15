# Critique of the Two Lemur Skills

Skills audited:
- `/workspace/.claude/skills/lemur-paper-analysis/SKILL.md`
- `/workspace/.claude/skills/lemur-prior-work-survey/SKILL.md`

Lens: I just used both skills to audit Lemur's field-placement and
comparison-fairness claims. This is feedback from that specific use.

---

## 1. Did `lemur-prior-work-survey` help triage references efficiently?

**Mostly yes.** The Step 1 four-bucket triage cut the 17-reference list
to the 4-5 papers I actually needed to read (the same 4-5 the skill
predicts: [3] BLS, [7] Drake, [8] Chipmunk, [9] Squirrel, [11]
Hint-MLWE). The Step 2 "find the body-text use of each citation"
discipline matters — without it I would have wasted time on [12]
Lyubashevsky '12, which the skill correctly relegates to "foundational
tool" but the bibliography entry makes look central.

**Where it slowed me down:**

The bucket boundaries are not as clean as the skill suggests. Three
specific frictions:

1. **Boneh-Kim '20 [2] is placed in "competing paradigms" by the
   skill, but the Lemur paper itself places it as a KOTS ancestor in
   §1.2.** This is a *bucket disagreement* between the skill and the
   paper. The skill is right that it's a competing paradigm; the
   paper is wrong (or deliberately framing it as an ancestor to
   avoid the comparison). The skill should explicitly call out that
   the paper *mis-places* [2] — that's actionable feedback for the
   reader doing the audit. As written, the skill says "Lemur cites
   [2] only as conceptual context for the KOTS, not as a baseline"
   in the notes — but a survey-skill reader wants to know
   "**Lemur's framing of [2] is misleading; treat as competing
   paradigm regardless.**"

2. **[15] MOR '01 is in the bibliography but never used in the body.**
   The skill assigns it to "Adjacent / historical" and the notes
   say "Not cited in body that I could find" — but this isn't an
   "adjacent" reference; it's **bibliography filler**. A useful
   addition to the skill: a fifth bucket or sub-label for "in bib,
   not in body — likely included for completeness or by mistake."
   When auditing, finding such references is a positive signal
   ("the authors' related-work breadth is performative").

3. **The bucket of "Theoretical lever" is conflated with "the
   assumption introducing it"**. Hint-MLWE [11] is the assumption;
   Lyubashevsky-Micciancio [13] is the KOTS root; both are described
   as theoretical levers in different places. They're different:
   [11] gives the proof technique, [13] gives the construction
   template. The skill should split "assumption lever" from
   "construction template" — both are 1-2 papers each, both deserve
   careful reading, but for different reasons.

---

## 2. Is the paradigm map in Step 3 correct?

**Mostly. It misses one bucket that has emerged since the skill was
written.**

The current map:
```
Multi-signatures
├── Pairing-based
│   └── BLS (deployed, not PQ)
├── Hash-based + SNARKs
│   └── Drake et al. (PQ Ethereum, Winternitz + Merkle + pqSNARK)
└── Lattice-native
    ├── Synchronized (Squirrel → Chipmunk → Lemur)
    └── Non-synchronized (Boneh-Kim '20)
```

**Missing bucket: Lattice signature + lattice proof system.** The
Aardal-Aranha-Bhattacherjee-Khairallah-Tiwari "Aggregating Falcon
Signatures with LaBRADOR" (CRYPTO 2024, eprint 2024/311) sits
between "Hash-based + SNARK" and "Lattice-native" — it uses a
standardised lattice signature (Falcon) and aggregates with a
lattice proof system (LaBRADOR). Same paradigm-pattern as
hash+SNARK but every component is lattice-based.

**Concrete edit to `lemur-prior-work-survey/SKILL.md`, Step 3:**

Replace the ASCII tree with:

```
                       Multi-signatures
                              │
        ┌──────────────┬──────┴───────┬──────────────────────┐
        │              │              │                      │
  Pairing-based   PQ-sig + SNARK   PQ-sig + lattice-PoK   Lattice-native
        │              │              │                      │
       BLS         Drake et al.   Aardal et al.              │
   (deployed,    (Winternitz+SNARK) (Falcon+LaBRADOR,        │
    not PQ)      (PQ Ethereum)      CRYPTO '24)              │
                                                            │
                              ┌─────────────────────────────┤
                              │                             │
                   Synchronized (slot schedule)    Non-synchronized
                              │                             │
                              ▼                             ▼
                       Squirrel '22                  Boneh-Kim '20
                              │                       (OTS + interactive)
                              ▼
                       Chipmunk '23
                              │
                              ▼
                       Lemur '26
                              │
                              ▼
              Anada-Fukumitsu-Hasegawa '25
              (standard model variant)
```

And update Step 6 ("competing paradigm checks") to add a fourth
subsection:

> ### Lattice-sig + lattice-PoK (Aardal et al. '24)
> - Construction: standardised PQ signature (e.g., Falcon)
>   aggregated using a lattice-based non-interactive argument
>   (e.g., LaBRADOR).
> - Why it competes: every component is post-quantum and
>   standardised; the proof system is the only "new" cryptography.
> - Why it's hard: LaBRADOR is **interactive** at heart; the
>   non-interactive variant requires Fiat-Shamir over a complex
>   protocol, and the security analysis is recent (knowledge
>   soundness in ROM established only in the 2024 paper).
> - Compares against Lemur on: smaller aggregate (proof-system-
>   bounded, tens of KB) at the cost of standardised-signature
>   sign cost and proof-generation latency.

Step 4 (the four building blocks: KOTS, HVC, Aggregation, Stateful
signing) is fine as-is — those four blocks are specific to the
*synchronized lattice* column. Outside that column they don't apply.
The skill correctly flags this in its closing paragraph but the
disclaimer could be hoisted higher.

---

## 3. The 4-bucket triage — clean or did references fall between?

Most fell cleanly. The ones that didn't:

- **[13] Lyubashevsky-Micciancio '08:** Notes call it "Direct
  lineage — KOTS conceptual root." Skill's Step 1 puts "Direct
  lineage" as 2-4 papers including [13]. But [13] is *not* a
  predecessor in the same way [9] Squirrel is — Lemur doesn't
  inherit a construction from [13]; it inherits a *style*. Cleaner
  split: "Direct lineage (construction inherited)" vs "Direct
  lineage (style inherited)". The former is [8] and [9]; the
  latter is [13]. Reading effort differs: for [8] and [9] you
  need their construction figures; for [13] the abstract suffices.

- **[5] Buser et al. '22 (PQ-VRF):** Skill puts in "Adjacent /
  historical". I'd put in "Future-work hook" — a separate
  micro-bucket. The body cite is "As shown in [5], a unique
  one-time signature can be extended to a many-time VRF" (§1.1),
  which is forward-looking, not historical.

- **[2] Boneh-Kim '20:** Already discussed — bucket
  disagreement between skill (competing paradigm) and paper (KOTS
  ancestor). The skill is right; this should be flagged.

**Concrete edit to Step 1:** add a "Bucket disputes" rubric:

> If the paper's framing of a reference disagrees with this
> taxonomy, *trust the taxonomy*. A common move is to bury a
> competing paradigm as an "ancestor" to avoid the comparison;
> for Lemur, this happens with Boneh-Kim '20 [2]. The skill
> taxonomy reflects the functional role, not the paper's
> narrative role.

---

## 4. What's missing for surveying *future* papers in this lineage?

This is the most useful audit lens — the skill exists to be used on
papers not yet written.

**Missing tools:**

1. **A "this paper's submission date is X — what came out before X
   that they should have cited?" checklist.** For Lemur (submission
   date implied ~late 2025 from "anonymized for review"), the
   should-have-cited list includes:
   - Aardal et al. '24 (CRYPTO '24, online June 2024)
   - Anada-Fukumitsu-Hasegawa '25 (ProvSec '25, online ~early 2025)
   - Drake et al.'s successor "LeanSig" eprint 2025/1332

   The skill doesn't tell the reader to make this list. Adding a
   Step 2.5 "build the should-have-cited list from the submission
   date" would catch the Anada et al. omission.

2. **A "what came out AFTER submission that supersedes this paper?"
   step.** The current skill's Step 8 is for "summarising the
   field" but doesn't address "the auditor is reading this in 2026
   and the paper claims a 2026 publication date — what's already
   happened?". For Lemur in May 2026, this means citing the leanSig
   ecosystem and noting that Ethereum's PQ roadmap is moving
   towards hash+SNARK (and Falcon+LaBRADOR), not lattice-sync.

3. **A note on Ethereum-deployment relevance.** The skill says
   Lemur's 394 KB at N=2²⁰ is "the realistic range" — but for
   Ethereum specifically, the validator set is ~10⁶ ≈ 2²⁰, the
   slot frequency is 12 seconds, and the message size is fixed.
   Knowing these constants makes the "is this deployable?"
   question concrete. The current skill doesn't anchor the
   N choice to a deployment scenario.

**Concrete edits:**

- Add a **Step 2.5: "Sanity-check the citation horizon"** between
  Steps 2 and 3:

  > Note the paper's submission date. Build a 12-24 month
  > pre-submission list of works in the same column (using
  > eprint.iacr.org and Google Scholar). Cross-check against
  > the paper's reference list. Missing recent works in the
  > same column are red flags for the related-work section.

- Add a **Step 9 (after Step 8): "Post-submission supersession
  check"** for auditors reading the paper after submission:

  > Search for papers in the same paradigm that appeared
  > after the submission date. List anything that (a)
  > targets the same deployment scenario or (b) claims
  > better numbers at the same security level. Note which
  > of these the paper *could* have addressed in a revision
  > and which are genuinely post-camera-ready.

---

## 5. Other concrete edits

### `lemur-paper-analysis/SKILL.md`

- **Step 3, item 3 (security claims):** The phrasing "Chipmunk's
  published parameters fail this estimator by ~70 bits" implies
  112 - 70 ≈ 42 bits, which is in the right ballpark for the
  *median* cell but not the *worst* (which is ~23-27 bits, see
  `chipmunk_original_security_summary.txt`). Sharpen to:
  "Chipmunk's published parameters realize 23-40 bits under
  modern estimation (worst at large N), vs. their claimed 112."

- **Step 4 ("notation traps"):** Good as-is. Add one more:
  - **"τ values in Lemur vs Chipmunk":** Lemur's tables use τ ∈
    {12, 16, 20, 24} (Table 5), Chipmunk uses τ ∈ {21, 23, 24, 26}
    (chipmunk_original_security_summary.txt). When comparing
    cells, match by *N* not by *τ*, because *τ* is not the
    signer-count parameter.

- **Step 5 ("when comparing schemes"):** Bullet point on Rice/
  Golomb-coded sizes is correct but undersells. Lemur's Table 1
  has Lemur columns Rice-coded and Chipmunk columns raw,
  disclosed only in the caption. Promote this to a top-level
  warning:
  - "**Asymmetric encoding disclosure:** check that all
    columns in a comparison table use the same encoding.
    Lemur's Table 1 mixes Rice-coded Lemur with raw Chipmunk."

- **Step 6 ("what the paper does NOT prove"):** add:
  - "No comparison against Falcon+LaBRADOR (Aardal et al. '24)
    or Anada-Fukumitsu-Hasegawa '25 (ProvSec '25). The paper's
    related-work coverage stops at Drake et al. '25 for
    competing paradigms and Chipmunk '23 for lattice-sync;
    2024-2025 work in adjacent paradigms is uncovered."

### `lemur-prior-work-survey/SKILL.md`

- **Step 5 ("read the three flagship abstracts, then stop"):**
  The three are Squirrel, Chipmunk, Hint-MLWE. For an *audit*
  use case (not a summary), this is too few. Add a fourth:
  - "**Drake et al. '25 [7]** — eprint 2025/055 — read the
    intro and benchmark section. The Lemur dismissal of this
    paradigm is a single sentence; auditing that requires
    knowing Drake et al.'s actual numbers."

- **Step 6 ("competing-paradigm checks"):** The lattice
  non-synchronized subsection on Boneh-Kim says "Cite for
  completeness; do not compare numerically." Strengthen to:
  - "Cite for completeness; do not compare numerically.
    **Note that Boneh-Kim '20 achieves logarithmic aggregate
    size, which is better than any synchronized lattice
    scheme.** The non-synchronized model is the reason it
    isn't deployed, not the size."

- **Step 7 ("red flags"):** Add:
  - "**8. Same encoding in all comparison columns?** Rice-coded
    one column, raw the other, is a common slip. Check captions
    *and* the underlying script outputs."

---

## 6. What's good in the skills as-is

- The four-bucket triage (Direct lineage / Theoretical lever /
  Competing paradigm / Foundational tool) really does compress
  17 papers to 5. That's the skill's central value-add.
- The KOTS/HVC/Aggregation/BDS08 blueprint (Step 4) lets you
  read any future paper in the lineage and identify what's
  novel in 10 minutes. Tested on Lemur: novelty is in KOTS and
  HVC, aggregation/BDS08 inherited unchanged. Skill predicts
  this exactly.
- The notation-traps section (Step 4 of lemur-paper-analysis)
  is high-value because the paper switches between α, α_w, α_H,
  α freely. Without that warning a reader will conflate them.
- The "trap claims" categorisation (size deterministic / timing
  machine-dependent / security estimator-bound) is exactly right
  and matches the order of investigation a careful reviewer
  performs.
- The Boneh-Kim PDF being already fetched at
  `/workspace/tmp/boneh-kim-2020-agg-ots.pdf` saved a web round
  trip.

---

## 7. Summary of recommended edits

| Skill | Section | Change |
| --- | --- | --- |
| `lemur-prior-work-survey` | Step 1 (buckets) | Add "Bucket disputes" rubric; flag Boneh-Kim mis-placement |
| `lemur-prior-work-survey` | Step 1 buckets | Split "Direct lineage" into "construction" vs "style" |
| `lemur-prior-work-survey` | Step 2.5 (new) | "Sanity-check the citation horizon" pre-submission |
| `lemur-prior-work-survey` | Step 3 (paradigm map) | Add 4th bucket: PQ-sig + lattice-PoK (Aardal et al.) |
| `lemur-prior-work-survey` | Step 3 (paradigm map) | Add Anada et al. '25 under Lemur in synchronized column |
| `lemur-prior-work-survey` | Step 5 (flagship reads) | Add Drake et al. '25 as fourth must-read |
| `lemur-prior-work-survey` | Step 6 (competing paradigms) | Add Falcon+LaBRADOR subsection; sharpen Boneh-Kim note |
| `lemur-prior-work-survey` | Step 7 (red flags) | Add encoding-asymmetry red flag |
| `lemur-prior-work-survey` | Step 9 (new) | Post-submission supersession check |
| `lemur-paper-analysis` | Step 3 (security claims) | "23-40 bits" not "~70-bit gap" — be quantitative |
| `lemur-paper-analysis` | Step 4 (notation traps) | Add τ-cell-mismatch warning Chipmunk vs Lemur |
| `lemur-paper-analysis` | Step 5 (comparison) | Promote Rice asymmetry warning to top level |
| `lemur-paper-analysis` | Step 6 (paper does NOT prove) | Note absent Falcon+LaBRADOR, Anada et al. comparisons |

Of these, the highest-value edits are:

1. **Adding the Falcon+LaBRADOR bucket to the paradigm map**
   (current map is genuinely incomplete).
2. **Adding the citation-horizon Step 2.5** (catches the Anada
   et al. and other 2024-2025 omissions).
3. **Adding the Bucket-disputes rubric** (Boneh-Kim mis-placement
   is exactly the kind of move future papers will repeat).

Without these three, the next user of the skill will repeat the
same audit and arrive at the same gaps — i.e., the skill won't
have learned from this audit pass.
