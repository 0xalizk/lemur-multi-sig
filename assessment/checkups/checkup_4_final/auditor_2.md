# Auditor 2 — final-round cross-consistency audit

**Scope.** Cross-consistency between:

- `/workspace/repo/README.md` (top-level, 125 lines)
- `/workspace/repo/assessment/checkups/README.md` (14 lines)
- `/workspace/repo/assessment/review.md` (1142 lines) — verified only insofar
  as it is the target of hyperlinks from the other two.

Out of scope per instructions: `submission/`, the `checkups/` audit
transcripts under `checkup_0/`, `checkup_1/`, `checkup_2/`. The two ledgers
`open-issues.md` and `unresolved.md` are consulted because the checkups
README's "all addressed except §10" claim implicitly references them.

---

## VERIFIED cross-consistency

### V1. All 11 anchors in README "What the assessment found" resolve

Cross-checked each `assessment/review.md#sec-…` link in the root README
(lines 80-119) against the anchor inventory in `review.md`. Anchors live at
the lines listed below:

| README phrase | Link target | review.md anchor line | Section title | Content verdict |
| --- | --- | ---: | --- | --- |
| "sizes" | `#sec-5-3` | 365 | "Rice-encoded sizes for larger N" | Matches claim: 201.2 / 283.5 / 394.4 KB exact (line 373-375) |
| "byte-equivalent" | `#sec-5-4` | 385 | "Python ↔ Rust byte-equivalence" | Matches: 8/8 fields MATCH (line 393-404) |
| "tests" | `#sec-5-1` | 313 | "Rust test suite" | Matches: 52/52 tests (line 319, 332) |
| "checkpoints" | `#sec-4-4` | 296 | "Audit checkpoints — none of the foot-guns triggered" | Matches: four foot-guns checked in `kots.py`/`hvc.py`/`lemur.py` (line 302-308) |
| "estimator" | `#sec-5-6` | 431 | "Sage parameter-estimator outputs — reproduced from source" | Matches: byte-identical / field-identical (line 440-444) |
| "40-bit" | `#sec-5-5` | 413 | "Chipmunk security recomputation" | Matches: 22.8–39.4 bits core-SVP, max-over-instances reading (line 419-429) |
| "Timings" | `#sec-6` | 468 | "Benchmark measurements vs paper" | Matches: ±15 % of linear thread-count slowdown (table line 486-497) |
| "caveats" | `#sec-9-2` | 689 | "Notable framing softening worth flagging" | Matches first two sub-bullets (Rice-vs-raw, 4.64×/12.8×); the third sub-bullet (no runtime comparison) is anchored separately to `#sec-10`. Coherent. |
| "comparison" | `#sec-10` | 718 | "Open issues" | Matches: first §10 bullet is "No runtime comparison vs Chipmunk" (line 723-730) |
| "uncited" | `#sec-9-1` | 659 | "Missing references the paper *should* have engaged with" | Matches: covers both Aardal and Anada (line 664-681) |
| "loss" | `#sec-10` | 718 | "Open issues" | Matches: second-to-last §10 bullet is "Reduction loss not absorbed in parameters" (line 768-772) |

All 11 anchors exist and the section content directly substantiates the
README phrase that links to it.

### V2. Numerical figures in README "What the paper delivers" all trace to review.md

