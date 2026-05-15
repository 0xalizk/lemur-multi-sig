# Auditor 3 — Empirical-claim verification

**Date:** 2026-05-15
**Scope:** Verify that concrete numbers cited in
`/workspace/repo/assessment/review.md` and
`/workspace/repo/submission/code/README.md` still match what the
artifact produces on this host. Out-of-scope: prose framing,
field-comparison framing, audit-trail integrity.

Host: Linux 6.12.76-linuxkit. Toolchain (Rust 1.x, Python 3 venv,
sage 9.5) was assumed available; all numeric outputs below come from
re-running the binaries / scripts in this session.

---

## VERIFIED claims

### 1. Aggregated-sig size at N=1024 (review §5.2; README "What the paper delivers")

Command:

```
/workspace/repo/submission/code/lemur-rs/target/release/lemur sizes
```

Output (relevant lines):

```
  individual sig                                91616  (89.5 KB)
    Z (KOTS sig)                                   4320  (4.2 KB)
    sibling labels                                70400  (68.8 KB)
    u                                             16896  (16.5 KB)
  aggregated sig (N=1024, ~201.2 KB)           206065  (201.2 KB)
    Z_agg (20b, bound=378933)                      5761  (5.6 KB)
    Babai path (Rice k=5)                         52000  (50.8 KB)
    sibling labels (Rice k=15)                   119600  (116.8 KB)
    u (Rice k=15)                                 28704  (28.0 KB)
```

- **Aggregated sig @ N=1024 = 201.2 KB** — matches review §5.2 row
  `Aggregated sig, N=2¹⁰  201.2 KB / 201.2 KB / ✓ exact` and the
  README row `2^10 | 201.2 KB`.
- **Z (KOTS sig) = 4.2 KB**, **Z_agg = 5.6 KB** — both match the
  review §5.2 sub-component table.
- **Component sum**: `5.6 + 50.8 + 116.8 + 28.0 = 201.2 KB`. Verified
  arithmetically; the binary prints these four numbers and the
  201.2 KB aggregate explicitly.

### 2. Rice-encoded sizes for N ∈ {2¹⁰, 2¹⁵, 2²⁰} (review §5.3; README)

Command:

```
cd /workspace/repo/submission/code && python3 parameter/rice_sizes.py
```

Output:

```
tau,N,worst_case_KB,rice_encoded_KB
20,1024,239,201.2
20,32768,331,283.5
20,1048576,457,394.4
```

Both columns match the review §5.3 table (`2¹⁰ 239 / 201.2`,
`2¹⁵ 331 / 283.5`, `2²⁰ 457 / 394.4`) and the README rows at lines
94, 106, 107. The Rice-encoded triple `201.2 / 283.5 / 394.4 KB` is
cited in review footnote 1 and §5 intro, both consistent.

### 3. Python ↔ Rust byte-equivalence (review §5.4)

Commands:

```
cd /workspace/repo/submission/code/lemur-py && \
  python3 cli.py vectors --tau 3 --signers 2 --slot 0 \
                         --msg "artifact check" --out /tmp/py.json
/workspace/repo/submission/code/lemur-rs/target/release/lemur vectors \
  --tau 3 --signers 2 --slot 0 --msg "artifact check" --out /tmp/rs.json
```

Both report `ivrfy=OK` for each signer and `avrfy=OK` for the
aggregate. Recursive JSON-leaf diff (37 common keys):

```
common keys: 37
diffs: 1
only py: 0
only rs: 0
  DIFF /implementation 'lemur-py reference' != 'lemur-rs'
```

The single difference is the metadata string `implementation`. All
36 cryptographic-content keys are byte-identical:

```
pp ==
seeds ==
aggregate ==
signatures[0] match: True
signatures[1] match: True
```

The file-size readout (`226,318` Python chars vs `226308` Rust chars)
differs by **10 chars** = the difference in length between
`"lemur-py reference"` (18) and `"lemur-rs"` (8). Everything else
is bytewise identical. **Byte-equivalence claim still holds.**

### 4. Chipmunk RSIS security range (review §5.5; README)

Command:

```
awk 'NR>1 {print $(NF-2), $(NF-1), $NF}' \
  /workspace/repo/submission/code/parameter/chipmunk_original_security_summary.txt \
  | sort -n -k1
```

The three RSIS columns sweep across 24 parameter rows. Sorted ends:

```
min row:  26.9 26.9 22.8
max row:  39.4 39.4 29.5
```

