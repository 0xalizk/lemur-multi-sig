# Reviewer 1 — verification of Auditor 1's `review.md` checkup

Independent spot-checks against `/workspace/repo/assessment/review.md`
(1 142 lines on disk). Line numbers below cite the same file.

---

## Per-claim verdicts

### 1. TOC ↔ `id="sec-…"` symmetry (Auditor §A) — **AGREE**

- `grep -oE '"#sec-[0-9-]+"' review.md | sort -u | wc -l` → **27**
- `grep -oE 'id="sec-[0-9-]+"' review.md | sort -u | wc -l` → **27**
- `diff` of the two stripped sets (sed away `"#…"` / `id="…"`) →
  exit 0, empty output. Every TOC href lands on an `id`, no orphans.

Verified.

### 2. `ref-4b` orphan still fixed (Auditor §B last bullet) — **AGREE**

- `grep -n 'ref-4b' review.md` → empty.
- `grep -n 'ref-4' review.md`: only `ref-4` (line 102), `ref-4c`
  (line 420), `ref-4d` (line 652); resolution arrow on line 865:
  `[↩](#ref-4) [↩](#ref-4c) [↩](#ref-4d)`. The 4-family is exactly
  `{4, 4c, 4d}`, no `4b`. Checkup_2 fix has held.

Verified.

### 3. D1 — §6 ratio row says "18 : 1" (line 504) — **AGREE on the bug, REVISE the recommendation**

- Line 504 verbatim:
  `| Secure aggregation : batch verify (N=2¹⁰) | 19 : 1 (both paper Table 2) | 18 : 1 | ✓ |`
- Line 494: measured aggregation = **1.41 s** (= 1 410 ms).
- Line 495: measured batch verify = **25.63 ms** (zero-fixture, 5-rep mean).
- Arithmetic check:
  - `1410 / 79.31 = 17.78` → matches audit's "old-number math".
  - `1410 / 25.63 = 55.01` → matches audit's "new-number math".
  - `567 / 30.1 = 18.84` → audit's "paper baseline 18.83" rounds correctly.
- `grep -n '79.31' review.md` → 0 matches. The stale denominator
  is no longer in the file but the ratio it produced still is.

**The 18:1 cell is mathematically stale: it embeds the retired 79.31 ms.**

Reviewer note Auditor 1 missed (but flagged in Dr5): the asymmetry
goes deeper than they let on. The aggregation row (494, `1.41 s`)
and the batch-verify row (495, `25.63 ms`) come from different
runs — the 25.63 is explicitly tagged "zero-fixture, 5-rep mean"
while 1.41 s carries no such tag. If 25.63 is the refreshed
measurement and 1.41 s is from the older `bench --fast` session
(which also produced the now-replaced 79.31 ms), then the
aggregation row is *also* session-mismatched, and the right fix
is to re-measure aggregation under the same protocol as
batch-verify, not just to update one ratio. The audit's
"55 : 1 ✓ with a parenthetical" recommendation papers over a
session-consistency problem rather than fixing it.

**Verdict: AGREE that line 504's "18 : 1" is arithmetically stale.
REVISE-TO** the stronger fix: either (a) re-measure aggregation
under the zero-fixture/5-rep protocol and propagate, or (b) tag
both numbers with their measurement provenance and explicitly
note that the ratio mixes two protocols. Auditor 1's "tighten the
denominator and keep the ✓" understates the issue.

### 4. D2 — fn-9 framing (lines 888–893) — **AGREE**

Lines 888–893 read:

```
9. Anada, Fukumitsu, Hasegawa. "Tightly Secure Lattice-Based
Synchronized Aggregate Signature in Standard Model." ICISC 2024,
Springer LNCS 15596. DOI [10.1007/978-981-96-5566-3_4](…).
First lattice-based synchronized aggregate signature secure in the
standard model (not ROM). [↩](#ref-9) [↩](#ref-9b) [↩](#ref-9c) [↩](#ref-9d)
```

Confirmed: only the proof-model axis appears. The certified-key
threat-model caveat that §2 (lines 152–158) and §9.1 (lines 670–681)
both establish is absent. Four forward refs (`ref-9`, `9b`, `9c`,
`9d` — lines 119, 154, 163, 672) target this footnote; a reader
following the citation from the §1 TLDR (line 119) or the §2
tree caption (line 145) lands on a one-axis description.

