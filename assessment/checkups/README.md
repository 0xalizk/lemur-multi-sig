# Checkups

Raw audit transcripts behind [`../review.md`](../review.md):
[`checkup_0/`](checkup_0/) paper audit (theory / implementation /
field lenses), [`checkup_1/`](checkup_1/) per-claim fact-check on
`review.md` (197 atomic claims), [`checkup_2/`](checkup_2/)
repo-wide integrity sweep. Each round had an independent reviewer
pass.

## Open issues

Substantive:

- **No runtime comparison vs Chipmunk** in review §10 — captured as a reviewer add-on, not as an audit finding. [details](open-issues.md)
- **EUF-RK vs standard-model for Anada et al.** — review §9.1 calls it a "standard-model sibling" without confirming its security-model target. [details](open-issues.md)
- **RHF ≤ 1.0045 threshold** chosen by the paper itself — could be a parameter that disadvantages Chipmunk's ability to re-fit. [details](open-issues.md)
- **Drake-vs-Lemur timing self-undercut** — Lemur's own ~12 min aggregation at N=2²⁰ falls in the same range as the pqSNARK aggregation it dismisses as "heavy proof machinery". [details](open-issues.md)

Unverified (session-specific empirical claims — all resolvable in ~20 min of `bench --fast` + `wc -l`):

- **`bench --fast` realized aggregate size** (195.8 KB at N=1024; review §5.2 cites ~3% Rice-coding variance around the 201.2 KB formula)
- **Online sign (KOTS-only) timing** (304 µs; review §6 row, absolute number not re-measured)
- **Stateful sign (BDS08)** (3.91 ms; review §6 row)
- **Batch verify at N=2¹⁰** (79.31 ms; review §6 row)
- **Derived ratios in review §6** (1:26000 stateful:full-sign and 18:1 aggregation:batch-verify on the test host)
- **Python LOC ~3.6 kLOC, Rust LOC ~9 kLOC** (review §4 intro; order-of-magnitude plausible but not counted)

[full ledger](unresolved.md) · [missed-angles ledger](open-issues.md)
