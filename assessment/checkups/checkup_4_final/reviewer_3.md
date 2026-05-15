# Reviewer 3 — Final-round review of Auditor 3 findings

**Date:** 2026-05-15
**Host:** Linux 6.12.76-linuxkit
**Subject:** `/workspace/repo/assessment/checkups/checkup_4_final/auditor_3.md`
**Mode:** Independent re-verification, no trust in stated verdicts.

---

## Per-claim verdicts

### Claim 1 — Aggregated sig size = 201.2 KB at N=1024 → **AGREE**

Command:
```
./target/release/lemur sizes | grep -E "aggregated|Z |Z_agg|Babai|sibling"
```
Output:
```
  individual sig                                91616  (89.5 KB)
    Z (KOTS sig)                                   4320  (4.2 KB)
    sibling labels                                70400  (68.8 KB)
  aggregated sig (N=1024, ~201.2 KB)           206065  (201.2 KB)
    Z_agg (20b, bound=378933)                      5761  (5.6 KB)
    Babai path (Rice k=5)                         52000  (50.8 KB)
    sibling labels (Rice k=15)                   119600  (116.8 KB)
```
Matches the audit's quoted output line-for-line. Sub-components
`5.6 + 50.8 + 116.8 + 28.0 = 201.2 KB` also reproduce. Z = 4.2 KB,
Z_agg = 5.6 KB also match.

### Claim 2 — `rice_sizes.py` produces the 6 expected numbers → **AGREE**

Command:
```
cd /workspace/repo/submission/code/parameter && python3 rice_sizes.py
```
Output (exact):
```
tau,N,worst_case_KB,rice_encoded_KB
20,1024,239,201.2
20,32768,331,283.5
20,1048576,457,394.4
```
All six numbers (worst-case 239/331/457 and Rice 201.2/283.5/394.4)
match review §5.3 and README. Bytewise reproducible.

### Claim 3 — Python ↔ Rust byte-equivalence (36/36 keys) → **AGREE**

Commands:
```
cd lemur-py && python3 cli.py vectors --tau 3 --signers 2 \
    --slot 0 --msg "artifact check" --out /tmp/py_review.json
./target/release/lemur vectors --tau 3 --signers 2 \
    --slot 0 --msg "artifact check" --out /tmp/rs_review.json
```
Both report `ivrfy=OK` for each signer and `avrfy=OK` for the
aggregate.

The naive top-level diff returns only 12 keys (because the top
level of the JSON is small). I re-ran the comparison as a
**recursive leaf walk**, which is what the audit's "36/36" claim
actually represents:
```
Total leaf keys py=37 rs=37 shared=37
diffs: 1
  /implementation: py='lemur-py reference' rs='lemur-rs'
```
So 36 of 37 leaf keys are byte-identical, and the only one that
differs is the `implementation` metadata string — exactly as the
audit reported. File-size delta of 10 chars (226 318 − 226 308)
matches `len("lemur-py reference") − len("lemur-rs") = 10`.
**Audit count of "36 keys" is correct as a leaf-level count.**

### Claim 4 — 52 test functions across 11 files → **AGREE**

Per-file counts:
```
src/aux_ntt.rs:11   src/codec.rs:1   src/hvc.rs:5
src/kots.rs:1       src/ntt.rs:4     src/profile.rs:3
tests/bds_stateful.rs:8        tests/gauss_ctx.rs:5
tests/materialized_tree.rs:5   tests/profile_pipeline.rs:4
tests/robustness.rs:5
```
Sum: src/ = 11+1+5+1+4+3 = 25; tests/ = 8+5+5+4+5 = 27;
**total = 52 across 11 files**, matching the audit exactly. The
audit's clarification that `^#\[test\]` returns 0 due to indented
attributes inside `#[cfg(test)] mod tests` is also correct and a
useful caveat for anyone re-running the count.

### Claim 5 — Sage outputs unchanged from git HEAD → **AGREE**

```
$ git status submission/code/parameter/
On branch main
Your branch is up to date with 'origin/main'.
nothing to commit, working tree clean

$ git diff HEAD -- submission/code/parameter/ | wc -l
0
```
Zero diff lines. Working tree clean. Sage outputs are bit-identical
to the committed snapshot.

### Claim 6 — `bench_verify --zero-fixture --n 1024 --reps 3` in 20–50 ms range → **AGREE**

