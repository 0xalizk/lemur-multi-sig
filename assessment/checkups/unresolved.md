# Unresolved fact-check items

Atomic claims that were marked **UNVERIFIABLE** by a fact-checker (checkup_1)
and remain unresolved at the end of the round. Items that a reviewer
later upgraded to VERIFIED are listed in the "resolved by reviewer"
section below with a back-reference.

Layout per entry:

- **ID** — the fact-checker's ledger ID, e.g. `FC2-045`.
- **Where** — section of `repo/assessment/review.md`.
- **Claim** — short paraphrase of the in-review.md statement.
- **Why unverifiable** — what made the fact-checker punt.
- **To resolve** — concrete next step (command, file, URL).

Ledger files: see `checkup_1/fact_checker_{1,2,3}.md` and
`checkup_1/review_fact_checker_{1,2,3}.md`.

---

## Still unresolved at end of checkup_1

### FC2-001 — review.md §4 intro
- **Claim:** Python reference is "~3.6 kLOC".
- **Why unverifiable:** the fact-checker didn't `wc -l` the python files; order-of-magnitude plausible from sampling.
- **To resolve:** `find submission/code/lemur-py -name '*.py' -not -path '*/.venv/*' | xargs wc -l`.
- **Severity:** cosmetic — non-load-bearing.

### FC2-002 — review.md §4 intro
- **Claim:** Rust port is "~9 kLOC".
- **Why unverifiable:** same as FC2-001.
- **To resolve:** `find submission/code/lemur-rs/src -name '*.rs' | xargs wc -l`.
- **Severity:** cosmetic.

### FC2-045 — review.md §5.2
- **Claim:** `bench --fast` realized aggregate-size is **195.8 KB** vs 201.2 KB formula prediction.
- **Why unverifiable:** fact-checker didn't run `bench --fast` (15+ min). The 195.8 figure was reported in an earlier session log, not in the committed repo.
- **To resolve:** `cd submission/code/lemur-rs && cargo run --release --bin bench -- --fast | grep "Agg Sig Size"`.
- **Severity:** documented variance — the review explicitly notes the ~3% Rice-coding fluctuation (footnote 32). Verifying the specific 195.8 KB number is not load-bearing for the claim.

### FC2-061 — review.md §6 Table
- **Claim:** Online sign (KOTS only) measured at **304 µs**.
- **Why unverifiable:** the 304 µs value is from a specific bench run in an earlier session; not committed in the repo. `submission/code/README.md` quotes 347 µs from a 24-thread run.
- **To resolve:** `cd submission/code/lemur-rs && cargo run --release --bin bench -- --fast | grep "Online Sign (mean)"`.
- **Severity:** review.md already flags this row as "implementation-internal, not a paper claim" — verifying the exact 304 µs is informational only.

### FC2-064 — review.md §6 Table
- **Claim:** Stateful sign (BDS08) measured at **3.91 ms** on 11 threads.
- **Why unverifiable:** not committed; needs a fresh `bench --fast` run.
- **To resolve:** as FC2-061.
- **Severity:** the ratio claim against the paper's 4.1 ms (0.95×) is what's load-bearing; the absolute number is one decimal of fluctuation.

### FC2-070 — review.md §6 Table
- **Claim:** Batch verify N=2¹⁰ measured at **79.31 ms** on 11 threads.
- **Why unverifiable:** not committed; needs fresh `bench --fast`.
- **To resolve:** as FC2-061.
- **Severity:** consistent in magnitude with the 2.6× thread-count ratio expected from 24/11; verifying the exact number doesn't change the verdict.

### FC2-072 / FC2-074 — review.md §6 ratios
- **Claim:** Measured ratios "1 : 26 000" and "18 : 1" for stateful-sign-to-full-sign and aggregation-to-batch-verify.
- **Why unverifiable:** derived from FC2-064 / FC2-070 / equivalent measurements, none committed.
- **To resolve:** run `bench --fast`, recompute. Arithmetic: FC2-074's `1410/79.31 = 17.78 ≈ 18:1` consistent with the claimed ratio.
- **Severity:** the *paper-side* ratios (4.1 ms : 1.3 min ≈ 1:19 000, 567 ms : 30.1 ms = 19:1) are paper-stated; only the *measured-side* numbers are unverified.