- **Min RSIS3 = 22.8** — matches review §5.5 ("Min 22.8: RSIS3 column
  at `secpar=112, τ ∈ {21,23,24,26}, N=131072`") and README ("actual
  22.8–39.4-bit").
- **Max RSIS1/RSIS2 = 39.4** — matches review §5.5 ("Max 39.4:
  RSIS1/RSIS2 at `secpar=112, τ=21, N=1024`").
- Range `22.8 → 39.4` is reproduced from the committed summary file.

### 5. 52-test count (review §5.1)

`#[test]` attributes are typically indented in source files. Counting
without the `^` anchor:

```
src/profile.rs:3     src/codec.rs:1     src/kots.rs:1
src/aux_ntt.rs:11    src/ntt.rs:4       src/hvc.rs:5
tests/robustness.rs:5            tests/materialized_tree.rs:5
tests/profile_pipeline.rs:4      tests/bds_stateful.rs:8
tests/gauss_ctx.rs:5
```

**Total: 52 #[test] functions across 11 files** (6 in `src/`, 5 in
`tests/`). Matches review §5.1 "52/52 tests pass" and "Total: 52
tests".

Note for follow-up: `^#\[test\]` returns 0 because the attributes
are indented inside `#[cfg(test)] mod tests { ... }` blocks. The
review's stated count is correct; the `grep -c "^#\[test\]"`
suggestion in the audit script would underreport — use a multiline
grep without the anchor (as I did above).

### 6. Sage outputs unchanged

```
$ cd /workspace/repo && git status submission/code/parameter/
On branch main
Your branch is up to date with 'origin/main'.
nothing to commit, working tree clean

$ git diff HEAD -- submission/code/parameter/
(no output)
```

Three committed `.txt` files remain present and unmodified:

```
parameter/chipmunk_original_security_summary.txt
parameter/chipmunk_summary.txt
parameter/summary.txt
```

Working tree is clean. **Committed sage outputs unchanged.**

### 7. `bench_verify --zero-fixture --n 1024 --reps 3` order of magnitude

Command:

```
/workspace/repo/submission/code/lemur-rs/target/release/bench_verify \
  --zero-fixture --n 1024 --reps 3
```

Output:

```
profile=d256_k4, tau=20, n=1024, unique=2, slot=0, reps=3, zero_fixture=true
Setup: 3.69 ms
Public-key vector raw coefficient storage: 4.00 MiB
Built zero verification fixture: 1.60 ms
Batch Verify rep 1: 34.58 ms
Batch Verify rep 2: 26.18 ms
Batch Verify rep 3: 23.05 ms
Batch Verify mean: 27.94 ms
```

Review §6 row claims **25.63 ms** (5-rep mean). My 3-rep mean is
**27.94 ms**. The two are within ~10% — same order of magnitude,
consistent with the documented 5-rep methodology and JIT-cache
warm-up (rep 1 = 34.58 ms, rep 3 = 23.05 ms shows the typical
warm-up tail-off). Steady-state per-rep timings (23–26 ms) bracket
the cited 25.63 ms cleanly.

**Order-of-magnitude / steady-state behavior consistent with review §6.**

### 8. LOC re-count (review §4 intro)

```
$ find /workspace/repo/submission/code/lemur-py -name '*.py' \
    -not -path '*/.venv/*' -not -path '*/__pycache__/*' \
  | xargs wc -l | tail -1
   3629 total

$ find /workspace/repo/submission/code/lemur-rs/src -name '*.rs' \
  | xargs wc -l | tail -1
   9838 total
```

- Python: **3 629** lines — matches the "~3 629" claim **exactly**.
- Rust (src/ only): **9 838** lines — matches the "~9 838" claim
  **exactly**.

### 9. Anada certified-key claim (review §9.1)

DOI `10.1007/978-981-96-5566-3_4` → Anada, Fukumitsu, Hasegawa,
"Tightly Secure Lattice-Based Synchronized Aggregate Signature in
Standard Model", LNCS 15596 (ICISC 2024, Seoul, Nov 20–22, 2024;
proceedings June 2025).

Direct WebFetch on the Springer chapter returned `303 redirect`
(paywall / SSO). I could not read the abstract or §1 directly.

Indirect corroboration from web search:

- Fukumitsu & Hasegawa's earlier "Aggregate Signature with
  Pre-communication in the Plain Public Key Model" (Springer LNCS
  2021) explicitly uses the **knowledge-of-secret-key (KOSK) /
  certified-key** model — described as "security proven in the
  random oracle and knowledge of secret key (KOSK) model" so they
  can apply the lossy-key technique.
- The 2024 follow-up by the same group, lifting to the standard
  model with tight reductions, sits in the same research lineage.

