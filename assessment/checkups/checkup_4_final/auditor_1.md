# Checkup 4 final — Auditor 1: internal consistency of `review.md`

Scope: `/workspace/repo/assessment/review.md` (1 142 lines).
No other files audited; other auditors cover README, `submission/`, and
sibling checkup files.

All line numbers in this report refer to the file as it currently
stands on disk.

---

## VERIFIED claims

### A. Section-anchor (`sec-N`) symmetry — clean

- `grep -oE '"#sec-[0-9-]+"' review.md | sort -u | wc -l` → **27**
- `grep -oE 'id="sec-[0-9-]+"' review.md | sort -u | wc -l` → **27**
- Stripped diff of the two sorted sets (sans `"#…"` vs `id="…"` syntax):
  `diff <(… | sed 's/"#//;s/"//') <(… | sed 's/id="//;s/"//')` →
  **empty** (exit 0). Every TOC `href` lands on an `id`, every `id` is
  referenced from the TOC.
- The 27 anchors cover §1–§11 plus 4.1–4.4, 5.1–5.6, 6.1, 7.1–7.3,
  9.1–9.2 (line 84–790). §10 renamed to "Open issues" (line 718) —
  TOC line 39 matches.

### B. Footnote integrity — clean

- `id="fn-N"` count: **51** (fn-1…fn-51, all integers, no gaps).
- Forward `(#fn-N)` references: 51 unique values, set-equal to the
  declared ids (`diff` empty).
- `id="ref-N[a-d]?"` count: **68** unique anchors.
  Multi-cite expansions: ref-1/1b, ref-2/2b, ref-4/4c/4d, ref-5/5b,
  ref-7/7b, ref-8/8b/8c, ref-9/9b/9c/9d, ref-20/20b, ref-23/23b,
  ref-45/45b, ref-46/46b, ref-51/51b/51c. Total = 51 base + 17 extras
  = 68. Matches.
- Back-arrow `[↩](#ref-N[a-d]?)` count: 68, set-equal to ref-id set
  (`diff` empty). Every back-arrow resolves; every ref-id has a
  corresponding back-arrow.
- **`ref-4b` orphan: confirmed STILL FIXED.** No `4b` substring
  anywhere in the file. The 4-family is `ref-4, ref-4c, ref-4d`, all
  resolved on line 865 (`[↩](#ref-4) [↩](#ref-4c) [↩](#ref-4d)`). The
  checkup_2 commit a9ded67 fix has held.

### C. §10 structure — present as described

Line 718–757: "**Substantively unresolvable**" prose lead-in, then
4 bulleted items: no Chipmunk runtime, RHF threshold, Drake timing
self-undercut, N=2²⁰ end-to-end not measured.

Line 758–787: "limitations of this *review's* scope" lead-in, then
5 items: proof not formally audited, reduction loss not absorbed,
Lemma 4.1 ℓ=1 restriction, side-channel not assessed, profile
coverage.

Item-count claim (4 + 5) matches.

### D. RHF threshold consistency

`RHF` appears at lines 729, 732, 736, 737, 967. The §10 unresolvable
item (line 732, `RHF ≤ 1.0045`) cites the same threshold as fn-23
(line 967): "λ = 128 and root Hermite factor RHF ≤ 1.0045". Both
quote the paper's Table 5 caption. Framing consistent.

### E. Anada framing — §2 and §9.1 align

