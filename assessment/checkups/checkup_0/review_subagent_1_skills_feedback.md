# Review of subagent_1_skills_feedback.md (skills critique)

Source materials consulted: `lemur-paper-analysis/SKILL.md` (241 lines),
`lemur-prior-work-survey/SKILL.md` (281 lines), and the paper-feedback
file from the same subagent.

## Per-edit verdict

### Edit 1 — Add lemma-dependency diagram in Step 1, paper-analysis

**Verdict: SENSIBLE, with caveat.** The proposed diagram correctly traces
Theorem 4.1 ← Lemma 4.1 ← {Lemma 4.2, Lemma 4.3, Theorem 3.1} ← {Lemma
3.2, 3.3, 3.4, 3.5} ← Lemma 2.6 ← Appendix D. I verified this chain
against the paper: Theorem 4.1's proof "via Lemmas D.1, 4.1, D.2 and
D.3" (line 610); Lemma 4.1 invokes Lemma 4.3 (line 685) and Lemma 4.2
(line 719); Theorem 3.1 invokes Lemmas 3.2, 3.3, 3.4 (lines 480-483)
and Lemma 3.5 (line 472); Lemma 3.5 invokes Lemma 2.6 (line 534).

**Caveat:** the proposed diagram omits Lemma 3.1 (game G0 ≡ G1, line
454) which is also a load-bearing step inside Theorem 3.1. Minor. The
diagram as proposed is correct enough to be useful.

**Caveat 2:** the diagram puts Lemma D.1/D.2/D.3 at the same level as
Lemma 4.1 under Theorem 4.1. That's correct per line 610. Good.

**Level of abstraction:** appropriate. The existing Step 1 already has
a `§1.1 → §1.2 → §3 → §4` flow diagram; adding a proof-internal
dependency map under the "Audit the proof" branch fits the existing
abstraction level.

### Edit 2 — Expand Lemma 3.5 sentence in Step 1