Verified. Auditor 1's one-sentence patch is appropriate.

### 5. D3 — §5.4 `diff <(jq …)` cosmetic drift — **AGREE**

Lines 385–411 verified. Code block (387–391) shows:
```
diff <(jq -S . py.json) <(jq -S . rs.json)
```
Result block (395–404) shows labelled `MATCH` lines (`pp: MATCH`,
`signatures: MATCH`, …). `diff` of two `jq -S` streams produces
either no output or unified-diff hunks — never `KEY: MATCH` lines.
The displayed output is the output of *some other script* (a Python
JSON-walk or equivalent). This is the same drift Auditor 1 attributes
to checkup_2; the fix has not landed.

Verified. Auditor 1's two-way fix (replace the command, or replace
the output) is correct.

### 6. Drift items

**Dr1. TLDR line 119 one-axis Anada description — AGREE.**
Line 119 verbatim: `ICISC '24 standard-model lattice synchronized<sup id="ref-9">[9](#fn-9)</sup>`.
No threat-model qualifier. The audit's 4-word patch
("standard-model / certified-key lattice synchronized") is defensible
but minor. Aesthetic.

**Dr2. §2 tree caption line 145 — AGREE.**
Line 145 verbatim: `                              │      (standard model)`.
The Anada branch of the ASCII tree carries only `(standard model)`.
Two-axis framing introduced 7 lines below in the prose (lines
152–158) is not reflected in the caption. Concur with the
recommended caption tweak.

Also worth flagging (Auditor 1 caught this in Dr3): line 163's
"standard-model sibling" phrase is technically licensed by the
prose at lines 153–158 but reads one-sidedly out of context.

**Dr4. §10 bucketing — PARTIAL AGREE.**
Read §10 lines 718–787 carefully. Bucket definitions are implicit:
"unresolvable" lead-in (line 720) reads "remain unresolvable
without resources beyond this review's scope", and the lower
bucket (line 758) is "limitations of this *review's* scope".
The audit is correct that under those definitions:

- "**N=2²⁰ end-to-end not measured**" (lines 750–756) explicitly
  identifies the obstacle as host RAM (`8 GiB`), then says "A
  16+ GiB host would close both" → review-scope, not unresolvable.
  Strong case to move.
- "**RHF threshold is itself a choice**" (lines 732–740) — the
  obstacle here is partly intrinsic (no party has currently
  re-fit Chipmunk at the new RHF) and partly resource (this
  reviewer didn't have Sage at first; the text now notes Sage
  *was* installed). Mixed. Less clear-cut than Auditor 1 implies.
  DISPUTE the audit's claim this is "closer to a scope item";
  the "what trade-offs" question is genuinely open at the
  field level, not just for this reviewer.
- "**Reduction loss not absorbed in parameters**" (lines 768–772):
  this is a paper-side observation; agree with audit it would
  fit in §3 or in the unresolvable bucket better.
- "**Lemma 4.1 ℓ=1 restriction**" (lines 774–777): paper-side
  proof-scope observation; agree it does not fit the "this
  review's scope" framing.
- "**Implementation profile coverage**" (lines 784–787): about
  what the artifact ships, not what the review did; partial
  case for moving.

Net: 3 of 5 items the audit flagged are defensibly mis-bucketed,
2 are debatable. The audit's recommendation to either re-bucket
or re-title the lower bucket as "Items left for follow-up beyond
this review's scope" is sound and the cheaper of the two fixes.

### 7. §4 LOC values (lines 264–266) — **AGREE**

- Line 264: "(`lemur-py/`, 3 629 LOC)".
- Line 265: "(`lemur-rs/src/`, 9 838 LOC)".

Both present, no competing values (`grep '3,600\|~9 kLOC\|9 000 LOC'`
→ 0). The audit's "unique and not stale" claim verified.

---

## Things the audit got right that should be applied

1. **D1 line 504 ratio** is stale (`55.01` not `18`).
2. **D2 fn-9** needs the certified-key caveat sentence.
3. **D3 §5.4** is the same drift checkup_2 already flagged;
   either the code block or the result block has to change.
4. **Dr1 + Dr2**: line 119 TLDR Anada one-liner and line 145
   tree caption both describe Anada one-sidedly on the proof-model
   axis — minor but easy 5-word fixes.
