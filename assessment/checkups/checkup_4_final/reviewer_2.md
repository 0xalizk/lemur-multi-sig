# Reviewer 2 — verification of Auditor 2's findings

Scope: independently spot-check each verified item, drift item, and
recommendation in `auditor_2.md`. Verdicts are AGREE / REVISE / DISPUTE.

---

## Claim-by-claim verdicts

### V1 — all 11 anchors in README "What the assessment found" resolve

**AGREE.** Independently confirmed via
`grep -nE 'id="sec-(5-3|5-4|5-1|4-4|5-6|5-5|6|9-2|9-1|10)"' review.md`:
all anchors exist (`review.md:296, 313, 365, 385, 413, 431, 468, 659,
689, 718`). Spot-checked three:

- `#sec-5-3` (review.md:365 "Rice-encoded sizes for larger N") — body
  at 373-375 reports exactly the 201.2 / 283.5 / 394.4 KB the README
  cites. ✓
- `#sec-5-5` (review.md:413 "Chipmunk security recomputation") — body
  at 419-429 carries the "22.8–39.4 bits / max-over-instances" reading
  the README leans on. ✓
- `#sec-9-1` (review.md:659 "Missing references") — body at 664-681
  covers Aardal and Anada exactly as the README bullet 5 advertises. ✓

The duplicate `#sec-10` (used for both "comparison" and "loss")
resolves twice to the same anchor at `review.md:718`. First content
match is the "No runtime comparison" bullet (723-730); second is the
"Reduction loss" bullet (768-772). Both present. ✓

### V2 / D1 — "25.3 s at N=2²⁰" is not in review.md

**AGREE.** `grep "25.3"` returns zero hits in `review.md` and exactly
one in `README.md:63`. The number is **paper Table 2** (PDF
`pdftotext -layout` line 97: `Batch verification  30.1 ms  812 ms§
25.3 s§`). The `§` mark in the paper denotes linear extrapolation
from smaller cells, not a fresh measurement — so the paper itself
labels this cell as extrapolated, which makes the omission from
review.md a defensible choice (§6.1 line 524 explains the local
machine cannot run N=2²⁰ batch verify at 9 GiB RAM).

Auditor 2's recommendation R2 (add paper-only row to §6, or rewrite
README line 63 to flag "per Table 2") is correct. The provenance is
crystal-clear: Table 2 of the paper, extrapolated cell.

### V3 — checkups/README → review.md#sec-10 link

**AGREE.** `checkups/README.md:13` relative path resolves; anchor at
`review.md:718` matches the cited title "Open issues". Verified.

### V4 — genealogy tree names all surface in review.md §9 / §9.1

**AGREE.** Independent grep confirmed all 9: BLS, Drake, LeanSig,
HAPPIER, Squirrel, Chipmunk, Anada, Aardal, Boneh-Kim all appear in
review.md. Spot-checked §9.1 (review.md:659-687): Aardal at 664-669,
Anada at 670-681, LeanSig+HAPPIER at 683-684. Tree at review.md:129-150
carries BLS, Drake, LeanSig, HAPPIER, Squirrel, Chipmunk, Anada,
Boneh-Kim, Aardal. ✓

### V5 — README tree vs review.md §2 tree structurally identical

**AGREE.** README tree at lines 30-51, review.md §2 tree at 129-150.
Diffing label by label: identical topology, identical labels
including `(standard model)` on Anada at README:46 / review.md:145.
No drift between the two tree renderings.

### V6 — walltime arithmetic

**AGREE with refinement.** Sum confirmed:
`15 + 2 + 0 + 0 + 50 + 5 + 0 + 3 + 25 + 25 + 30 = 155 min = 2h 35m`.
Footer says "~2 h 45 min" (README:20). Gap is 10 min, within rounding
tolerance given the table uses `~` qualifiers on most rows and notes
"multiple runs across session" on the bench row. The `~` in the
footer total absorbs this. AGREE the claim is "defensible but on the
upper edge."

### V7 — §10 items match prior-round verdicts

**AGREE.** Confirmed via cross-read of `open-issues.md`:
- C1 (no runtime comparison) ↔ §10 bullet 1 (review.md:723-730). ✓
- C3 (RHF threshold choice) ↔ §10 bullet 2 (review.md:732-740). ✓
- C6 (Drake-vs-Lemur self-undercut) ↔ §10 bullet 3 (review.md:742-748). ✓
- §10 also lists "N=2²⁰ end-to-end not measured locally", which maps
  to `unresolved.md`'s FC2-045/061/064/070 batch of "fresh-bench"
  unresolved items.