**Verdict: SENSIBLE.** The current skill says "Lemma 3.5 (sampleability
over non-full-rank lattices) is the technically novel proof step" — true
but uninformative. The proposed expansion ("standard samplers handle
only full-rank lattices but `ker(φ_H)` is rank-`(k-ℓ)d` in `Z^{kd}`")
is correct (Lemma 2.5 at line 270: "There is a probabilistic polynomial
time algorithm that, given a basis B = (b1, . . . , bn) of a full-rank
n-dimensional lattice Λ..." — full-rank requirement explicit; Lemma 2.6
at line 286 explicitly titled "Reduction to full-rank sampler").

**Note:** the proposed expansion concludes with "8mε_2-tight reduction"
which is correct (Theorem 3.1, line 434). Good integration.

### Edit 3 — Delete KOTS+HVC table in paper-analysis Step 2, cross-reference prior-work-survey

**Verdict: MOSTLY SENSIBLE but I disagree on direction.** The two
skills do indeed duplicate the KOTS+HVC blueprint (paper-analysis
lines 67-77 vs prior-work-survey lines 136-153). Deduplication is
correct. However:

- The paper-analysis version is shorter and explicitly framed at the
  "how to read a single paper" level ("Lemur's surface area: KOTS and
  HVC").
- The prior-work-survey version is the "field map" framing.

These are slightly different audiences and **the two tables aren't
fully redundant** — paper-analysis's right column ("Where Lemur
differs") is paper-specific, while prior-work-survey's right column
("Where the smarts go") is generic. So a clean dedup would keep both
*but* refactor the paper-analysis one to just point at the lineage
table and add Lemur-specific deltas.

Audit's recommendation to delete entirely with a one-line pointer is
slightly too aggressive. Better: shrink to a 3-line Lemur-deltas
block. Either way the duplication concern is real.

### Edit 4 — Add "Reduction tightness" as 4th category in Step 3

**Verdict: SENSIBLE and overdue.** Step 3 covers size/timing/security
but does not call out *reduction tightness* — a real gap for theory
work. The proposed addition (multiplying through `N · Q^k` factors,
demanding `λ + log_2(N Q^2)` underlying hardness) is correct
asymptotically. The example numbers (`N=2^{20}, Q=2^{60}` giving 140
bits of loss) are accurate.

**Caveat:** the proposed text says "Lemur loses `N(Q+1)^2`" — this
needs the `ℓ=1` and `k ≥ 4ℓ` context to be precise; otherwise it reads
as a universal claim about Lemur's bound. A more careful phrasing
would help.

### Edit 5 — Explain `RSIS rop` columns in Step 3 / footer

**Verdict: SENSIBLE.** I verified that
`chipmunk_original_security_summary.txt` does have RSIS1/2/3 rop
columns and the audit's `[22.8, 39.4]` range is correct (min 22.8 on
the 112-secpar ρ=131072 rows, max 39.4 on 112-secpar τ=21 ρ=1024).
For 128-secpar, the audit's `[23.1, 38.8]` is also reproducible.

The proposed footnote that the columns are "base-2 log of estimated
SVP-attack operation cost — i.e. bit-security in core-SVP" is a
correct semantic gloss and helpful to a theory reader. **One refinement:**
this should explicitly note that these columns come from
`SIS.estimate.rough` (lattice-estimator function) — that lets a
reviewer trace back to the actual cost model.

### Edit 6 — Add new "Step 8: When auditing a security proof"

**Verdict: SENSIBLE, but might bloat the skill.** The 8-bullet
checklist is correct in content (main theorem → lemma graph → load-
bearing novel step → parameter regime restrictions → reduction loss
arithmetic → statistical slack sum → assumption sanity-check). All of
these are real practices.

**Concern:** the current skill is already 241 lines; adding an 8-bullet
audit checklist plus all the other proposed edits would push it
toward 350-400 lines. The audit itself notes (§E) that some current
content (Step 5, Step 7) is not theory-relevant. If you're going to
add a theory-audit branch, the right move is to factor out the
summarizer/comparator content into a separate file — not pile on.

**Suggestion:** split into `lemur-paper-analysis-summarizer` and
`lemur-paper-analysis-theory-audit` rather than a single fat skill.
The audit hints at this in §E but does not commit; if I were the
maintainer, I'd commit.

### Edit 7 — Expand [11] Hint-MLWE bullet in prior-work-survey Step 5

**Verdict: SENSIBLE.** Current Step 5 gives a 3-line summary of
Hint-MLWE; the proposed expansion (telling a theory reader what
specifically to extract from a predecessor-assumption paper) is
useful. The "(a) formal definition; (b) reduction loss factor;
(c) parameter regime" tripartite is correct prescription.

### Edit 8 — Add Hint-MLWE vs Dual Hint-MLWE comparison table in Step 5

**Verdict: SENSIBLE.** The proposed table fields (hint form, secret
placement, hint count, reduction loss, smoothing precondition) are
all real comparison axes. The "Dual" reduction loss being `8mε_2`
statistical with no multiplicative term is correct (Theorem 3.1 at
line 434, as verified in paper-audit review). The Hint-MLWE column
("loss scales with hint width and Q") is structurally right but
the audit does not actually open Kim et al. to verify the exact
loss form; it could be more careful.

### Edit 9 — Consolidate red-flag content (Step 4/Step 7 in prior-work-survey)

**Verdict: MILDLY SENSIBLE.** Step 4 (building blocks) and Step 7
(red flags) do not actually overlap that much when I read the SKILL.
Step 4 is about *which block does the new paper touch*; Step 7 is
about *whether the numerical comparison is apples-to-apples*. The
audit overstates the overlap. Both are useful in their current form;
consolidation would lose clarity.

**Recommendation:** Reject this edit, or apply it only by adding
cross-references rather than merging content.

### Edit 10 — Add new "Step 9: When the theoretical lever is novel" in prior-work-survey

**Verdict: SENSIBLE.** The 4-bullet credibility test (reduction to
standard? predecessor differences? parameter regime match? subtler
reduction loss?) is a useful pattern. It does not duplicate existing
content. The Lemur-specific examples in each bullet are accurate per
the paper-audit verifications.

### Edit 11 — Both skills: explain `RSIS rop` columns at file-list footer

**Verdict: SENSIBLE, low cost.** Already discussed under Edit 5. Yes,
add a one-liner in both `Files this skill uses` sections.

---

## Things the audit got right about skills usability

1. The observation that ~60% of `lemur-paper-analysis` is summarizer-
   oriented (size/timing/security claims, summarizing-for-others) and
   only Step 1/Step 3 fragments speak to proof audits is **accurate**.
   This is a genuine usability gap for theory-lens users.
2. The KOTS+HVC blueprint duplication between the two skills is real.
3. The lemma-dependency diagram as a quick-orientation aid is the
   single most useful proposed addition, as the audit itself
   identifies in §F.
4. The `chipmunk_original_security_summary.txt` columns are not
   currently explained; the gloss is a no-cost improvement.

## Things the audit missed about skills usability

1. **Neither skill references the `lemur.txt` line-number layout** used
   in the audit itself. If the workflow is "grep `lemur.txt` for line
   numbers, cite them in audits", that workflow should be documented.
   Otherwise a future theory-lens user reaches for `pdftotext` from
   scratch.
2. **No mention of `Tables 4 and 5`** which encode the constraints
   (and hence the proof-side restrictions). Theorem 4.1's preconditions
   correspond to numbered constraints; the skill should point at the
   constraint table.
3. **The skills don't say anything about the `parameter/lemur_param.sage`
   script** which is the source of truth for the parameter computations.
   For a theory user checking that the picked parameters satisfy the
   theorem preconditions, this script is the load-bearing artifact.
4. **No explicit advice on when to trust vs verify the paper's
   numerical inequalities** (eq. 4, 5 in §3; α=61 derivation in §7).
   A theory user typically wants to re-derive these.
5. **The skills don't warn that `Q := Q_H + N`** (Lemma 4.1, line 647) —
   a subtle definition that affects the reduction loss interpretation.
   Small but exactly the kind of thing a proof skill should call out.

## Things the audit proposed that would make the skills worse

1. **Consolidating Step 4 and Step 7 of prior-work-survey** (audit
   Edit 9). They serve different purposes; merging would dilute both.
2. **Aggressively deleting the paper-analysis Step 2 KOTS+HVC table**
   (audit Edit 3) without replacing with Lemur-specific deltas. A
   single-paper reader benefits from "Lemur's surface area is KOTS+HVC"
   even if the full lineage table lives elsewhere.
3. **Piling additions onto an already-long skill** (audit Step 8 + the
   diagram + 4 other expansions in paper-analysis) without first
   factoring out the summarizer content. The audit itself notes this
   issue in §E but doesn't propose the factor-out; without it, the
   skill becomes harder to navigate.
4. **The proposed Step 8 audit checklist's bullet 6** ("Estimate
   reduction loss... 128-bit signature with 2^140 loss needs ~268-bit
   underlying hardness") is correct in form but in practice
   community-standard lattice multi-sig parameters do not absorb
   `N·Q^2` losses in their hardness baseline. If the skill prescribes
   this as a hard requirement, it would mark every paper in this
   lineage as "broken" — which is technically defensible but not
   actionable. Soften to "report the loss; flag if it exceeds the
   author's stated tolerance".

## Overall

The audit's proposals are mostly good and well-targeted at a real
usability gap (theory-lens users find ~40% of the skill content
relevant). The single highest-value change is the lemma-dependency
diagram (audit §F's choice is correct). The audit's content additions
are factually accurate against the paper. The main risks are
(a) skill bloat without compensating refactor, and (b) one or two
over-aggressive deletions/consolidations. If the aggregator applies
~70% of the proposed edits (the diagram, the Lemma 3.5 expansion, the
reduction-tightness category, the `RSIS rop` gloss, the Hint-MLWE
comparison table, and the new Step 9 in prior-work-survey) and
refuses the over-aggressive ones (Step 4/7 consolidation, full
deletion of paper-analysis Step 2 table), the skills will be
materially better for theory users without becoming unwieldy.
