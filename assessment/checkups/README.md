# Checkups

Raw audit transcripts behind [`../review.md`](../review.md):
[`checkup_0/`](checkup_0/) paper audit (theory / implementation /
field lenses), [`checkup_1/`](checkup_1/) per-claim fact-check on
`review.md` (197 atomic claims), [`checkup_2/`](checkup_2/)
repo-wide integrity sweep. Each round had an independent reviewer
pass.

## Open issues

Unverified further:

- **Walltime totals.** Rows in root `README.md` sum to ~2h35m, header reports ~2h45m. [details](checkup_2/auditor_1.md)
- **Genealogy tree alignment.** Pipe under "Non-synchronized" is off by ~5 columns in root `README.md`. [details](checkup_2/auditor_1.md)
- **Vendored `.venv/` + `__pycache__/`** in git history — 4,647 tracked files, ~3,994 venv + ~1,551 pyc; needs `git rm --cached`. [details](checkup_2/auditor_1.md)
- **No runtime comparison vs Chipmunk** in review §10 — captured as reviewer add-on, not as audit finding. [details](open-issues.md)
- **EUF-RK vs standard-model for Anada et al.** — review §9.1 calls it a "standard-model sibling" without confirming its security-model target. [details](open-issues.md)
- **RHF ≤ 1.0045 threshold** chosen by the paper itself — could be a parameter that disadvantages Chipmunk's ability to re-fit. [details](open-issues.md)
- **Drake-vs-Lemur timing self-undercut** — Lemur's own ~12 min aggregation at N=2²⁰ falls in the same range as the pqSNARK aggregation it dismisses as "heavy proof machinery". [details](open-issues.md)
- **7 atomic bench measurements** still unverified — one fresh `bench --fast` run + 2 `wc -l` commands clears all. [details](unresolved.md)