### D1 — 25.3 s drift

**AGREE.** Already verified under V2 above. Paper-true,
review.md-absent. R2 fix is correct.

### D2 — README "strictly stronger security model" vs review.md §9.1

**AGREE — high-value fix.** Direct contradiction confirmed:

- `README.md:111-112`: "a direct sibling in the synchronized lattice
  column with a **strictly stronger** security model"
- `review.md:673-675`: "**stronger on the proof-model axis** (standard
  model, not ROM) but **weaker on the threat-model axis** (certified-key
  model, not rogue-key-safe)"
- `review.md:155-156`: also explicit — "stronger proof-model than Lemur's
  ROM" but uses certified-key threat model, "weaker than Lemur's
  rogue-key-safe".

The README claim is exactly the framing §9.1 was rewritten to refute.
"Strictly stronger" is false by review.md's own current standard.

### D3 — `(standard model)` tree label

**AGREE-PARTIAL.** Tree label is literally correct (Anada's proof
model is standard model) and identical in both trees. It's a
4-character compression that inherits D2's framing problem only via
selective reading. Standalone, it's accurate. R4 (optional refinement)
is reasonable but low-priority — and would require changing both trees
to stay in sync.

### D4 — "all issues addressed except §10" overstates coverage

**AGREE.** Independently audited `open-issues.md` and `unresolved.md`:

`open-issues.md` ledger (non-load-bearing per its own triage at
line 105-128):
- A2, A3, A4 (theory micro-gaps) — not in §10. Triage labels them
  "third audit round" material.
- B1, B2, B3 (rounding nit, rayon provenance, test scope) — not in §10.
- C4 (Boudgoust-Takahashi follow-up) — not in §10. Triage: "third
  round."
- C5 (Dual Hint-MLWE outside Lemur) — not in §10.

That's 8 items, matching Auditor 2's count.

`unresolved.md` lists 8 still-unverified atomic items
(FC2-001, FC2-002, FC2-045, FC2-061, FC2-064, FC2-070, FC2-072,
FC2-074) — all session-specific measurements. Confirmed:
"None block review.md's central claims" (unresolved.md:102).

So the literal "all issues addressed except §10" elides 8 open-issues +
8 unresolved = 16 items, *all triaged as non-load-bearing*. The claim
is technically overstated but the underlying engineering judgment
("the only load-bearing open items are in §10") holds. R3 fix is
appropriate: tighten to "load-bearing" and point at the ledger files.

### V "Genealogy" / numerical claims (V2 table)

**AGREE on 12 of 13.** Independently spot-confirmed:
- 201/284/394 KB ↔ review.md:373-375 ✓
- 4.1 ms stateful sign ↔ review.md:491 ✓
- 30.1 ms / 812 ms ↔ review.md:495, 497 ✓
- 567 ms ↔ review.md:494 ✓
- 12 min at N=2²⁰ ↔ review.md:743 ✓
- 22.8–39.4 bits ↔ review.md:419 ✓
- 2.1× ↔ review.md:99, 696 ✓
- 4.64× / 12.8× ↔ review.md:692-693 ✓

The 13th (25.3 s) is the D1 item.

---

## Things Auditor 2 missed (or under-emphasised)

### M1. The paper's `§` mark on the 25.3 s cell

Paper Table 2 (PDF line 97) marks 812 ms and 25.3 s with `§`. The
paper's own table footnote convention treats `§` cells as
**linear-extrapolated**, not measured. Auditor 2 calls 25.3 s
"paper-true" but doesn't surface that the paper itself flags it as
non-measured. This sharpens R2: the correct README phrasing isn't
just "per Table 2" — it's "per Table 2 (paper's linear extrapolation,
not measured)."

### M2. README's bullet 5 also lists Anada with a stale paradigm framing

