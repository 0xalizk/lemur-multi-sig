# Review of `subagent_3_skills_feedback.md` (skill critique)

Independent verification against
`/workspace/.claude/skills/lemur-prior-work-survey/SKILL.md` and
`/workspace/.claude/skills/lemur-paper-analysis/SKILL.md`. Each
proposed edit assessed on whether the underlying problem is real
and whether the proposed remedy improves usability for related-work
surveys.

---

## Per-edit verdicts

### Edit 1 — "Add a 'Bucket disputes' rubric to Step 1; flag Boneh-Kim mis-placement"
**VERIFIED useful.** The underlying observation is correct: the
paper places [2] as KOTS ancestor while the skill correctly classifies
it as a competing paradigm. A general "if the paper's framing
disagrees with the taxonomy, trust the taxonomy" rule generalises
beyond Lemur and would help reviewers of future papers in the
lineage. **Strengthen:** the proposed rubric should also say *why*
this move is common — "a paper may bury a competitor as ancestor
to avoid comparing on size/latency." The audit hints at this but
the wording in the proposed insertion is bland.

### Edit 2 — "Split 'Direct lineage' into 'construction inherited' vs 'style inherited'"
**PARTIALLY USEFUL, slightly over-engineered.** The distinction
between [8]/[9] (construction inherited from Chipmunk/Squirrel) and
[13] (style inherited from Lyubashevsky-Micciancio) is real. But the
existing skill bucket "Direct lineage" already says "Read intro + a
key construction figure" — for [13] there is no construction to
read, just the abstract. A user following the skill literally will
already arrive at the right reading effort. The split would help a
reader who wants pre-flagged depth, but adds taxonomy churn.
**Recommendation:** keep the bucket; add a one-sentence parenthetical
in the existing prose ("style-only descendants like [13] need only
the abstract"). Don't create a new bucket.

### Edit 3 — "Step 2.5: Sanity-check the citation horizon"
**VERIFIED useful.** This is the highest-value addition the audit
proposes. The skill currently has no instruction to build a 12-24
month pre-submission list of works in the same column, which is
exactly the discipline that catches Anada et al. and Aardal et al.
This generalises to any paper in any lineage. The proposed wording
is concrete and actionable. **Sharpen:** the audit should also tell
the user *which databases to search* (eprint.iacr.org by year, dblp
by venue, Google Scholar by keyword in title) rather than just
"eprint.iacr.org and Google Scholar" as if that were enough.

### Edit 4 — "Add a 4th bucket to the paradigm map: PQ-sig + lattice-PoK (Aardal et al.)"
**VERIFIED useful.** The current paradigm map's trichotomy (BLS /
hash+SNARK / lattice-native) does not have a slot for Falcon+LaBRADOR.
Adding it is correct. The proposed ASCII tree is functional but
visually noisier than the original. **Concern:** the proposed tree
also adds Anada et al. as a descendant of Lemur. This is wrong on
direction — Anada et al. is **independent** (ICISC 2024, not a
follow-up to Lemur), and "descendant" implies inheritance which
isn't claimed. The Anada-under-Lemur placement should be a sibling
node, not a child.

### Edit 5 — "Add Drake et al. as 4th flagship abstract to read (Step 5)"
**VERIFIED useful.** The current Step 5 ("read the three flagship
abstracts, then stop") names Squirrel, Chipmunk, Hint-MLWE — all
inside Lemur's lineage. For an **audit** use case (vs a quick
summary), the user needs to also read the competing paradigm's
flagship to assess whether the dismissal is fair. Adding Drake et
al. as a fourth abstract is consistent with how the audit was
actually performed and is generally good advice. **Sharpen:** the
audit could also recommend LeanSig (eprint 2025/1332) as a second
hash+SNARK read for papers submitted post-2025, since LeanSig
explicitly extends Drake's framework.