- §2 paradigm prose (lines 152–158) correctly describes the
  (proof-model × threat-model) grid: "Anada targets the *standard
  model* (stronger proof-model than Lemur's ROM) but uses the
  *certified-key* threat model (weaker than Lemur's rogue-key-safe).
  They occupy a different cell of the (proof-model × threat-model)
  grid, not a strict ordering."
- §9.1 Anada bullet (lines 670–681) fully restates this:
  "**stronger on the proof-model axis (standard model, not ROM)**
  but **weaker on the threat-model axis (certified-key model, not
  rogue-key-safe)**" + the apples-to-apples requirement framing.
- Neither block uses the obsolete "strictly stronger" phrasing.
  Verified by `grep -i 'strictly stronger'` → 0 matches.

### F. §6 refreshed timings — internally numerically consistent

- Online sign 302.8 µs (line 489) ↔ "Online sign : stateful sign
  ≈ 1 : 12" parenthetical (line 506). 302.8 / 3850 = 0.0786 ≈ 1/12.7.
  Consistent.
- Stateful sign 3.85 ms (line 491) ↔ ratio `1 : 26 043` (line 503):
  3.85 × 26 043 = 100 265.55 ms = 100.27 s = **1.671 min**,
  displayed as "1.7 min" in the Full sign row (line 490). Consistent
  to the displayed precision.
- Batch verify zero-fixture 25.63 ms (line 495) vs baseline 30.1 ms:
  ratio 0.85× — stated and arithmetically correct (25.63/30.1 = 0.851).

### G. §4 intro LOC values — unique and not stale

- Line 264–266: "3 629 LOC" Python, "9 838 LOC" Rust.
- `grep '3,600\|3\.6 kLOC\|~9 kLOC\|9 000 LOC'` → 0 matches.
- Stale values 304 µs, 3.91 ms, 79.31 ms also absent
  (`grep '304 µs\|3\.91 ms\|79\.31 ms'` → 0 matches).

### H. `[↑](#toc)` back-to-top on every header

`grep -nE '^####? ' review.md | grep -v '\[↑\](#toc)'` → empty.
Every `###` and `####` header line (and `### Footnotes` at line 836)
has the back-to-top link.

### I. Footnote 51 family — fully wired

fn-51 declared at line 1122; back-arrows at line 1133 cover
`[↩](#ref-51) [↩](#ref-51b) [↩](#ref-51c)`. The three forward refs
are at line 259 (ref-51), line 537 (ref-51b), and presumably the
§5.6 anchor (verified by ref-id count = 68).

---

## DISCREPANCIES

### D1. §6 "Secure aggregation : batch verify" measured ratio is stale

Line 504:
```
| Secure aggregation : batch verify (N=2¹⁰) | 19 : 1 (both paper Table 2) | 18 : 1 | ✓ |
```

The "Measured" column says **18 : 1**, but with the refreshed numbers
from §6:

- Measured secure aggregation N=2¹⁰: **1.41 s** (= 1 410 ms; line 494)
- Measured batch verify N=2¹⁰: **25.63 ms** (line 495)

→ 1 410 / 25.63 = **55.0**, not 18.

Cross-check against the *previous* batch-verify value the user's
prompt flagged as stale (**79.31 ms**): 1 410 / 79.31 = 17.78 ≈ 18:1.
So the ratio is *arithmetically* consistent with the old, *not* the
refreshed, batch-verify number.

**Correction:** the measured ratio should read **55 : 1** (or
"≈ 55 : 1"). The ✓ verdict needs re-examination: paper baseline ratio
is 18.83 : 1 (567/30.1); measured ratio 55 : 1 is ~2.9× larger. The
gap reflects the zero-fixture batch-verify path being *faster* than
the linear thread-count scaling predicted, which actually agrees with
the "0.85× verdict" on the verify row — but the ratio row's ✓
becomes nontrivial to defend and should be re-framed (e.g., "55 : 1
— zero-fixture verify path inflates the ratio; non-zero-fixture
verify would land near baseline").

### D2. Footnote 9 omits the certified-key threat-model caveat

Line 888–893:
```
9. Anada, Fukumitsu, Hasegawa. "Tightly Secure Lattice-Based
Synchronized Aggregate Signature in Standard Model." ICISC 2024,
Springer LNCS 15596. DOI [10.1007/978-981-96-5566-3_4](…).
First lattice-based synchronized aggregate signature secure in the
standard model (not ROM). [↩](#ref-9) [↩](#ref-9b) [↩](#ref-9c) [↩](#ref-9d)
```

This footnote is the resolved target for **four** forward references
(ref-9, ref-9b, ref-9c, ref-9d) which appear in §1 (line 119), §2
(lines 154, 163), and §9.1 (line 672). Three of those callsites
embed the now-canonical "different cell of the (proof-model ×
threat-model) grid" / "certified-key model" framing. A reader
following the [9] callout from §1's TLDR or §2's tree-diagram caption
("(standard model)" on line 145) lands on fn-9 and sees *only* the
proof-model axis — the threat-model caveat that distinguishes Anada
from Lemur is absent.

**Correction:** extend fn-9 with one sentence, e.g.:
"Secure in the *standard model* (stronger proof-model axis than
Lemur's ROM) but in the *certified-key* threat model (weaker
threat-model axis than Lemur's rogue-key-safe); the two papers
occupy different cells of the (proof-model × threat-model) grid —
see §2 and §9.1."

### D3. §5.4 narrative drift — `diff <(jq …)` shown, custom Python output displayed

Lines 387–404. The code block shows:
```
diff <(jq -S . py.json) <(jq -S . rs.json)
```
but the displayed result block is the custom MATCH-style output:
```
pp:           MATCH
signatures:   MATCH
…
```

`diff` of two sorted JSON streams either prints nothing on success or
prints unified-diff hunks on mismatch — it does not produce
labelled-key `MATCH` lines. This is the same drift the checkup_2
audit flagged; it has not been corrected. The MATCH listing is the
output of a Python equality-on-parsed-JSON walk over the two files
(or an equivalent script), not of `jq`+`diff`.

**Correction:** either (a) replace the shown command with the actual
command that produced the MATCH listing (e.g., a small Python
snippet), or (b) keep the `diff <(jq …)` invocation and replace the
displayed block with the bare statement "diff is empty — both files
sort-key-canonicalised to byte-identical JSON" plus the explanatory
sentence on line 406–411.

---

## DRIFT items (recent edits left stale references)

### Dr1. §1 TLDR Anada one-liner

Line 119 still describes Anada bibliographically as "**ICISC '24
standard-model lattice synchronized**", without the certified-key
qualifier introduced in §2 and §9.1. For TLDR brevity this is
defensible, but it leaves a reader who stops at §1 with the
"stronger sibling" mental model that §2/§9.1 explicitly walks back.
A 4-word inline tweak ("standard-model / certified-key lattice
synchronized") would close the drift.

### Dr2. §2 paradigm tree caption row

Line 145:
```
                                    │      (standard model)
```
The caption on the Anada branch of the ASCII tree calls out only
the proof-model axis. After the certified-key framing update, the
caption is one axis out of two. Suggested edit:
`(standard model, certified-key)` or `(std-model / cert-key)`.

### Dr3. "Standard-model sibling" phrase still used in §2

Line 162–163:
```
PQ-sig+lattice-PoK bucket (Aardal et al.) and the standard-model
sibling (Anada et al.). …
```
The word "sibling" appears 7 times (lines 154, 163, 348, 353, 672,
1008, 1015 — only 154, 163, 672 are about Anada). The §2 prose on
153–158 fully unpacks the threat-model caveat right before line 163,
so "standard-model sibling" on 163 is technically licensed by
context. But the phrase, read in isolation, lands close to the
"strictly stronger" framing that was supposed to be retired.
Minor: consider "standard-model / certified-key sibling" or
"different-cell sibling" on 163 to remove ambiguity.

### Dr4. §10 unresolvable-vs-limitations bucketing

The split is plausible but two pairs are debatable; flagging for
authorial review rather than asserting they are wrong:

- **N=2²⁰ end-to-end not measured** (lines 750–756, unresolvable
  bucket): the item itself says "A 16+ GiB host would close both
  the `N=8192` aggregation cell and the realized aggregate-size
  variance check." That phrasing explicitly identifies the obstacle
  as **this review's host resources**, which is the textbook
  definition of the "limitations of this review's scope" bucket.
  Recommend moving to the lower bucket, or rewriting the obstacle as
  paper-side (e.g., "the paper extrapolates linearly without
  measurement" — then it's a substantive paper-side observation).

- **RHF ≤ 1.0045 threshold is itself a choice** (lines 732–740,
  unresolvable bucket): the bullet says "would let Chipmunk re-fit
  cleanly under the modern estimator, with what size-and-security
  trade-offs is an open empirical question this review cannot
  answer." Like the previous item, the obstacle is identified as a
  resource limit on *this review*, not an intrinsic obstruction.
  A well-resourced reviewer (Sage + time) could re-run Chipmunk at
  RHF ≤ 1.005 and produce the comparison. So in the strict
  unresolvable-vs-scope sense, this is closer to a scope item.

- **Reduction loss not absorbed in parameters** (lines 768–772,
  scope bucket): this is a paper-side observation about a parameter
  choice the paper made — not a limitation of *this review*. It
  would be at home in §3 ("what the paper proves and what it does
  not") or in the unresolvable bucket as "Lemur ships parameters
  that do not absorb the multi-user loss; the paper acknowledges
  this, and the choice cannot be undone within the paper's existing
  parameter tables."

- **Lemma 4.1 ℓ=1 restriction** (lines 774–777, scope bucket):
  similarly a paper-side proof-scope observation, not a review-scope
  limitation. Better placed in §3.

- **Implementation profile coverage** (lines 784–787, scope bucket):
  this is about what the *implementation* ships, not what the
  *review* did. Belongs in §3 or a "what the artifact ships"
  paragraph.

Net effect: the heading split reads cleanly but several items leak
across the boundary. Either tighten the bucket definitions
(e.g., "unresolvable" = "no party can resolve without redesign";
"scope" = "this reviewer's host/time/skill ran out") and re-place
items, or re-title the lower bucket "Items left for follow-up
beyond this review's scope" (broader, less prescriptive).

### Dr5. §6 batch-verify row carries two distinct measurements

Line 495 lists 25.63 ms ("zero-fixture, 5-rep mean"); line 504's
ratio uses a *different* underlying value (79.31 ms or similar — see
D1). The two rows of §6 are using inconsistent batch-verify
measurements. Pick one and propagate.

---

## RECOMMENDATIONS

Concrete, smallest-diff first:

1. **Fix the §6 ratio (D1).** Edit line 504 to read either
   `1 410 / 25.63 ≈ 55 : 1` with a clarifying parenthetical, OR
   refresh the underlying batch-verify number on that row to match
   line 495. The current `18 : 1 ✓` does not survive the refreshed
   §6 numbers.

2. **Update footnote 9 (D2).** Add one sentence at end of fn-9
   restating the certified-key axis and pointing to §2/§9.1.

3. **Fix §5.4 (D3).** Either change the shown command to a Python
   equality walk, or change the shown output block to a `diff`-style
   "no output" report.

4. **Tweak §1 / §2 Anada one-liners (Dr1, Dr2, Dr3).** Three small
   inline edits land the (proof-model × threat-model) framing
   wherever Anada is named: line 119 (TLDR), line 145 (paradigm
   tree caption), line 163 ("standard-model sibling" prose). None
   exceeds a 5-word edit.

5. **Re-bucket §10 items (Dr4).** Either tighten the dividing
   definition and move items across, or re-title the lower bucket so
   items 5–9 can all live there without straining.

6. **No action needed:** footnote graph (51 fn, 68 ref), TOC↔anchor
   set, ref-4b orphan, LOC values, online-sign-to-stateful-sign
   ratio, fn-51 multi-cite, every `[↑](#toc)` back-to-top arrow.
   These are all verified clean.

---

## Summary table

| Check | Status |
| --- | --- |
| TOC ↔ `id="sec-…"` | clean (27 = 27) |
| Footnote forward (`#fn-N`) ↔ id | clean (51 = 51) |
| ref-N ids ↔ back-arrows | clean (68 = 68) |
| ref-4b orphan | still fixed |
| §10 split (4 + 5) | present; bucket-fit debatable (Dr4) |
| RHF cross-ref consistency | clean |
| Anada §2 ↔ §9.1 prose | clean |
| Anada fn-9 | drift (D2) |
| Anada §1 TLDR / §2 tree caption | drift (Dr1, Dr2) |
| §6 timings (302.8 µs, 3.85 ms, 25.63 ms) | clean within rows |
| §6 ratio row 504 | **stale (D1, 55:1 not 18:1)** |
| §4 intro LOC (3 629 / 9 838) | clean |
| §5.4 `diff`/`jq` vs MATCH listing | drift (D3, persists from checkup_2) |
| Every header has `[↑](#toc)` | clean |
| fn-51 multi-cite | clean |

Two corrections (D1, D3) are arithmetic / textual bugs;
one (D2) is a one-sentence reframing;
the rest are 1–5 word inline drift tweaks.