`README.md:109-112` calls Anada "a direct sibling in the synchronized
lattice column with a strictly stronger security model." The
"sibling" framing is *also* refined in review.md §2 (line 153: "Lemur
and Anada et al. are siblings, **not ancestor/descendant**") which
implicitly concedes they don't sit in the same cell. The audit
focused on "strictly stronger" but the whole bullet 5 framing of
Anada needs touching — not just the trailing four words.

### M3. The footnote-9 disambiguation in review.md is itself a tell

`review.md:893`: footnote 9 ("standard model (not ROM)") only mentions
the proof-model edge. Read alone, footnote 9 reads like the older
"strictly stronger" framing the §9.1 rewrite was meant to retire.
Internal inconsistency between footnote 9 and §9.1 body. Audit didn't
flag this.

### M4. README's "PQ-sig + lattice-PoK" paradigm assertion in bullet 5

Bullet 5 in README (line 107: "Aardal et al. ... a PQ-sig +
lattice-PoK paradigm Lemur's introduction trichotomy does not
contain") accurately matches review.md:668 ("PQ-sig + lattice-PoK").
Not drift — but worth confirming, which audit didn't.

---

## Proposed exact text for high-value fixes

### R1 (Anada framing) — exact replacement for `README.md:109-112`

The audit's 4-word fix is too compressed (it just swaps "strictly
stronger security model" for "(certified-key threat model)" without
also addressing the "direct sibling" framing). Suggested **full
bullet**:

```markdown
   and Anada-Fukumitsu-Hasegawa *"Tightly Secure
   Lattice-Based Synchronized Aggregate Signature in Standard Model"*
   (ICISC 2024) — synchronized-lattice and standard-model (stronger
   proof model than Lemur's ROM), but adversary cannot register
   arbitrary keys (certified-key, weaker threat model than Lemur's
   rogue-key-safe). The two papers do not sit in the same
   (proof-model × threat-model) cell.
```

Net change: +27 words, removes "strictly stronger" and "direct
sibling" — both of which contradict review.md §9.1's refined position.

If a shorter delivery bullet is preferred:

```markdown
   and Anada-Fukumitsu-Hasegawa (ICISC 2024) — standard model, but
   under a weaker (certified-key) threat model than Lemur's
   rogue-key-safe setting.
```

This is the minimum surgical fix and is internally consistent with
both review.md §2 (line 153, "siblings, not ancestor/descendant") and
review.md §9.1 (line 673-675).

### R2 (25.3 s provenance) — exact replacement for `README.md:62-63`

```markdown
3. **Timings (24-thread baseline):** stateful sign 4.1 ms, batch
   verification 30.1 ms at `N = 2¹⁰`, 812 ms at `N = 2¹⁵`,
   25.3 s at `N = 2²⁰` (Table 2 linear extrapolation,
   not directly measured); secure aggregation 567 ms at `N = 2¹⁰`,
   linear-extrapolated to ≈ 12 min at `N = 2²⁰`.
```

Net change: +9 words. Truthful and explicitly flags the
paper-side extrapolation `§` mark.

Alternative: add a paper-only row to `review.md §6` table:

```markdown
| Batch verify, N=2²⁰ (paper extrap.) | 25.3 s | Table 2 (§) | not measured (RAM-bound, see §6.1) | — | paper-side extrapolation |
```

### R3 (checkups README) — Auditor 2's R3 phrasing is acceptable as-is

Recommend adopting Auditor 2's R3 verbatim. Add at end: "See also
[`open-issues.md`](open-issues.md) and [`unresolved.md`](unresolved.md)."

### R4 (Anada tree label)

Conditional on R1 being applied: leave the tree label as
`(standard model)` since both trees carry the same compression and
the surrounding prose now carries the disambiguation. Modifying the
tree adds churn without proportional clarity.

---

## Summary

Of Auditor 2's findings, **all V1-V7 verifications hold**, **D1 / D2
/ D4 are real and fixable**, **D3 is minor / conditional**.

Two items Auditor 2 under-emphasised:
1. The 25.3 s cell is paper-extrapolated, not paper-measured — R2's
   fix language should say so.
2. The Anada bullet has *two* defects, not one — "strictly stronger"
   plus "direct sibling" both contradict review.md §9.1. R1 needs to
   be ~27 words, not 4.

Priority for publication:
1. **R1 (Anada framing)** — high-value, exact text proposed above.
2. **R2 (25.3 s)** — medium-value, exact text proposed above with
   `§` extrapolation tag.
3. **R3 (checkups/README tightening)** — low-value polish.
4. **R4 (tree label)** — skip.

Files cited:
- `/workspace/repo/README.md` (lines 30-51, 63, 109-112)
- `/workspace/repo/assessment/review.md` (lines 129-150, 153-156,
  468-525, 659-687, 718-748, 893)
- `/workspace/repo/assessment/checkups/README.md` (lines 12-14)
- `/workspace/repo/assessment/checkups/open-issues.md`
- `/workspace/repo/assessment/checkups/unresolved.md`
- `/workspace/repo/submission/report.pdf` (Table 2, PDF line 97)