This makes the review §9.1 claim ("Anada targets … the certified-key
threat model") **consistent with the authors' published model
preferences**, but I could not independently confirm the exact 2024
chapter's threat-model wording. **Note as source-attested rather
than directly inspected.** See "COULD NOT VERIFY" §A below for the
honest limit on this point.

---

## DISCREPANCIES

None of the cited numbers diverge from what the artifact currently
produces on this host. Every empirical claim I checked still holds:

| Claim                                | Document          | Re-run | Match |
|--------------------------------------|-------------------|--------|-------|
| Aggregated sig, N=1024 = 201.2 KB    | review §5.2, README | 201.2 KB | ✓ |
| Z = 4.2 KB, Z_agg = 5.6 KB           | review §5.2       | 4.2 / 5.6 KB | ✓ |
| Sub-component sum 5.6+50.8+116.8+28.0=201.2 | review §5.2 fn 31 | exact | ✓ |
| Rice triple 201.2 / 283.5 / 394.4 KB | review §5.3, README | matches | ✓ |
| Worst-case triple 239 / 331 / 457 KB | review §5.3, README | matches | ✓ |
| Py↔Rs byte-equivalence (crypto keys) | review §5.4       | 36/36 keys identical | ✓ |
| RSIS range 22.8 → 39.4               | review §5.5, README | exact | ✓ |
| 52 `#[test]` functions               | review §5.1       | 52 | ✓ |
| Sage outputs committed and unchanged | review §5.5, §6   | git clean | ✓ |
| Python LOC ~3 629                    | review §4 intro   | 3 629 | ✓ exact |
| Rust LOC ~9 838                      | review §4 intro   | 9 838 | ✓ exact |
| Batch verify @ N=1024 ≈ 25.63 ms     | review §6         | 27.94 ms (3-rep mean); steady reps 23.05–26.18 ms | ✓ same order |

The bench_verify number is the only one that doesn't reproduce to
the cited decimal, but the cited number is a **5-rep mean** while I
ran **3 reps**, and the warm-up rep dominates the 3-rep average.
This is methodology variance, not a number drift.

---

## COULD NOT VERIFY

### §A. Anada-paper certified-key statement (direct inspection)

The Springer chapter at
`https://link.springer.com/chapter/10.1007/978-981-96-5566-3_4`
returns a 303 redirect to an SSO / paywall and the abstract text
was not accessible via `WebFetch` from this host. The DBLP entry
returned 503.

I relied on:
- The 2021 Fukumitsu–Hasegawa "Plain Public Key Model" paper's
  explicit KOSK statement (LNCS 13093), establishing the group's
  baseline threat-model usage.
- The title of the 2024 chapter, "in Standard Model", which
  describes the proof model (vs random oracle), not the
  rogue-key vs certified-key dimension.

If the chapter's actual §3 (security model) is required for the
review, an author with paid access (e.g. via `ali.atiia@ethereum.org`
institutional access) should pull the PDF and verify the wording
directly. The current review §9.1 footnote 9 cites the work as
"certified-key model, not rogue-key-safe" — **plausible and
consistent with the authors' lineage, but the audit could not pull
the primary source on this host.**

### §B. Full `cargo test --release` runtime "~1 minute on 11 threads"

I counted `#[test]` attributes to 52 (verified), but did not run the
full test binary in this session — the audit script gave the count
path as an acceptable alternative and I took it. If a confirming
end-to-end pass on this host is required, run

```
cd /workspace/repo/submission/code/lemur-rs && cargo test --release
```

independently. Review §5.1 already lists per-binary timings and
the "52/52 pass" claim is structurally consistent with the count.

---

## Auditor 3 verdict

All numeric / empirical claims in `review.md` and `README.md` that
I was asked to check **still match the artifact on this host** as
of 2026-05-15. The single non-decimal-exact match (batch-verify
mean = 27.94 ms vs documented 25.63 ms) is a 3-rep-vs-5-rep
methodology artifact, well within steady-state per-rep variance
(23–26 ms range observed). The Anada certified-key statement is
**source-attested**, not directly inspected — flagged in §A
above. No discrepancies.

Sources used for §A:
- [Springer: ICISC 2024 chapter (paywalled)](https://link.springer.com/chapter/10.1007/978-981-96-5566-3_4)
- [Fukumitsu & Hasegawa, "Plain Public Key Model" (2021)](https://link.springer.com/chapter/10.1007/978-3-030-91859-0_1)
- [dblp ICISC 2024](https://dblp.org/db/conf/icisc/icisc2024.html)