| README claim (line) | review.md citation | Match |
| --- | --- | --- |
| 201 KB at N=2¹⁰ (line 58) | 201.2 KB (lines 350, 373, 850) | ✓ rounds match |
| 284 KB at N=2¹⁵ (line 58) | 283.5 KB (lines 374, 850) | ✓ rounds match (284 ≈ 283.5) |
| 394 KB at N=2²⁰ (line 59) | 394.4 KB (lines 375, 850) | ✓ rounds match |
| 239 / 331 / 457 KB worst-case (line 60) | 239 / 331 / 457 KB (lines 373-375) | ✓ exact |
| Stateful sign 4.1 ms (line 61) | 4.1 ms paper / 4.13 ms README (line 491) | ✓ |
| Batch verification 30.1 ms at N=2¹⁰ (line 62) | 30.1 ms (line 495) | ✓ |
| 812 ms at N=2¹⁵ (line 62) | 812 ms (line 497) | ✓ |
| 25.3 s at N=2²⁰ (line 63) | **not quoted in review.md** — see DRIFT D1 | ⚠ |
| Secure aggregation 567 ms (line 63) | 567 ms (line 494) | ✓ |
| "~12 min at N=2²⁰" (line 64) | 12 min (line 743) | ✓ |
| 22.8–39.4 bits Chipmunk (line 71-72) | 22.8–39.4 bits (lines 419, 652, 707) | ✓ |
| 2.1× smaller (line 75) | 2.1× at N=2²⁰ (lines 99, 696, 858) | ✓ |
| 4.64× / 12.8× KOTS shrink (line 100-102) | 4.64× / 12.8× (lines 109, 692-693, 874) | ✓ |

12 of 13 numerical claims trace cleanly to a citation in review.md. The
"25.3 s at N=2²⁰" claim is verifiable from the paper PDF (Table 2) but
absent from review.md — see D1.

### V3. Checkups README hyperlink to §10 resolves correctly

`checkups/README.md:13` says
"`[except those listed in §10 \"Open issues\"](../review.md#sec-10) of review.md`".

- The relative path `../review.md` resolves from `assessment/checkups/` to
  `assessment/review.md`. ✓
- `sec-10` anchor exists at `review.md:718` and the section title is
  `10. Open issues`. ✓ Title matches the prose claim exactly.

### V4. Genealogy tree names all surface in review.md §9 / §9.1

Names in the root README tree (lines 30-51): BLS, Drake, LeanSig,
HAPPIER, Squirrel, Chipmunk, Anada, Aardal, Boneh-Kim.

- BLS [3]: §9 table row, line 649. ✓
- Drake [7]: §9 table row, line 650 (with LeanSig + HAPPIER inline as
  "2025+ successors"). ✓
- LeanSig: §9 line 650, §9.1 line 683. ✓
- HAPPIER: §9 line 650, §9.1 line 684. ✓
- Squirrel [9]: §9 table row, line 651. ✓
- Chipmunk [8]: §9 table row, line 652. ✓
- Anada: §2 lines 143/153-156, §9.1 line 670-681. ✓
- Aardal: §9.1 line 664-669. ✓
- Boneh-Kim [2]: §9 table row, line 655. ✓

All 9 names are present and discussed in review.md.

### V5. Genealogy trees in README and review.md §2 are structurally identical

Side-by-side compare of root README's tree (lines 30-51) vs review.md §2
tree (lines 129-150): same topology, same labels, same parenthetical
notes including "(standard model)" on the Anada node. No drift between
the two tree representations themselves.

### V6. Walltime table arithmetic plausible

Summing the rows in the `<details>` table (README lines 9-19):

- 15 (toolchain) + 2 (cargo) + ~0 (sizes) + ~0 (vectors) + 50 (bench)
  + 5 (bench_verify) + ~0 (chipmunk_param 13 s) + ~3 (chipmunk_original 2:45)
  + 25 (lemur_param) + 25 (audit subagents) + 30 (FC subagents)
  = ~155 min ≈ **2 h 35 min**.

Footer claims "~2 h 45 min" (line 20). The discrepancy is ~10 min,
consistent with rounding the sub-row times up (many `~` qualifiers) and
the "multiple runs across session" note on the `bench --fast` row. The
stated total is defensible but on the upper edge of the row sum.

### V7. Substantive §10 items match prior-round verdicts

The four "substantively unresolvable" items in §10 (lines 723-756):

1. **No runtime comparison vs Chipmunk.** Paper §1.1 quoted at line 727
   matches what the field-lens audit flagged (open-issues.md C1).
