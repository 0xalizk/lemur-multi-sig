# Auditor 3 — Consolidated review.md structural audit

**Scope:** `/workspace/repo/assessment/review.md` (1128 lines). Only this file; not
the top-level README, not the submission folder (other auditors handle those).

**Verdict at a glance.** The document is *near pristine* on structural
integrity: TOC↔anchor balance, footnote↔ref balance, submission link targets,
and heading back-arrows are all internally consistent except for exactly one
broken back-arrow inside footnote 4 (a dangling `[↩](#ref-4b)`). Numeric
self-consistency between §3, §5.2, §5.3, §5.5, §6, §10, and the §1 TLDR is
solid; the §6 thread-count ratios reproduce to within a rounding digit when
recomputed from the table's own absolute values. The Sage §5.6 section is
freshly added and the on-disk artifacts have mtimes consistent with the
claimed re-run window. Earlier-round FC2 flags about §6's "Paper (24 thr)"
column-header attribution appear to have been incorporated (the header has
been renamed to "Baseline (24 thr)" with an explicit "Source" column).

---

## VERIFIED structural claims

### V1. TOC anchors balance perfectly (27↔27)

```
grep -oE '"#sec-[0-9-]+"' assessment/review.md | sort -u | wc -l       → 27
grep -oE 'id="sec-[0-9-]+"' assessment/review.md | sort -u | wc -l    → 27
diff <(...href...) <(...id...)                                         → empty
```

Targets and anchors: `sec-1, sec-2, sec-3, sec-4, sec-4-1..4-4, sec-5,
sec-5-1..5-6, sec-6, sec-6-1, sec-7, sec-7-1..7-3, sec-8, sec-9,
sec-9-1, sec-9-2, sec-10, sec-11` — every entry in the TOC table (lines
11–41) resolves to an inline `<a id="...">` on the corresponding heading.
`id="toc"` declared at line 3; `id="footnotes"` declared at line 822.

### V2. Every section/subsection heading carries the back-arrow

All 27 headings end with `[↑](#toc)`. Verified by inspection of the heading
listing — lines 84, 123, 174, 259, 265, 274, 285, 293, 308, 310, 331, 362,
382, 410, 428, 465, 519, 541, 546, 560, 575, 591, 635, 656, 681, 710, 776
plus the Footnotes heading at 822. No heading missing the arrow.

### V3. Footnote anchor set is contiguous 1..51

```
grep -c '<a id="fn-' review.md                  → 51
grep -oE 'id="fn-[0-9]+"' | sort -u | wc -l     → 51
```

Footnote IDs `fn-1` through `fn-51` all declared exactly once. The in-body
citation `<sup id="ref-N">[N](#fn-N)</sup>` form is used consistently;
all 51 distinct `#fn-N` targets cited from the body resolve to footnotes.

### V4. In-body ref↔back-arrow balance (one exception, see D1)

```
grep -oE 'id="ref-[0-9]+[a-z]?"' | sort -u | wc -l           → 68
grep -oE '\(#ref-[0-9]+[a-z]?\)' | sort -u | wc -l           → 69
```

68 in-body anchors versus 69 back-arrow link targets. The 1-entry surplus
on the back-arrow side is `(#ref-4b)`, which has no matching `id="ref-4b"`
(see D1).

Footnotes-cited-multiple-times distribution: 51 base IDs + 13 suffixed
duplicates (`-1b, -2b, -5b, -7b, -8b, -8c, -9b, -9c, -9d, -20b, -23b, -45b,
-46b, -51b, -51c`). Plus `ref-4c, ref-4d` (without a `ref-4b` between them).
Total declared: 68. All in-body suffix patterns are well-formed.

### V5. Submission-relative links resolve

All 9 distinct `../submission/...` targets exist on disk:

| Link target | Resolves |
| --- | --- |
| `../submission/code/` | yes (dir) |
| `../submission/code/parameter/` | yes (dir) |
| `../submission/code/parameter/chipmunk_original.sage` | yes |
| `../submission/code/parameter/chipmunk_original_security_summary.txt` | yes |
| `../submission/code/parameter/chipmunk_param.sage` | yes |
| `../submission/code/parameter/chipmunk_summary.txt` | yes |
| `../submission/code/parameter/lemur_param.sage` | yes |
| `../submission/code/parameter/summary.txt` | yes |
| `../submission/report.pdf` | yes |

### V6. External URLs are well-formed and not truncated

11 distinct URLs found, all terminate cleanly on `.pdf`, a numeric eprint
identifier, or a Springer DOI fragment ending in a digit/chapter slug. No
URL ends a line with a dangling slash, ellipsis, or unmatched bracket.

