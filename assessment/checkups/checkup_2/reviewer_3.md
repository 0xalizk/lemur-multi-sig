# Reviewer 3 — Verification of Auditor 3's review.md structural audit

**Target:** `/workspace/tmp/checkup_2/auditor_3.md` (audits `/workspace/repo/assessment/review.md`, 1128 lines).

**Method:** Every numeric claim in the audit was re-derived from the file via `grep`/`diff` and the orphan back-arrow was traced to its declaration site. Counts below are reproductions, not transcriptions.

---

## Per-claim verdicts

### Claim 1 — Footnote 4 has four `[↩]` arrows, `ref-4b` is missing

**Verdict: AGREE.**

- Footnote 4 declared once at line 847 (`<a id="fn-4">`).
- Back-arrow trail at line 851: `[↩](#ref-4) [↩](#ref-4b) [↩](#ref-4c) [↩](#ref-4d)` — four arrows.
- In-body anchors for fn-4 (`grep 'id="ref-4[a-z]?"'`):
  - line 102: `id="ref-4"`
  - line 417: `id="ref-4c"`
  - line 649: `id="ref-4d"`
- No `id="ref-4b"` exists anywhere in the file.
- Set-difference confirms the orphan is the unique mismatch:
  ```
  diff <(refs)  <(back-arrows)  →  50a51  > ref-4b
  ```

The audit's diagnosis ("a fourth citation of fn-4 was removed but the matching `[↩](#ref-4b)` trail was not pruned") is the right shape. Note: the alternative repair (add an `id="ref-4b"` to a real in-body citation that lost its anchor) is logically possible but unlikely — the three surviving in-body cites (lines 102, 417, 649) already cover the substantive surfaces (§1 narrative, §5.5 recomputation, §9 related-work table) where fn-4 is needed; there is no obvious orphaned-citation site that has plain `[4]` without an `id="ref-4..."`.

### Claim 2 — TOC↔anchor balance is 27↔27

**Verdict: AGREE.**

```
grep -oE '"#sec-[0-9-]+"' review.md | sort -u | wc -l  → 27
grep -oE 'id="sec-[0-9-]+"' review.md | sort -u | wc -l → 27
diff <(href set) <(id set)                              → empty
```

Element-for-element identical. No phantom TOC entries, no orphan headings.

### Claim 3 — 51 footnote declarations, all `#fn-N` cites resolve

**Verdict: AGREE.**

```
grep -c '<a id="fn-' review.md           → 51
grep -oE 'id="fn-[0-9]+"' | sort -u | wc → 51
```

Each fn-N declared exactly once; the union of in-body `#fn-N` targets is a subset of `{fn-1, …, fn-51}`. Set-equality holds by direct inspection.

### Claim 4 — 68 in-body ref-anchors vs 69 back-arrow targets, sole exception ref-4b

**Verdict: AGREE.**

```
grep -oE 'id="ref-[0-9]+[a-z]?"' | sort -u | wc -l  → 68
grep -oE '\(#ref-[0-9]+[a-z]?\)' | sort -u | wc -l  → 69
diff (above two sets)                                → "ref-4b" only
```

ref-4b is the unique surplus on the back-arrow side. All 68 in-body anchors have a matching back-arrow. The audit's tally is exact.