2. **RHF ≤ 1.0045 threshold itself is a choice.** Matches open-issues.md
   C3 verbatim in substance.
3. **Drake-vs-Lemur timing self-undercut.** Matches open-issues.md C6.
4. **N=2²⁰ end-to-end not measured locally.** Matches the §6.1 "What I
   could not run" entry (line 524) and the FC-round notes about the
   bench-measurement-not-committed entries (FC2-045, FC2-061, FC2-064,
   FC2-070 in unresolved.md).

These four are the items that the checkups/README claim explicitly carves
out as "still open."

---

## DRIFT items

### D1. README cites "25.3 s at N=2²⁰" but review.md does not quote that number

`README.md:63` lists "25.3 s at N=2²⁰" alongside the other paper
timings. Searched `review.md` for the literal "25.3" — no match (zero
hits). The number is real (paper Table 2, confirmed by `pdftotext` of
`submission/report.pdf` line 97), and review.md §6.1 explicitly says
this cell could not be measured locally (line 524: "N=2²⁰ batch verify
… needs ~9 GiB RAM; container has 8 GiB").

Consequence: the README quotes a paper number that review.md
deliberately *omits* (because it was unreproducible on the review host).
A reader following the "Timings" hyperlink from `#sec-6` will find the
table at lines 486-497, but no row for N=2²⁰ batch verify, and a 25.3-s
claim with no corroborating review entry.

Fix options (pick one):
- Add a Table-2-only row to review.md §6 or §6.1 quoting the 25.3 s with
  a "paper-only, unreproduced in this review" tag, or
- Drop "25.3 s at N=2²⁰" from the README delivery summary and note that
  the largest measured batch-verify cell was N=2¹⁵ at 812 ms.

### D2. README "strictly stronger security model" framing is now stale relative to review.md §9.1

`README.md:111-112` says of Anada:

> "a direct sibling in the synchronized lattice column with a strictly
> stronger security model"

This phrasing pre-dates the §9.1 refinement. `review.md:670-681` now says:

> "stronger on the proof-model axis (standard model, not ROM) but
> weaker on the threat-model axis (certified-key model, not
> rogue-key-safe) — the adversary cannot register arbitrary public keys,
> sidestepping rogue-key attacks rather than defending against them.
> Apples-to-apples comparison therefore requires either lifting Lemur to
> standard-model security or lifting Anada to a rogue-key-safe threat
> model; the two papers do not sit in the same cell of the (proof-model
> × threat-model) grid."

`review.md:153-158` (§2) reinforces: "*not a strict ordering*".

The README's "strictly stronger security model" is **the older framing**
the §9.1 update was explicitly correcting (the open-issues.md C2
trail). It is now misleading by review.md's own current standard.

Fix: change the README bullet to something like "a direct sibling in
the synchronized lattice column under the standard model (different
threat model)" — short enough for a delivery bullet, accurate enough
to not contradict §9.1.

### D3. README genealogy tree label "(standard model)" on Anada node — not strictly drift, but partially redundant with D2

The tree label `(standard model)` (`README.md:46`, `review.md:145`) is
literally accurate (Anada's proof model *is* standard model). But once
the §9.1 prose says the proof-model edge is offset by a weaker
threat-model edge, the bare label "(standard model)" leans toward the
"strictly stronger" reading.

This is a defensible minor compression for a tree label — the same
abbreviation appears in review.md §2's tree, so the two are internally
consistent — but it inherits whatever framing problem D2 carries.

Fix (optional): annotate the Anada leaf as `'24 ICISC / (std model,
certified-key)`. Trades 13 characters for full disclosure. Only worth
doing if D2 is fixed at the same time.

### D4. checkups/README "all issues addressed except §10" overstates coverage

`checkups/README.md:12-14`:

> "All issues that auditors and reviewers flagged have been addressed
> [except those listed in §10 \"Open issues\"](../review.md#sec-10) of
> review.md."

The §10 hyperlink resolves correctly (V3 above). The factual claim is
*partially* accurate, but is on the strong side. Cross-referencing the
ledger files:

`open-issues.md` carries 20+ items. Reconciling each against
review.md §10 / current review.md:

| open-issues.md item | Status |
| --- | --- |
| A1 (Thm 4.1 vs Lem 4.1 ℓ-restriction) | Tracked in §10's "Lemma 4.1 ℓ=1 restriction" bullet (line 774). ✓ addressed |
| A2 (Q = Q_H + N hidden N³) | Not in §10, not addressed. ⚠ |
| A3 (Constraint 9 chain) | Not in §10, not addressed. ⚠ |
| A4 (2⁻¹⁶² magic number) | Not in §10. Cosmetic per triage. ⚠ |
| A5 (m=20 origin) | Resolved per open-issues.md triage; review.md §3 distinguishes d=128 illustration. ✓ |
| B1 (BDS 134.4 KB rounding) | Not addressed; cosmetic. ⚠ |
| B2 (rayon scaling 1.69× provenance) | Not addressed. ⚠ |
| B3 (test-scope τ caveat) | Not addressed. ⚠ |
| B4 (567 ms reproduced) | Addressed: §6 line 494 reports 1.41 s on 11 threads. ✓ |
| C1 (no runtime comparison) | In §10. ✓ |
| C2 (Anada EUF-RK / model) | Addressed via §9.1 rewrite. ✓ |
| C3 (RHF threshold) | In §10. ✓ |
| C4 (Boudgoust-Takahashi) | Not addressed. ⚠ |
| C5 (Dual Hint-MLWE motivation outside Lemur) | Not addressed. ⚠ |
| C6 (Drake timing self-undercut) | In §10. ✓ |
| D1-D5 (skills usability) | Out of scope for review.md. — |
| E1, E2 (FC-round notes) | Resolved/dead-letter per the file's own triage. ✓ |

`unresolved.md` carries 8 still-genuinely-unverified items (FC2-001,
FC2-002, FC2-045, FC2-061, FC2-064, FC2-070, FC2-072, FC2-074). None
of these are in review.md §10. All are documented in `unresolved.md`
with "session-specific measurements, none block central claims" caveat.

So the checkups/README claim "addressed except §10" elides:
- ~6 minor open-issues.md items (A2, A3, A4, B1, B2, B3, C4, C5)
  that are neither in §10 nor independently resolved, and
- 8 unresolved.md atomic measurements that exist only in the
  unresolved.md ledger.

These are all explicitly labeled as non-load-bearing or cosmetic in
their respective triage sections, which is presumably why the checkups
README treats them as "not flagged" rather than "open." But the
*literal* claim "all issues addressed except §10" is stronger than the
truth.

Fix options:
- Tighten the claim to: "All **load-bearing** issues that auditors and
  reviewers flagged have been addressed except those listed in §10
  'Open issues' of review.md; non-load-bearing items are tracked in
  `open-issues.md` and `unresolved.md` for completeness."
- Or add a one-line pointer: "See also `open-issues.md` for
  non-load-bearing items deferred to a hypothetical third audit round,
  and `unresolved.md` for atomic measurements not re-run this session."

---

## MISSING / UNCLEAR

### M1. README's "788k tokens" figure is unverifiable from the three audit files alone

`README.md:3` reads `788k tokens 🦞 ~2h45m 🕒`. I have no access to a
context-usage telemetry stream from this session and cannot directly
verify the 788k figure. The walltime "~2h45m" is consistent with the
table sum (V6 above) to within 10 min, so plausible. The 788k token
count is plausible-magnitude for a session that ran 6+ audits, 6+
fact-checks, ~80 min of bench output, and ~25 min of Sage logs, but
without telemetry I can confirm only that the round number is not
obviously wrong.

### M2. README doesn't tell readers where to find the ledgers

The root README hyperlinks `assessment/review.md` extensively but never
points at `checkups/README.md`, `open-issues.md`, or `unresolved.md`.
Readers who want to know "what did this audit miss?" must find
`checkups/` themselves. Not strictly drift — by design `assessment/review.md`
is the public document — but cross-doc navigation is thin.

### M3. Tree label `(standard model)` does not match the new §2 prose nuance

Already covered in D3. Restated here because it's a *labelling*
question rather than a drift between the three files. The label is
internally consistent across both trees (README and §2); the question
is whether either tree should be updated. The §2 prose just below the
tree carries the disambiguation, so the tree's brevity is at most
mildly misleading. The README has no equivalent prose backstop.

---

## RECOMMENDATIONS

In severity order. The first two are substantive; the rest are
polishing.

### R1. Reconcile the "strictly stronger security model" framing (D2)

Edit `README.md:111-112`. Suggested replacement:

> "and Anada-Fukumitsu-Hasegawa *\"Tightly Secure Lattice-Based
> Synchronized Aggregate Signature in Standard Model\"* (ICISC 2024)
> — a synchronized-lattice sibling under the standard model but with a
> weaker (certified-key) threat model."

This is 4 words longer than the current bullet and removes the
contradiction with §9.1.

### R2. Resolve the "25.3 s at N=2²⁰" provenance (D1)

Two coherent options:
- Add to `review.md §6` a paper-only row reading roughly:
  "Batch verification, N=2²⁰ — 25.3 s (Paper Table 2, not reproduced
  locally; see §6.1)."
- Or rewrite `README.md:63` to read: "812 ms at N=2¹⁵ (largest cell
  measured locally), 25.3 s at N=2²⁰ per Table 2."

Either keeps the README and review.md telling the same story.

### R3. Tighten the checkups/README "all addressed except §10" claim (D4)

Replace `checkups/README.md:12-14` with something like:

> "All load-bearing issues raised in `checkup_0` and `checkup_1` have
> been folded into `../review.md`; the four substantively unresolvable
> items are listed in [§10 \"Open issues\"](../review.md#sec-10).
> Non-load-bearing items (theory micro-gaps, session-specific bench
> measurements, skills-usability notes) are tracked in `open-issues.md`
> and `unresolved.md` for completeness."

### R4. Optional: refine the Anada tree label (D3)

Only if R1 is accepted. Change `'24 ICISC / (standard model)` to
`'24 ICISC / (std model,\n                           certified-key)` in
both trees, or simply drop the parenthetical and let the §2 prose carry
the nuance.

### R5. No change needed for the genealogy tree, walltime, or the 11 in-bullet hyperlinks

V1, V4, V5, V6 verified clean. The README's tree, anchor links, and
walltime claim are all internally consistent with review.md and with
each other. R5 is "leave alone."

---

## Summary

- **Hyperlinks (11):** all resolve to existing anchors with content
  that substantiates the claim. ✓
- **Numerical claims (13):** 12 of 13 trace cleanly to a review.md
  citation; one ("25.3 s at N=2²⁰") is paper-true but missing from
  review.md (D1).
- **Anada framing:** README's "strictly stronger security model"
  contradicts the §9.1 update that explicitly says the ordering is not
  strict (D2). High-value fix; 4 words.
- **Checkups README claim:** literally overstates coverage; should
  acknowledge the two ledger files (D4). Medium-value fix.
- **Genealogy tree:** internally consistent across both files; all
  9 named works are discussed in review.md §9 / §9.1. ✓
- **Walltime:** row sum ≈ 2h35m; stated total ~2h45m; rounding within
  tolerance. ✓
- **§10 contents:** the four substantive unresolved items match the
  prior-round verdicts in open-issues.md and unresolved.md. ✓

Two drift items (D1, D2) are worth fixing before publication. D4 is a
polish item. The other findings are clean.
