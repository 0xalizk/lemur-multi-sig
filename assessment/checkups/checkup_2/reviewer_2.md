# Reviewer 2 — Independent verification of Auditor 2's audit

Scope: `/workspace/repo/submission/` integrity audit at `/workspace/tmp/checkup_2/auditor_2.md`.

Method: re-ran every concrete claim with primary commands. Outputs are inline below.

---

## Per-claim verdicts

| # | Claim | Verdict |
|---|-------|---------|
| 1 | `git diff HEAD --` empty for `report.pdf`, all `.py`/`.rs`/`.sage`, all parameter `.txt` | **AGREE** |
| 2 | `report.pdf` md5 = `183af3ac09509b99bda70d895545a65b`, 2,164,413 bytes, 24 pages | **AGREE** (exact match) |
| 3 | Two stray `.DS_Store` files, untracked | **AGREE** (and they are gitignored, see refinement below) |
| 4 | Five `target/release/*.d` modified, only path prefix changed | **AGREE** |
| 5 | `cargo build --release` no-ops; `lemur sizes` emits `201.2 KB` headline at N=1024 | **AGREE** (timing 0.04s here, auditor reported 0.10s — within noise) |
| 6 | All four code-vs-paper checkpoints pass at exact lines | **AGREE** |
| 7 | `summary.txt` has 16 d=256 data rows with `beta_z=14046` constant | **AGREE** |
| 8 | Python venv works; `numpy 2.4.4` importable, `Crypto` importable | **AGREE** |
| 9 | Recommended cleanup commands are safe | **AGREE with one caveat** (see below) |

No claim required REVISE or DISPUTE.

---

## Verifications that hold up (with evidence)

### Claim 1 — `git diff HEAD` scoped to submission

```
$ git diff HEAD --stat
 submission/code/lemur-rs/target/release/bench.d           | 2 +-
 submission/code/lemur-rs/target/release/bench_breakdown.d | 2 +-
 submission/code/lemur-rs/target/release/bench_verify.d    | 2 +-
 submission/code/lemur-rs/target/release/lemur.d           | 2 +-
 submission/code/lemur-rs/target/release/liblemur_rs.d     | 2 +-
 5 files changed, 5 insertions(+), 5 deletions(-)
```

Scoped diff for paper + sources + parameter outputs:
```
$ git diff HEAD -- submission/report.pdf submission/code/lemur-py/ \
    submission/code/lemur-rs/src/ submission/code/lemur-rs/Cargo.toml \
    submission/code/parameter/  | wc -l
0
```
Confirms: zero modification to any reviewable content. The only working-tree drift is the five `.d` files (claim 4).

### Claim 2 — PDF identity