### Edit 6 — "Add Falcon+LaBRADOR subsection to Step 6 (competing paradigms)"
**VERIFIED useful.** Follows directly from Edit 4. The proposed
subsection text ("Construction / Why it competes / Why it's hard /
Compares against Lemur on") matches the format of the existing BLS
and Drake et al. subsections, so it slots in cleanly. **Verified:**
the technical claim about LaBRADOR being "interactive at heart"
and Fiat-Shamir requiring careful soundness analysis is consistent
with what the WebSearch on the Aardal et al. paper returned.

### Edit 7 — "Strengthen Boneh-Kim subsection to note log-size aggregate"
**VERIFIED useful.** The current skill says "Cite for completeness;
do not compare numerically" which is *too* dismissive. Boneh-Kim's
log-size aggregate is the technical headline that explains why it's
worth knowing about even though it's non-synchronized. The proposed
addition is informative and correct.

### Edit 8 — "Add 8th red flag in Step 7: same encoding in all comparison columns"
**VERIFIED useful.** Rice/Golomb-coded vs raw asymmetry is already
mentioned in red-flag #2, but the audit's point is that the wording
focuses on *the paper under audit* using one or the other, not on
*the comparison table* mixing them. Promoting this to its own red
flag is fair. **However:** the proposed wording duplicates the
existing #2 substantially. Recommend merging into #2 with a
sub-bullet rather than creating #8.

### Edit 9 — "Step 9: Post-submission supersession check"
**PARTIALLY USEFUL.** The intent is good — when reviewing a paper
in 2026 that's dated 2026, check what came out post-submission. But
the proposed wording conflates two distinct cases:
- (a) papers the author *could have* cited in revision (these matter
  for the "weak related work" critique)
- (b) papers that genuinely post-date camera-ready (these matter for
  "is this paper still current?", not for blaming the authors)
The audit's wording mixes both. **Recommendation:** keep the step
but make the distinction explicit so reviewers don't unfairly
penalise authors for not citing genuinely-later work.

### Edit 10 — "Step 3 of `lemur-paper-analysis` (security claims): say '23-40 bits' not '~70 bit gap'"
**VERIFIED useful with caveat.** The current skill says "fail this
estimator by ~70 bits" → implies 112-70=42 bits, which is in the
right ballpark for the median cell. The audit's "23-40 bits" range
is the *correct* phrasing and matches `chipmunk_original_security_summary.txt`. **Caveat:** the skill should state which RSIS column
is being minimised over and clarify that the lowest cell is
SIS-3 (RSIS3 column), at large N. Without that, future users will
re-derive different numbers depending on which column they pick.

### Edit 11 — "Add τ-cell-mismatch warning Chipmunk vs Lemur"
**VERIFIED useful.** Chipmunk's summary file uses τ ∈ {21, 23, 24, 26}
while Lemur's tables use τ ∈ {12, 16, 20, 24}. A user comparing
cells could match by τ and silently compare incomparable settings.
The proposed warning is small but high-value.

### Edit 12 — "Promote Rice asymmetry warning to top-level in Step 5"
**VERIFIED useful.** Same observation as Edit 8 — the encoding
asymmetry is currently buried at bullet 3 of 5 in Step 5, and is
specifically pertinent to Table 1 of Lemur. Promoting it to its own
labeled paragraph is fair.

### Edit 13 — "Step 6 of `lemur-paper-analysis`: note absent Falcon+LaBRADOR, Anada et al. comparisons"
**VERIFIED useful** for the present paper. But this is a paper-specific
fact, not a general reading-skill teaching. A skill should not
hard-code the absence of one specific reference; better is to teach
"check for post-2023 PQ aggregate signature papers in adjacent
paradigms" generally. The audit conflates "fix this specific
skill's blind spot" with "make the skill teach a general method".
**Recommend:** keep the spirit, generalise the wording.

---

## Audit-internal issues

### Internal inconsistency about Anada et al.
In §6.3 of the audit, Anada et al. is cited as "Anada-Fukumitsu-
Hasegawa '25 (ProvSec 2025)"; the skills critique inherits the same
attribution. WebSearch shows the venue is **ICISC 2024**, not ProvSec
2025. Audit skills-feedback file should be corrected accordingly.

### Reliance on un-cited claims about HAPPIER numbers
The audit asserts "HAPPIER reports 2-3 MB aggregate sigs and 2^16
signers on a laptop" without citing the paper section or page. The
HAPPIER paper exists (verified) but the specific numbers are not
double-checked here. Skill edits derived from these numbers (e.g.,
"~5-8× smaller" claims) should be flagged as "audit-asserted, not
verified" if the underlying numbers haven't been confirmed.

---

## Bad suggestions that would harm the skill

1. **The replacement ASCII paradigm tree in Edit 4** is visually
   busy and hard to read. The original tree is ASCII-friendly; the
   replacement uses Unicode box-drawing in awkward positions
   (e.g., the "├" hanging mid-line) that won't render well in
   monospace fonts. Recommend keeping the original tree structure
   and just *adding* the 4th bucket as a sibling.
2. **Placing Anada et al. as a child of Lemur** in the tree is
   misleading: Anada et al. is a sibling (independent contemporaneous
   work), not a descendant. The proposed tree gets this wrong.
3. **Edit 8 creating a separate red-flag #8 for encoding** when the
   existing #2 already covers the same ground is taxonomic bloat.
   Merge as a sub-bullet of #2 instead.
4. **Edit 13's hard-coding of "Lemur doesn't compare to Falcon+LaBRADOR
   or Anada et al."** into the skill text — this is a fact about
   one paper, not a general teaching. If the skill is reused on a
   future paper that *does* cite these, the hard-coded claim will be
   wrong. The skill text should teach the discipline of checking,
   not assert a specific gap.

---

## Things the audit missed about skill usability

1. **The skills' "files this skill uses" sections** (bottom of each
   SKILL.md) reference paths under `/workspace/tmp/`. These are
   ephemeral working-directory paths, not stable repo paths. A
   skill that survives across sessions needs paths that won't be
   wiped. Audit doesn't flag this; it should.