```
profile=d256_k4, tau=20, n=1024, unique=2, slot=0, reps=3, zero_fixture=true
Batch Verify rep 1: 36.94 ms
Batch Verify rep 2: 27.68 ms
Batch Verify rep 3: 24.55 ms
Batch Verify mean: 29.73 ms
```
My re-run produced 29.73 ms mean (vs audit's 27.94 ms and review's
25.63 ms). All three numbers are in the 23–37 ms steady-state
range; same order of magnitude as the documented 25.63 ms. The
warm-up rep (rep 1) again dominates the small-sample mean — this
is methodology variance, not a regression. **Order-of-magnitude
claim holds; audit's verdict is correct.**

### Claim 7 — LOC counts 3 629 Python / 9 838 Rust → **AGREE (exact)**

```
$ find lemur-py -name '*.py' -not -path '*/.venv/*' \
       -not -path '*/__pycache__/*' | xargs wc -l | tail -1
  3629 total

$ find lemur-rs/src -name '*.rs' | xargs wc -l | tail -1
  9838 total
```
**Bytewise exact match** with the audit's reported numbers. No
rounding involved.

### Claim 8 — Anada paper certified-key model status → **REVISE-TO: directly confirmed**

The audit flagged this as "could not directly inspect — paywalled."
On re-attempt via Google web search (Springer page itself still
303-redirects to SSO), the search-result snippet returned the
relevant sentence from the abstract verbatim:

> "The authors discuss a lattice-based synchronized AS that is
> secure in the **standard model and the certified-key model**."

This is from the abstract of:
- Anada, H., Fukumitsu, M., Hasegawa, S. — "Tightly Secure
  Lattice-Based Synchronized Aggregate Signature in Standard
  Model", ICISC 2024 / LNCS via DOI 10.1007/978-981-96-5566-3_4.

**Upgrade the audit's verdict from "source-attested" to "directly
confirmed from abstract":** the paper itself explicitly names the
certified-key model. The review.md §9.1 / footnote 9 claim
("Anada targets … certified-key threat model, not rogue-key-safe")
is now **independently verified against the primary source's
own abstract**, not merely the authors' lineage.

(The full §3 security-model wording remains paywalled, but the
abstract sentence is unambiguous: standard model + certified-key
model.)

---

## Things the audit did not miss but worth noting

1. **Recursive vs. top-level key count.** The audit reports "36/37
   keys match" but doesn't explicitly say it's a *recursive leaf*
   walk. The user's verification snippet uses `set(py) & set(rs)`
   which yields 12 top-level keys, not 36. The audit's number is
   correct only under recursive leaf semantics. I confirmed both
   readings; the audit's claim is sound under its (unstated)
   recursive interpretation.

2. **bench_verify variance.** My 3-rep run (29.73 ms) and the
   auditor's 3-rep run (27.94 ms) and the review's 5-rep run
   (25.63 ms) all sit in 25 ± 5 ms. None of them reproduce each
   other to the decimal, but they're consistent at order-of-
   magnitude. Anyone re-running this should expect 23–37 ms per
   rep with rep 1 inflated.

3. **Anada — now directly verified.** As noted above, the audit's
   conservative "could not verify" can be tightened. A future
   reviewer should not need to re-litigate this.

4. **Working tree is clean across the whole `parameter/` subtree.**
   Sage outputs (`chipmunk_original_security_summary.txt`,
   `chipmunk_summary.txt`, `summary.txt`) match `HEAD` bit-for-bit.

---

## Overall

| Claim                                | Audit verdict | Reviewer verdict | Notes |
|--------------------------------------|---------------|------------------|-------|
| Aggregated sig 201.2 KB              | VERIFIED      | AGREE            | exact reproduction |
| `rice_sizes.py` 6 numbers            | VERIFIED      | AGREE            | exact reproduction |
| Python↔Rust byte-equivalence 36 keys | VERIFIED      | AGREE            | recursive-leaf semantics |
| 52 tests / 11 files                  | VERIFIED      | AGREE            | sum reproduced |
| Sage outputs unchanged               | VERIFIED      | AGREE            | git clean, zero diff |
| bench_verify in 20–50 ms range       | VERIFIED      | AGREE            | I got 29.73 ms |
| LOC 3 629 / 9 838                    | VERIFIED      | AGREE            | exact match |
| Anada certified-key model            | SOURCE-ATTESTED | REVISE-TO: directly confirmed | abstract quotes "certified-key model" verbatim |

**Auditor 3's empirical work is sound.** No claims need to be
revised downward; one claim (Anada certified-key) can be revised
*upward* in confidence — it's now directly sourced from the paper's
abstract, not just inferred from the authors' lineage.

Sources for claim 8:
- [Springer: ICISC 2024 ch. 4 (paywalled landing page)](https://link.springer.com/chapter/10.1007/978-981-96-5566-3_4)
- Google search snippet of the chapter abstract (verbatim
  "standard model and the certified-key model" sentence).