```
https://crypto.stanford.edu/~skim13/agg_ots.pdf
https://eprint.iacr.org/2022/694
https://eprint.iacr.org/2023/1820
https://eprint.iacr.org/2023/623
https://eprint.iacr.org/2024/311
https://eprint.iacr.org/2025/055
https://eprint.iacr.org/2025/1332
https://link.springer.com/chapter/10.1007/3-540-45682-1_30
https://link.springer.com/chapter/10.1007/978-3-032-15541-2_1
https://link.springer.com/chapter/10.1007/978-981-96-5566-3_4
https://sh.rustup.rs
```

### V7. Section/subsection numbering is gap-free

| Top | Subs present | Subs expected per TOC | Match |
| --- | --- | --- | --- |
| 1 | none | none | ✓ |
| 2 | none | none | ✓ |
| 3 | none | none | ✓ |
| 4 | 4.1–4.4 | 4.1–4.4 | ✓ |
| 5 | 5.1–5.6 | 5.1–5.6 | ✓ |
| 6 | 6.1 | 6.1 | ✓ |
| 7 | 7.1–7.3 | 7.1–7.3 | ✓ |
| 8 | none | none | ✓ |
| 9 | 9.1–9.2 | 9.1–9.2 | ✓ |
| 10 | none | none | ✓ |
| 11 | none | none | ✓ |
| Footnotes | — | — | ✓ |

Top-level sequence 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, Footnotes — no
duplicates, no gaps.

---

## DISCREPANCIES

### D1. Dangling back-arrow `[↩](#ref-4b)` in footnote 4 (line 851)

Footnote 4 (lines 847–851) is cited from four locations in the body:

- line 102 — `<sup id="ref-4">`
- line 417 (§5.5) — `<sup id="ref-4c">`
- line 649 (§9 table, footnote-style cell) — `<sup id="ref-4d">`

…which is only **three** distinct ref anchors, yet the footnote's
back-arrow trail at line 851 reads:

```
specifically: [23.1, 38.8]. [↩](#ref-4) [↩](#ref-4b) [↩](#ref-4c) [↩](#ref-4d)
```

`#ref-4b` has no matching `id="ref-4b"` anywhere in the document
(confirmed via `comm -23` against the anchor set; only `ref-4b` is in the
"unmatched-back-arrow" diff). Clicking that arrow will scroll to nowhere.

**Likely cause:** during an edit round a fourth citation of fn-4 was
removed but the matching `[↩](#ref-4b)` trail was not pruned. The
suffix sequence `4 → (missing 4b) → 4c → 4d` makes this almost certainly
a stale leftover, not a missing in-body cite.

**Fix:** Remove the orphan back-arrow at line 851; the trail should
read `[↩](#ref-4) [↩](#ref-4c) [↩](#ref-4d)`.

### D2. (minor / non-blocking) §5.4 narration vs underlying tooling

Not flagged by the structural sweep, but called out by FC2 reviewer round:
review.md §5.4 (lines 385–387) writes the verification command as
`diff <(jq -S . py.json) <(jq -S . rs.json)`. The underlying README
walks Python's `==` over the parsed JSON dicts, not `diff`+`jq`. The
result table immediately below (lines 393–401) shows per-field MATCH
status which is exactly what the Python equality check emits, so the
narrative is internally coherent — just the command shown is not the
script's literal command. Cosmetic; no broken anchor, no wrong number.

---

## NUMERIC CONSISTENCY CHECK

### N1. Aggregate-signature sizes: 201.2 / 283.5 / 394.4 KB

Cross-section check across the four locations the audit requested:

| Location | Lines | Cited values |
| --- | --- | --- |
| §3 "Size claims" | 233–234 | 201.2 / 283.5 / 394.4 KB |
| §5.2 table | 347 | 201.2 KB (N=2¹⁰); ratio table footer 201.236 in fn-31 |
| §5.3 table | 370–372 | 201.2 / 283.5 / 394.4 KB |
| Top-level README | 81 | 201.2 / 283.5 / 394.4 KB |

All four match exactly. Footnote 31 (line 996) re-derives the §5.2 cell as
`5761 + 52000 + 119600 + 28704 = 206065 B = 201.236 KB`, consistent with
the 201.2 KB rounded headline.

### N2. Chipmunk security range: 22.8 to 39.4 bits