5. **Dr4 §10**: at least three of the five flagged items are
   mis-bucketed under the audit's reading of the bucket
   definitions; renaming the lower bucket is the lower-cost fix.

## Things the audit got wrong / understates

1. **D1 fix scope**: Auditor 1 recommends correcting the
   denominator (`1410/25.63 = 55`) and keeping the ✓. The
   deeper issue is that the *aggregation* number (1.41 s) and
   the *batch-verify* number (25.63 ms) appear to come from
   different measurement protocols — the 25.63 is tagged
   "zero-fixture, 5-rep mean", the 1.41 s carries no protocol
   tag and is co-resident with the original `bench --fast`
   session that also produced the (now-removed) 79.31 ms.
   The clean fix is to re-measure aggregation under the same
   protocol, not just to update the ratio. Auditor 1 hints at
   this in Dr5 but doesn't connect it to the D1 recommendation.

2. **Dr4 RHF item**: Auditor 1 places this on the same footing
   as the N=2²⁰ item, but the RHF question has an intrinsic
   "what does Chipmunk re-fit look like at corrected security"
   open-question component that is *not* purely review-scope.
   I'd keep it where it is, against the audit's recommendation.

## Things the audit missed

1. **§2 line 163 "standard-model sibling" phrasing**: the audit
   names this in Dr3 but doesn't recommend it for the
   "Recommendations" section. Should be included in the same
   sweep as Dr1/Dr2 — 3 lines, one consistent framing.

2. **§6 line 494 — aggregation row provenance**: no measurement
   protocol tag despite sitting next to a row that has one
   (line 495). Even if the audit fixes only the ratio at line
   504, a one-word annotation on line 494 ("`bench --fast`
   session" or similar) would prevent the next reviewer from
   asking the same question.

3. **fn-51 family wiring**: Auditor 1 verifies the back-arrow
   set on line 1133 covers `ref-51 / 51b / 51c` and accepts
   "presumably the §5.6 anchor" for ref-51c. Direct verification:
   forward refs are at lines 259 (`ref-51`, §3), 537 (`ref-51b`,
   §6.1), and 713 (`ref-51c`, §5.6) — confirmed, all three resolve.
   Audit was right, just incomplete in the spot-check.

4. **No `strictly stronger` substring anywhere** (audit §E).
   Independently confirmed: `grep -i 'strictly stronger'` → 0.
   Worth noting that the obsolete framing has been excised
   cleanly; the only residual drift is descriptive (Dr1/Dr2/Dr3).

5. **Header back-to-top sweep** (audit §H): independently
   confirmed with `grep -nE '^####? ' review.md | grep -v '\[↑\](#toc)'`
   → 0 lines. Every `###` and `####` header on disk has the
   back-to-top arrow.

---

## Summary table

| Audit claim | Verdict | Note |
| --- | --- | --- |
| TOC ↔ id 27=27 | AGREE | independently reproduced |
| Footnote graph 51/51, 68/68 | AGREE | reproduced via grep |
| ref-4b orphan still fixed | AGREE | grep returns empty |
| D1 line 504 stale (18:1 vs 55:1) | AGREE on bug, REVISE fix | re-measure, not just re-divide |
| D2 fn-9 missing certified-key axis | AGREE | one-sentence patch correct |
| D3 §5.4 diff/jq vs MATCH | AGREE | unchanged from checkup_2 |
| Dr1 line 119 TLDR | AGREE | minor, defensible |
| Dr2 line 145 tree caption | AGREE | minor |
| Dr3 line 163 "sibling" | AGREE | should be in Recommendations |
| Dr4 §10 bucketing | PARTIAL AGREE | 3 of 5 items defensible |
| §4 LOC 3 629 / 9 838 | AGREE | unique, no stale variants |
| §6 row internal arithmetic | AGREE | 302.8/3850, 25.63/30.1, 3.85×26043 all check |
| Anada §2 ↔ §9.1 prose | AGREE | both blocks use 2-axis framing |
| Every header has [↑](#toc) | AGREE | reproduced |

Total: 11 unambiguous AGREE, 1 AGREE-with-stronger-fix (D1),
1 PARTIAL AGREE (Dr4). No outright DISPUTEs on the audit's
substantive findings. The audit's one weakness is that D1's
recommendation treats a measurement-provenance issue as a
ratio-arithmetic issue.
