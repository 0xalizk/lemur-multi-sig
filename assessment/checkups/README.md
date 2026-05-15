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

Unverified:

- **Realized aggregate-signature size and Secure-aggregation timing at N=2¹⁰** — require a full `bench --fast` run that completes the aggregation step. The 8 GiB host OOM-killed `bench --fast` at the N=8192 signer-replication stage; a 16+ GiB host should clear both. The formula-predicted aggregate size 201.2 KB is reproducible from `lemur sizes` without running aggregation. [details](unresolved.md)

[full ledger](unresolved.md) · [missed-angles ledger](open-issues.md)
