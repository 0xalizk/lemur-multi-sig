# Checkups — audit & fact-check trail

This folder is the raw transcript of how
[`../review.md`](../review.md) was produced and stress-tested.
Three rounds of subagent work, each with a primary pass and an
independent verification pass. Useful if you're an agent exploring
the repo and want to know what was checked, what's known to be
right, and what's still open.

## What's in each round

| Round | Scope | Files |
| --- | --- | --- |
| **checkup_0** | First pass on `review.md` itself. Three lens-specific auditors (theory / implementation / field-comparison) plus three reviewer subagents. | `checkup_0/subagent_{1,2,3}_{paper,skills}_feedback.md` (audits) and `checkup_0/review_subagent_{1,2,3}_{paper,skills}_feedback.md` (per-audit reviews). |
| **checkup_1** | Fact-check of every atomic claim in `review.md` — 197 atomic claims, each assigned VERIFIED / PARTIALLY / FLAG / UNVERIFIABLE. Three FCs split by section group (§1-3 / §4-7 / §8-11) + three reviewers. | `checkup_1/fact_checker_{1,2,3}.md` and `checkup_1/review_fact_checker_{1,2,3}.md`. |
| **checkup_2** | Whole-repo integrity sweep after multiple readability + restructure passes. Three auditors (README + repo hygiene / `submission/` byte-integrity / `assessment/review.md` internal consistency) + three reviewers. | `checkup_2/auditor_{1,2,3}.md` and `checkup_2/reviewer_{1,2,3}.md`. |

## Bottom-line per round

- **checkup_0** — paper-level audit. No factual errors in the
  paper's claims that the audits could falsify; surfaced ~30
  "missed angles" / "should-have-caught" items the audits didn't
  pursue, captured in [`open-issues.md`](open-issues.md). The
  KOTS-shrink ratio framing ("order of magnitude" vs the actual
  4.6–12.8×) and the Rice-vs-raw asymmetry in paper Table 1 are
  the most consequential observations from this round.
- **checkup_1** — claim-by-claim ledger. Of 197 atomic claims:
  157 VERIFIED (80%), 16 PARTIALLY CORRECT (8%), 5 FLAG (2.5%),
  7 UNVERIFIABLE (3.5%), 12 framing. All 5 FLAGs traced to the
  same §6 column-attribution problem (paper Table 2 numbers mixed
  with `code/README.md` numbers), now fixed. Remaining
  UNVERIFIABLE atomic items live in
  [`unresolved.md`](unresolved.md).
- **checkup_2** — repo hygiene + structural integrity. Found
  three actionable defects (a stray `[↩](#ref-4b)` orphan
  back-arrow in `review.md` footnote 4; three `.DS_Store` files;
  five `.d` files with path-prefix drift from the `code/` →
  `submission/code/` rename), all applied in commit `a9ded67`.
  Submission folder content (paper PDF + every source file) is
  byte-identical to the original commit per `git diff HEAD`.

## Companion logs

- [`unresolved.md`](unresolved.md) — atomic UNVERIFIABLE items
  from checkup_1, with resolution status (7 still unresolved at
  end-of-round; all are session-specific bench measurements
  resolvable by running `cargo run --release --bin bench -- --fast`
  and `wc -l` on the Python sources).
- [`open-issues.md`](open-issues.md) — higher-level "missed
  angles" from both checkup_0 and checkup_1 reviewers. ~20 items
  spanning theory micro-gaps, implementation provenance,
  field-placement framing, and skills-usability.
- [`notes-related-work.md`](notes-related-work.md) — per-reference
  taxonomy for the Lemur paper's 17 cited works, built during the
  prior-work survey skill development.
- [`boneh-kim-2020-agg-ots.pdf`](boneh-kim-2020-agg-ots.pdf) — a
  copy of Boneh-Kim '20, fetched for the competing-paradigm
  check in checkup_0.

## Open items (still unresolved)

Items the audits + reviewers identified that have **not** been
auto-applied to `review.md` or `README.md`. Most are cosmetic or
require a judgement call; the technical content of the review is
unaffected.

**From checkup_2 (most recent):**

- **README walltime arithmetic.** Table rows sum to ~2 h 35 min
  but both the top-of-page tag and the table footer say
  `~2 h 45 min`. May be a deliberate "session walltime" vs
  "sum-of-rows" distinction; flag for owner judgement.
- **README genealogy tree alignment.** The `│` under
  "Non-synchronized" is ~5 columns left of the word's midpoint.
  The `🐒` emoji on the Lemur node is ≤1 column off — minor.
- **`.gitignore` is incomplete.** Repo tracks 4,647 files;
  ~3,994 are vendored `.venv/`, ~1,551 are `__pycache__/*.pyc`.
  These were added in the second commit (`734c452`); fixing
  cleanly requires `git rm --cached` plus history thoughtfulness
  — owner-scope.
- **README first-time-visitor essentials.** No top-level H1,
  no one-line "what is this", no directory map, no date or
  attribution. Reviewer-1 disputed this is unconditionally
  needed; depends on whether the README is a public landing or
  an audit companion.

**From checkup_0 + checkup_1 — high-priority for `review.md`
(see `open-issues.md` for the full list):**

- **No runtime comparison vs Chipmunk.** Paper §1.1 acknowledges
  "we are unable to provide a meaningful runtime comparison
  against Chipmunk." `review.md` §10 captures this but only as
  a reviewer add-on, not as an audit finding.
- **EUF-RK vs standard-model for Anada et al.** `review.md` §9.1
  calls Anada et al. (ICISC 2024) a "standard-model sibling" of
  Lemur without checking whether Anada targets EUF-RK
  specifically.
- **The RHF ≤ 1.0045 threshold itself.** Paper sets this and
  notes Chipmunk scripts "could not find feasible parameters"
  at the new RHF. A hostile reviewer would ask whether the
  threshold itself disadvantages Chipmunk's ability to re-fit.
- **Drake-vs-Lemur timing self-undercut.** Paper Table 2 reports
  ~12 min aggregation at `N=2²⁰`, comparable to the
  "seconds-to-minutes" pqSNARK ranges. `review.md` §10 notes
  this; the paper itself doesn't.

**From checkup_1 (atomic-level — see `unresolved.md`):**

- 7 still-unresolved atomic items, all session-specific bench
  measurements that need a fresh `bench --fast` run plus two
  `wc -l` commands to clear. Total resolution cost: ~20 minutes.

## How to use this folder

- **If you're trying to verify a specific claim in
  `../review.md`:** the footnotes there cite paper sections /
  `file:line` / external URLs directly. The
  `checkup_1/fact_checker_*.md` files are the per-claim ledger
  if you want the prior verification verdict.
- **If you're trying to find what's still untested:**
  [`unresolved.md`](unresolved.md) is the precise list.
- **If you're surveying the field for a follow-on paper:** the
  [`open-issues.md`](open-issues.md) reviewer notes flag what's
  worth scrutinising in any Chipmunk-family submission.
- **If you're auditing the audit:** every checkup round had an
  independent reviewer pass; the `review_*.md` files in each
  subfolder list per-claim AGREE / REVISE / DISPUTE verdicts.
  The fact-checking rounds caught a handful of errors in the
  primary audits (Anada venue, raw-Lemur size figure, `8m·ε₂`
  attribution); the corrections are summarised in
  `../review.md`'s footnotes.