2. **Neither skill mentions how to invoke pdftotext** despite both
   relying on grep-able PDF text. The user-facing reading order in
   `lemur-paper-analysis` Step 1 assumes the user already has a
   plain-text or pdftotext'd version. A "preprocess the PDF" Step
   0 would help first-time users.
3. **The `lemur-prior-work-survey` Step 5 names "three flagship
   abstracts" but doesn't tell the user where to fetch them**
   (eprint.iacr.org URLs aren't given). For first-time use, the
   skill assumes prior knowledge of where IACR ePrint lives.
4. **No skill addresses how to triage references by submission-date
   rather than by paradigm.** This is the discipline that catches
   post-submission supersession but is more general than just
   "check the citation horizon" — it's also useful when reading
   any paper that's dated months or years before the present.
5. **`lemur-paper-analysis` Step 4's `α`-disambiguation list** has
   an error the audit didn't catch: it lists "α_w, α_H, α_w again"
   as the three α's. The third entry "α_w again" is a typo for
   "α (KOTS secret width)". Worth correcting while making other
   edits.
6. **Neither skill cross-references the other.** A user invoking
   `lemur-prior-work-survey` doesn't know to also load
   `lemur-paper-analysis`; vice versa. A two-line "see also"
   pointer in each would improve composition.

---

## Summary of recommendations on the audit's table of edits

| Audit edit | Verdict |
| --- | --- |
| Step 1 "Bucket disputes" rubric | VERIFIED useful |
| Step 1 split "Direct lineage" two-way | PARTIALLY USEFUL — over-engineered |
| Step 2.5 "Sanity-check citation horizon" | VERIFIED — highest value |
| Step 3 add 4th paradigm bucket | VERIFIED useful (but redo the ASCII tree) |
| Step 3 add Anada under Lemur in tree | WRONG placement — should be sibling |
| Step 5 add Drake et al. as 4th must-read | VERIFIED useful |
| Step 6 Falcon+LaBRADOR subsection | VERIFIED useful |
| Step 6 sharpen Boneh-Kim note | VERIFIED useful |
| Step 7 add encoding-asymmetry red flag #8 | PARTIALLY — merge with #2 |
| Step 9 post-submission supersession check | PARTIALLY — clarify "could have cited" vs "post-CR" |
| Step 3 (`lemur-paper-analysis`) "23-40 bits" not "~70-bit gap" | VERIFIED useful (state which RSIS column) |
| Step 4 τ-cell-mismatch warning | VERIFIED useful |
| Step 5 promote Rice asymmetry top-level | VERIFIED useful |
| Step 6 note absent Falcon+LaBRADOR | PARTIALLY — generalise, don't hard-code |

**Highest-value edits (worth implementing first):**
1. Step 2.5 citation-horizon check
2. Bucket-disputes rubric
3. Paradigm-map 4th bucket (with sibling-placement fix for Anada)
4. τ-cell-mismatch warning in `lemur-paper-analysis`

**Edits to drop or rewrite:**
- The replacement ASCII tree (cosmetic regression)
- Hard-coded "absent Falcon+LaBRADOR comparison" claim (paper-specific
  not skill-general)
- The split of Direct-lineage bucket (over-engineering)

**Things the audit missed (worth adding):**
- Fix the `α_w` typo in `lemur-paper-analysis` Step 4
- Add cross-references between the two skills
- Stabilize the `/workspace/tmp/` paths or note they are ephemeral
- Add a Step 0 for "preprocess the PDF" (pdftotext invocation)