| Location | Lines | Citation |
| --- | --- | --- |
| §1 TLDR | (not separately surfaced; "approximately 40-bit" footnote pointer) | quotes paper claim |
| §3 (no explicit number) | — | — |
| §5.5 | 416–420 | "[22.8, 39.4] bits"; secpar=128 sub-range [23.1, 38.8] |
| §9 table | 649 | "22.8–39.4 bits of core-SVP security" |
| Footnote 4 | 847–851 | min 22.8 (RSIS3 secpar=112 …); max 39.4 (RSIS1/RSIS2 secpar=112 τ=21 N=1024); secpar=128 [23.1, 38.8] |
| Top-level README | 71 | "22.8–39.4 bits of core-SVP security" |

All five surfaces agree. The §1.1 "approximately 40-bit" paraphrase (fn-35,
line 1023) is correctly framed as the paper's max-over-instances rounding,
not the review's measurement.

### N3. §6 thread-count ratios are arithmetically self-consistent

Hardware: 11 threads vs paper's 24. Naïve linear factor = 24/11 = 2.18×.
Recomputing each row's quoted ratio from the table's own absolute values:

| Operation | Baseline | Measured | Recomputed | Quoted | OK |
| --- | ---: | ---: | ---: | ---: | --- |
| Key generation | 1.3 min | 1.7 min | 1.308 | 1.3× | ✓ |
| Full sign | 1.3 min | 1.7 min | 1.308 | 1.3× | ✓ |
| Stateful sign | 4.1 ms | 3.91 ms | 0.954 | 0.95× | ✓ |
| Individual pre-verify | 1.67 s | 4.40 s | 2.635 | 2.6× | ✓ |
| Aggregate after verified inputs | 2.40 s | 5.74 s | 2.392 | 2.4× | ✓ |
| Secure aggregation | 567 ms | 1.41 s | 2.487 | 2.5× | ✓ |
| Batch verification N=2¹⁰ | 30.1 ms | 79.31 ms | 2.635 | 2.6× | ✓ |
| Batch verify N=2¹⁵ | 812 ms | 1.22 s | 1.503 | 1.5× | ✓ |

Every quoted ratio is the recomputed ratio rounded to one decimal place
(or to 1.3× / 2.6× when ties). Internal arithmetic is clean.

The summary ratios table (lines 498–501) is consistent in its own right:
`3.91 ms : 1.7 min = 3.91 : 102000 ≈ 1 : 26090` (review quotes 1:26 000,
matches); paper's `4.1 ms : 1.3 min = 4.1 : 78000 ≈ 1 : 19024` (review
quotes 1:19 000, matches). Sampler microbench: `3.8 / 1.7 = 2.235` versus
quoted "~2.2×" (matches; also matches fn-36's "Ratio 3.8/1.7 = 2.24×" at
line 1028).

### N4. `N(Q+1)²` reduction loss claim

Cited in three places:

| Location | Line | Phrasing |
| --- | --- | --- |
| §3 (Proves block) | 194 | "multi-user reduction loss of `N(Q+1)²` where `Q := Q_H + N`" |
| §3 ("Loose ends") | 222–224 | "`N(Q+1)² ≈ 2¹⁴⁰` at `N=2²⁰, Q_H=2⁶⁰`" |
| §10 | 722–727 | "`N(Q+1)² ≈ 2¹⁴⁰` at `N=2²⁰, Q_H=2⁶⁰`" |
| Footnote 17 | 917–921 | Theorem 4.1 / Lemma 4.1 sourcing with `N(Q+1)²` |

All three places give the same value (`~2¹⁴⁰`) under the same parameters
(`N=2²⁰, Q_H=2⁶⁰`). Sanity: `log2(N(Q+1)²) ≈ 20 + 2·60 = 140`. Consistent.

### N5. §5.6 Sage validation runtimes — plausibility

- `chipmunk_original.sage`: 2m45s — searches `2 × 4 × 3 = 24` cells of a
  Dilithium-style estimator. Per-cell ≈ 7 s. Plausible.
- `chipmunk_param.sage`: 13 s — same search space under the MSIS-optimised
  estimator (faster inner solve). Per-cell ≈ 0.5 s. Plausible.
- `lemur_param.sage`: ~25 min chunked, OOM on single end-to-end at 3 min
  on 8 GiB. 16 sub-runs covering `{τ ∈ {12,16,20,24}} × {N ∈ {1024, 32768,
  131072, 1048576}}`. Per-sub-run ~94 s avg. Plausible given the
  lattice-estimator heap behavior at large β.

The chipmunk `.txt` files were last modified at 17:23–17:24 (today),
consistent with the section's "re-run during this round" framing.
`summary.txt` was last modified at 17:59, also today, consistent with the
later chunked re-run.