### FC3-042 — review.md §11 (reproduction recipe)
- **Claim:** `cargo test --release` reports "52 tests".
- **Why unverifiable:** fact-checker did not re-run cargo test.
- **To resolve:** FC2 confirmed this by source tally (25 unit + 27 integration); the corresponding entry FC2-031 is VERIFIED. This FC3 entry is dead-letter.
- **Severity:** **already resolved via FC2-031**; cross-link only.

### FC3-051 — review.md §11 (reproduction recipe)
- **Claim:** Toolchain commands `curl … sh.rustup.rs`, `uv python install 3.11`, `pip install numpy pycryptodome` are correct.
- **Why unverifiable:** syntactically standard but environment-dependent.
- **To resolve:** the entire recipe has been exercised at least three times this session (Rust toolchain bootstrap, Python venv, Sage install, full test run, Sage validation). Empirically verified.
- **Severity:** **already resolved via session history**; cross-link only.

---

## Resolved by reviewer upgrade

These items were filed UNVERIFIABLE by the fact-checker but the
corresponding reviewer pass (`checkup_1/review_fact_checker_X.md`) upgraded
them to VERIFIED. They are listed here for traceability; no further
action.

- **FC1-012 / FC1-013 / FC1-020** — Aardal et al. CRYPTO 2024 and Anada et al. ICISC 2024 existence. Reviewer R1 confirmed via WebSearch; eprint 2024/311 and Springer LNCS 15596 verified. Now footnoted in review.md as [8] and [9].
- **FC1-051** — empirical reference in §3 deferred forward. Resolved by the §5 reproductions documented in review.md.
- **FC2-052** — Python ↔ Rust byte-equivalence. Reviewer R2 re-ran `python3 cli.py vectors` and `lemur vectors` with matching params; all 8 cryptographic JSON fields matched. Now footnote 33.
- **FC3-010 / FC3-026** — HAPPIER LightSec 2025 existence. Reviewer R3 confirmed via WebSearch (Springer DOI `10.1007/978-3-032-15541-2_1`). Now footnote 46.
- **FC3-019** — Boneh-Kim [2] placement in §1.2. Reviewer R3 located the passage at PDF lines 211–213 ("the line of work of Lyubashevsky-Micciancio [13] and Boneh-Kim [2]"). FC3 had grepped for "1.2" but the heading is laid out with tab/whitespace that eluded literal search. Reviewer corrected the verdict.
- **FC3-033** — `8m·ε₂` slack attribution. FC3 marked PARTIAL ("from Lemma 3.3, not Theorem 3.1"); reviewer R3 found the bound is in **Theorem 3.1's statement** at PDF line 434; Lemma 3.3 is a hop *inside* the proof that contributes one summand. Reviewer upgraded to VERIFIED. Footnote 14 already attributes correctly.
- **FC3-040** — PDF line citation "545" (Lemma 4.1 ℓ=1 acknowledgement). FC3 marked PARTIAL; reviewer noted line numbers in `pdftotext -layout` dumps are inherently ±1-2 approximate. Not a real defect.

---

## Summary

Of the 17 entries originally filed UNVERIFIABLE (FC1: 4, FC2: 9, FC3: 4):

- **7 resolved by reviewer upgrade** during checkup_1 (Aardal, Anada, byte-equivalence, HAPPIER, §1.2 placement, 8m·ε₂ attribution, line-number tolerance).
- **2 dead-letter** (FC3-042 already covered by FC2-031; FC3-051 exercised by session activity).
- **8 still genuinely unverified** — all are session-specific measurements (FC2-001/002 LOC tallies; FC2-045/061/064/070/072/074 fresh-bench measurements). None block review.md's central claims; all share a single resolution path: `cargo run --release --bin bench -- --fast` plus a 2-command LOC count.

If a future agent wants to clear all 8 in one sitting: ~20 minutes of `bench --fast` + `wc -l`.