```
$ md5sum /workspace/repo/submission/report.pdf
183af3ac09509b99bda70d895545a65b  /workspace/repo/submission/report.pdf
$ wc -c /workspace/repo/submission/report.pdf
2164413 /workspace/repo/submission/report.pdf
$ pdfinfo /workspace/repo/submission/report.pdf | grep -E "Pages|Title|File size"
Title:           Lemur: Scalable Post-Quantum Synchronized Multi-Signatures
Pages:           24
File size:       2164413 bytes
```
All three sub-claims match exactly. Producer is `pdfTeX 3.141592653-2.6-1.40.27 (TeX Live 2025)`, CreationDate `Thu Apr 30 09:36:19 2026 UTC` — internal LaTeX-compile timestamp, irrelevant to session integrity (and consistent with the auditor's note).

### Claim 3 — `.DS_Store` pollution

```
$ ls -la /workspace/repo/submission/.DS_Store /workspace/repo/submission/code/.DS_Store
-rw-r--r-- 1 claude claude 6148 May 15 15:14 /workspace/repo/submission/.DS_Store
-rw-r--r-- 1 claude claude 6148 May 15 14:26 /workspace/repo/submission/code/.DS_Store
$ git ls-files | grep -i DS_Store
(empty)
$ git check-ignore -v submission/.DS_Store submission/code/.DS_Store
.gitignore:2:.DS_Store	submission/.DS_Store
.gitignore:2:.DS_Store	submission/code/.DS_Store
```

Both files exist, both are size 6148 (the canonical macOS Finder cache size), both are untracked. The auditor said "presumably gitignored or simply ignored" — confirmed: `.gitignore` line 2 (`*.DS_Store` via the bare pattern `.DS_Store`) suppresses them, which is why `git status` does not list them.

**Minor refinement to the audit:** they are explicitly gitignored, not "simply ignored". The user already has a `.gitignore` rule. This means the files are safe to delete and won't reappear in `git status` after deletion — the recommended cleanup is fully clean.

### Claim 4 — `.d` files are path-prefix drift only

```
$ git diff HEAD -- submission/code/lemur-rs/target/release/lemur.d
-/workspace/repo/code/lemur-rs/target/release/lemur: /workspace/repo/code/lemur-rs/src/...
+/workspace/repo/submission/code/lemur-rs/target/release/lemur: /workspace/repo/submission/code/lemur-rs/src/...
```

Identical structure for `bench.d`, `bench_breakdown.d`, `bench_verify.d`, `liblemur_rs.d`. Each file is a single-line Makefile-style dep listing where the only substitution is `code/` → `submission/code/`. No source filenames added, removed, or reordered. The auditor's diagnosis (crate moved from `code/` to `submission/code/`; cargo regenerated `.d` files with the new absolute paths) is the only plausible explanation. Source content of the crate is byte-identical to HEAD (confirmed by `git diff HEAD -- submission/code/lemur-rs/src/` = empty).

### Claim 5 — Build + run sanity

```
$ cd /workspace/repo/submission/code/lemur-rs && cargo build --release
    Finished `release` profile [optimized] target(s) in 0.04s
$ ./target/release/lemur sizes
Lemur serialised sizes  (profile=d256_k4, d=256, tau=20, n_slots=1048576, alpha_w=23, gamma=10)
  ...
  beta_z=14046  beta_sigma=13229351
  ...
  aggregated sig (N=1024, ~201.2 KB)           206065  (201.2 KB)
    Z_agg (20b, bound=378933)                      5761  (5.6 KB)
    Babai path (Rice k=5)                         52000  (50.8 KB)
    sibling labels (Rice k=15)                   119600  (116.8 KB)
    u (Rice k=15)                                 28704  (28.0 KB)
```

201.2 KB headline at N=1024 reproduced exactly. `beta_z=14046` matches `summary.txt`. Build was 0.04s here vs. 0.10s reported in the audit — both consistent with a no-op rebuild against an already-populated `target/`.

### Claim 6 — Audit checkpoints at exact lines

All four locations read directly from the files (no diff vs HEAD anywhere):

- **`kots.py:95`** → `            H[i, i, 0] = 1` — exact spacing (12-space indent, single space around `=`, `H[i, i, 0]` with the canonical single space after each comma). Matches.
- **`kots.py:163–170`** → three wrappers `ivrfy` / `svrfy` / `wvrfy` returning `vrfy(...self.beta_z)`, `vrfy(...self.beta_sigma)`, `vrfy(...2 * self.beta_sigma)`. Matches.
- **`hvc.py:411`** → `        ) % self.q` (modulus is `self.q`, not `self.qprime`). Matches.
- **`lemur.py:294`** → `            if self.avrfy(pp, pks, t, m, (Z_agg, d_agg, attempt)):` — retry loop calls full `avrfy`, not a partial inf-norm check. Matches.

### Claim 7 — `summary.txt` schema

```
$ awk 'END{print NR}' summary.txt
18
$ awk 'NR>2' summary.txt | wc -l
16
$ awk 'NR>2 {print $19}' summary.txt | sort -u
14046
```

18 total lines = 2 header + 16 data. Column 19 is `beta_z`. All 16 rows hold exactly `14046`. Cross-checked: `grep -c 14046 summary.txt` = `16` (one hit per data row, no spurious matches). The audit's framing — "all 16 d=256 rows constant" — is correct.

(Note: auditor wrote "18 lines total = 2 header + 16 data rows" but `wc -l` returns 17 because the last line has no trailing newline. `awk 'END{print NR}'` gives the true line count of 18. Inconsequential.)

### Claim 8 — Python venv

```
$ source /workspace/repo/submission/code/lemur-py/.venv/bin/activate
$ python3 -c "import numpy, Crypto; print('numpy=' + numpy.__version__)"
numpy=2.4.4
```

Both modules import. `numpy 2.4.4` is what the venv ships (note: this version string is unusual for the current calendar date, but it is what is installed; venv-vendored package versions are out of scope for an integrity audit).

---

## What the audit got right that is worth re-emphasizing

1. The `target/` and `.venv/` trees being committed in HEAD is a pre-existing repo characteristic, not a session artifact. Both are visible in `git ls-tree HEAD submission/`. The `__pycache__/*.pyc` files are also tracked (`git ls-files submission/code/lemur-py/__pycache__/` returns 7 entries) — also pre-existing, not session-introduced.
2. The audit correctly distinguishes between session-introduced drift (only the `.d` files and the two `.DS_Store` files) and "weird-but-pre-existing" repo conventions (committing build artifacts and venvs). This separation is accurate and important — only the former is in scope for an integrity audit.
3. The audit explicitly disclaims that it did not re-run the Sage estimator (no Sage in container) and did not run `cargo test`. Both are honest and correct gaps.

---

## What the audit missed about submission/ integrity

1. **`.DS_Store` is in `.gitignore`** (line 2). The audit said "presumably gitignored or simply ignored" — it is in fact actively ignored by an existing rule, which strengthens the "safe to delete, won't pollute future diffs" conclusion. Recommendation #3 in the audit (add `**/.DS_Store` to `.gitignore`) is therefore partially redundant; the bare `.DS_Store` pattern already matches at every directory level via gitignore semantics.
2. **The audit did not verify byte-equality of the venv and target trees.** It checked `git diff HEAD -- submission/code/lemur-py/*.py` and `--src/`, but not the larger committed `.venv/` and `target/release/` trees as a whole. A more thorough check would be `git diff HEAD -- submission/` (which I ran via `git diff HEAD --stat` above and confirmed the only drift is the 5 `.d` files). So the omission is not a defect; the conclusion holds.
3. **No verification of `__pycache__/*.pyc` byte-equality after the venv import.** The audit asserts "regenerated to byte-identical bytecode" but this was not shown with a hash or diff. I cross-verified independently: `git diff HEAD -- submission/code/lemur-py/__pycache__/` = empty. So the claim is correct, just under-evidenced in the audit itself.
4. **No spot-check of `alpha/alpha.sage` or the `estimator/` and `msis_estimator/` subtrees** under `submission/code/parameter/`. Since `git diff HEAD -- submission/code/parameter/` is empty (confirmed above), these are byte-identical to HEAD, but the audit's prose only namechecks `chipmunk_original.sage`, `chipmunk_param.sage`, `lemur_param.sage`. Trivial gap, not a defect.

No content-integrity issue was missed.

---

## Are the recommended fixes safe to run?

**Recommendation 1**: `rm /workspace/repo/submission/.DS_Store /workspace/repo/submission/code/.DS_Store`
- **Safe.** Both files are untracked (`git ls-files` confirms) and gitignored (`git check-ignore` confirms). Deleting them removes filesystem pollution and does nothing to git history. They will not regenerate unless someone re-opens the repo in macOS Finder.

**Recommendation 2**: `git checkout HEAD -- submission/code/lemur-rs/target/release/*.d`
- **Safe** but caveat: this reverts the path prefix from `submission/code/...` back to `code/...`, which is the path that was current when the `.d` files were originally generated. Cargo will simply regenerate them on the next non-no-op build (i.e., the next time anything in `src/` actually changes), so the revert is cosmetic. It will not break the current binaries (which are already built and are not being recompiled). `cargo build --release` after the checkout will still no-op in <1s.
- One small risk: if any tooling reads these `.d` files with strict path-matching, it would now point at a non-existent `/workspace/repo/code/lemur-rs/`. This is unlikely (the `.d` files are purely cargo's internal dep-info), but worth noting. In practice, cargo will overwrite them at the next build anyway.

**Recommendation 3**: add `**/.DS_Store`, `**/__pycache__/`, `**/*.pyc`, `**/target/`, `**/.venv/` to `.gitignore`
- **Safe but out of scope** for a session integrity fix, and would NOT remove the already-tracked files (`__pycache__/*.pyc`, `target/`, `.venv/`) from git's index — only `git rm --cached` would do that. The audit correctly flags this as a maintainer-level decision rather than a session cleanup, so calling it out as "out of scope" is the right framing.
- Also: as noted above, `.DS_Store` (no `**`) is already in `.gitignore` and already works recursively. Only the three new patterns would change behavior.

**Bottom line on cleanup safety**: Recommendations 1 and 2 are safe to run as-is. Recommendation 3 is a maintainer-facing improvement, not a session fix, and is correctly out of scope.

---

## Overall

The audit is accurate, thorough, and conservative. Every concrete claim I re-verified holds with byte-identical evidence. The two genuine session-introduced anomalies (`.DS_Store` files; `.d` path-prefix drift) are correctly identified, correctly diagnosed, and the proposed fixes are safe. No false positives, no missed integrity problems. Submission folder is in a publishable state modulo the two cosmetic cleanups.