The "byte-identical" / "field-identical row-by-row" distinction is
defensible: byte-identical only applies when the script's output is fully
deterministic (no Python dict-order variance, no timestamp embedding);
`lemur_param.sage` produces row sets that need to be re-stitched after
chunked runs, which is why the claim downgrades to field-identical
(values match per `(τ, N)` cell but the assembly order may differ).

---

## DRIFT FROM EARLIER ROUNDS

Three significant drift items were flagged by checkup_1's fact-checker and
reviewer subagents. All three are *resolved* in the current review.md:

1. **FC2-063/065/066 (and reviewer-promoted FC2-060/062) — §6 column
   header "Paper (24 thr)" mis-attributing README rows to the paper.**
   Resolved: §6 now renames the column to `Baseline (24 thr)` and adds an
   explicit `Source` column that says `Table 2` vs `README` per row
   (lines 483–494). The preamble at lines 473–481 calls this out
   explicitly: *"the baseline column below mixes two distinct 24-thread
   sources… Paper Table 2 reports only stateful sign, tree-in-memory sign,
   aggregation, batch verification, and the agg. signature size."*

2. **FC1-012 / FC1-013 — Aardal et al. and Anada et al. citation
   details.** Resolved: full author lists, venue, and ePrint/DOI now
   appear in footnotes 8 and 9 (lines 868–879).

3. **FC2-052 — Python↔Rust byte-equivalence claim was UNVERIFIABLE in the
   fact-check round.** Resolved: review.md §5.4 (lines 382–408) now shows
   the full 8-row MATCH table with the actual command. Footnote 33 also
   notes "Reproduced live during the implementation-lens audit's reviewer
   pass."

No claim that was FLAG'd or marked UNVERIFIABLE by checkup_1 appears in
review.md without having been addressed. Footnotes 8, 9, 33, 51 are
visibly the resolution surface.

The single broken back-arrow at `(#ref-4b)` (D1 above) is an *editorial*
drift residue from when fn-4 was cited four times rather than three —
not a content drift; the underlying numeric claim ([22.8, 39.4] / [23.1,
38.8]) is consistent everywhere.

---

## RECOMMENDATIONS

1. **Fix the orphan back-arrow at line 851.** Change
   ```
   specifically: [23.1, 38.8]. [↩](#ref-4) [↩](#ref-4b) [↩](#ref-4c) [↩](#ref-4d)
   ```
   to
   ```
   specifically: [23.1, 38.8]. [↩](#ref-4) [↩](#ref-4c) [↩](#ref-4d)
   ```
   This is the only structural defect found in the document.

2. **(optional, cosmetic) §5.4 command box.** Either change the displayed
   command from `diff <(jq -S . py.json) <(jq -S . rs.json)` to whatever
   `cli.py vectors` / `lemur vectors` literally invoke, or add a short
   parenthetical clarifying that the per-field `MATCH` lines come from
   the underlying Python equality, not from a `diff` of jq-sorted JSON.
   No anchors broken, no number wrong — just narrative tidying.

3. **(optional) Add a tiny self-check script.** A 10-line `awk`/Python
   block in `assessment/` that re-derives the anchor/footnote balance
   (`comm -23 …back-arrows… …ids…`) would catch the D1-class regression
   automatically the next time the footnote table is rearranged.

4. **No action needed elsewhere.** Numeric values agree across §3, §5.2,
   §5.3, §5.5, §6, §10, README, and footnotes; submission paths resolve;
   URLs are intact; TOC and headings balance; back-arrows on every
   heading are present; section/subsection numbering is gap-free; the
   §5.6 Sage validation runtimes are individually plausible and the
   on-disk artifact timestamps support the "re-run during this round"
   framing.

---

**Files cited in this audit (absolute paths):**

- `/workspace/repo/assessment/review.md`
- `/workspace/repo/submission/code/parameter/chipmunk_original.sage`
- `/workspace/repo/submission/code/parameter/chipmunk_original_security_summary.txt`
- `/workspace/repo/submission/code/parameter/chipmunk_param.sage`
- `/workspace/repo/submission/code/parameter/chipmunk_summary.txt`
- `/workspace/repo/submission/code/parameter/lemur_param.sage`
- `/workspace/repo/submission/code/parameter/summary.txt`
- `/workspace/repo/submission/report.pdf`
- `/workspace/tmp/checkup_1/review_fact_checker_2.md` (drift cross-check)
- `/workspace/tmp/checkup_1/review_fact_checker_1.md` (drift cross-check)