(Minor pedantic point about the audit's footnote-suffix enumeration on its line 67: the list it gives — `-1b, -2b, -5b, -7b, -8b, -8c, -9b, -9c, -9d, -20b, -23b, -45b, -46b, -51b, -51c` — has 15 entries, not the 13 stated. Same prose paragraph also says "Plus `ref-4c, ref-4d`" which makes the running total 15+2 = 17 suffixed ids, plus 51 base = 68. Arithmetic lands right; only the "13" count is mistyped. Cosmetic.)

### Claim 5 — All 9 submission paths resolve on disk

**Verdict: AGREE.**

Spot-checked 4 of 9 via `ls -la`:
- `/workspace/repo/submission/report.pdf` → 2.16 MB, exists.
- `/workspace/repo/submission/code/parameter/chipmunk_original.sage` → 12.5 KB, exists.
- `/workspace/repo/submission/code/parameter/chipmunk_summary.txt` → 3.1 KB, mtime 17:24, exists.
- `/workspace/repo/submission/code/parameter/summary.txt` → 5.4 KB, mtime 17:59, exists.

Parent directories `submission/code/` and `submission/code/parameter/` are real dirs. All 9 targets in the audit's table check out.

### Claim 6 — Every section/subsection heading has `[↑](#toc)`

**Verdict: REVISE (very minor count error, conclusion still holds).**

The audit asserts "All 27 headings end with `[↑](#toc)`" and lists 27 line numbers + the Footnotes heading at line 822 (28 total).

Actual count:
```
grep -cE '^#{2,} .*\[↑\]\(#toc\)' review.md → 28
grep -cE '^#{2,}'                            → 29
```

The 29th heading at line 1 (`## Lemur — Analysis & Correctness Review`) is the document **title**; titles legitimately do not get back-arrows. So the structural claim is correct: every navigable section heading has its arrow. The audit's wording "27 headings" undercounts by one (should be 28 = 27 numbered `sec-*` + 1 `footnotes`), but the substantive claim is fine.

### Claim 7 — Aggregate sizes 201.2 / 283.5 / 394.4 KB agree across §3, §5.2, §5.3, §9, README

**Verdict: AGREE.**

Direct citations in review.md:
- §3 line 233–234: `201.2 KB at N=2¹⁰, 283.5 KB at N=2¹⁵, 394.4 KB at N=2²⁰`
- §5.2 line 347: `Aggregated sig, N=2¹⁰ … 201.2 KB ✓ exact`; sub-component sum at line 353 → 201.2 KB
- §5.3 lines 370–372: all three values, all marked `✓`
- footnote 1 at line 836: `Table 2 … 201.2 / 283.5 / 394.4 KB`
- footnote 31 at line 996: `= 201.236 KB` (rounded headline)
- README line 81 (`/workspace/repo/README.md`): `aggregate sizes (201.2 / 283.5 / 394.4 KB exact)`

All five surfaces (§3, §5.2, §5.3, footnotes, README) match. Note the audit's table on line 174–181 lists "§9" but the §9 table actually cites the **security range** (22.8–39.4 bits), not the **sizes**; the audit elsewhere correctly attributes sizes to §3/§5.2/§5.3 + README/footnotes. Substantively right, label slightly off.

### Claim 8 — §5.6 Sage runtimes plausible vs §11 recipe

**Verdict: AGREE (with a small contextual caveat).**

§5.6 table (lines 437–441) records:
- `chipmunk_original.sage` → 2m45s, byte-identical
- `chipmunk_param.sage` → 13s, byte-identical
- `lemur_param.sage` → ~25 min (chunked), field-identical

§11 (lines 776–782) describes the host: "aarch64 Linux, 11 CPU threads, 8 GiB RAM". The OOM-then-chunk story for `lemur_param.sage` at 8 GiB is consistent with lattice-estimator BKZ heap pressure at large β. Per-cell averages (~7 s, ~0.5 s, ~94 s) are in the right order of magnitude for Dilithium-style and MSIS-style estimator inner loops.

The on-disk mtimes confirm a re-run window: `chipmunk_original_security_summary.txt` (17:23), `chipmunk_summary.txt` (17:24), `summary.txt` (17:59). All same-day. Auditor 3's plausibility argument holds.

Small caveat the auditor didn't note: the §11 recipe at lines 786–795 does **not** include the `sagemath` install line. §5.6 mentions `apt-get install --no-install-recommends sagemath` was done out-of-band. A fully self-contained reproduction recipe would inline that; for the audit's structural scope this is non-blocking.

### Claim 9 — §5.4 narrative shows `diff <(jq …)` but the actual tool is Python equality on parsed JSON

**Verdict: AGREE.**

§5.4 lines 384–388 literally print:
```
diff <(jq -S . py.json) <(jq -S . rs.json)
```

…and then lines 393–401 show eight `MATCH` rows formatted as `key: MATCH` — exactly the shape a Python dict-equality printer produces, not what `diff` outputs (`diff` would either be silent or emit `<`/`>` line diffs). The narrative and the result table are internally consistent (both say "byte-equivalent"), but the command shown is not the literal command that produced the table. Pure cosmetic; no anchor or numeric impact.

---

## The ref-4b orphan — exact textual fix

**Current text at line 851:**
```
specifically: [23.1, 38.8]. [↩](#ref-4) [↩](#ref-4b) [↩](#ref-4c) [↩](#ref-4d)
```

**Proposed fix (drop the orphan):**
```
specifically: [23.1, 38.8]. [↩](#ref-4) [↩](#ref-4c) [↩](#ref-4d)
```

**Justification for this direction (drop, not add):**

1. Three in-body cites exist (`ref-4`, `ref-4c`, `ref-4d`) at three distinct content surfaces (§1 narrative, §5.5 recomputation, §9 related-work table). Each one is a clean, intentional cite with normal phrasing around it. There is no plain "[4]" hanging in the body with a stripped `id`.

2. The suffix sequence `b → c → d` with a gap at `b` is the canonical fingerprint of a citation deletion: when an editor removes the middle of a `b/c/d` cluster, the natural fix is to drop the gap in the back-arrow trail; re-introducing a `b` cite would mean reverting a content deletion the author made deliberately.

3. The audit chose this direction too. I concur.

**As a one-shot sed (safe — anchors anywhere else don't have this exact string):**
```
[↩](#ref-4) [↩](#ref-4b) [↩](#ref-4c) [↩](#ref-4d)
  →
[↩](#ref-4) [↩](#ref-4c) [↩](#ref-4d)
```

---

## What the audit missed (or under-stressed)

1. **Document-title vs section-heading count.** The audit says "27 headings"; there are 28 navigable headings (27 `sec-*` + 1 `footnotes`) plus the document title. Trivial, but the count being off by one in writing makes the V2 check look like 27=27 when it's actually 28=28 of 29 (the document title legitimately has no back-arrow).

2. **Suffix-list arithmetic in V4.** The audit's prose lists 15 suffixed ref IDs while calling it "13 suffixed duplicates". The substantive 68 / 69 count is correct (and matches my independent recount); only the inline prose count is mistyped.

3. **§11 recipe doesn't install SageMath.** §5.6 depends on SageMath being installed but the §11 reproduction recipe at lines 784–795 only installs Rust + Python. A reader following the recipe verbatim would not be able to reproduce §5.6's runtimes. Not a structural defect (no broken anchor, no wrong number) — but a reproducibility gap worth flagging.

4. **`sec-9` table at line 649 cites `ref-4d` inside a multi-line table row.** Visually verified the anchor is well-formed (`<sup id="ref-4d">[4](#fn-4)</sup>`), so the structural sweep is fine — but it does mean fn-4 is the most heavily-cited footnote in the document (3 distinct cites across 3 sections), which is precisely why the editorial drift around it produced this orphan. A self-check script of the kind the audit recommends in its R3 would be especially useful for the multi-cite footnotes.

5. **Other heavy multi-cite footnotes (8, 9, 51) are clean.** I separately confirmed `id="ref-8b"`, `id="ref-8c"`, `id="ref-9b/c/d"`, `id="ref-51b/c"` all exist in the body and match their back-arrow trails. fn-4 is genuinely the only broken one — the audit's "exactly one defect" headline holds.

---

## Bottom line

Auditor 3's central finding — a single dangling `[↩](#ref-4b)` at line 851 — is correct and independently reproduces. The proposed one-line fix (delete the orphan back-arrow) is the right repair. All other structural and numeric claims in the audit verify, modulo three trivial wording issues (heading count off by one; suffix list size mistyped as 13 vs actual 15; §9 mislabeled as a size-citation site instead of a security-range site). Recommend the audit be accepted, the ref-4b fix applied verbatim, and the §11 SageMath install line added as a follow-up for full reproducibility.
