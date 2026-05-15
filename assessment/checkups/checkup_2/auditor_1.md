# Auditor 1 — Top-level structure & README audit

Scope: `/workspace/repo/README.md`, `/workspace/repo/.gitignore`, repo root layout.
Out of scope: paper, `submission/` code internals, `assessment/review.md` text (used only for cross-checking README claims).

---

## VERIFIED claims

### Link targets — all four resolve
- `README.md:26` `[paper](submission/report.pdf)` → `/workspace/repo/submission/report.pdf` exists (2.16 MB).
- `README.md:26` `[code](submission/code/)` → `/workspace/repo/submission/code/` exists (LICENSE, Makefile, README.md, `lemur-py/`, `lemur-rs/`, `parameter/`).
- `README.md:26` `[review](assessment/review.md)` → `/workspace/repo/assessment/review.md` exists (63.6 KB).
- `README.md:120` `` [`assessment/review.md`](assessment/review.md) `` → same target, exists.

### "What the paper delivers" numerics — match assessment exactly
- 201 / 284 / 394 KB at `N = 2^{10,15,20}` (README.md:58–59) → assessment confirms exact reproduction at `assessment/review.md:347, 370–372, 836` (`201.2 / 283.5 / 394.4 KB`; README rounds to 201/284/394).
- Worst-case theoretical bound 239 / 331 / 457 KB (README.md:60) → matches `assessment/review.md:370–372, 954`.
- Stateful sign 4.1 ms, batch verify 30.1 ms at `N=2^{10}`, 812 ms at `N=2^{15}`, 25.3 s at `N=2^{20}` (README.md:61–63) → 4.1 ms / 30.1 ms / 812 ms verified at `assessment/review.md:244–245, 488, 492, 494, 956–957`.
- Secure aggregation 567 ms at `N=2^{10}` (README.md:63) → `assessment/review.md:491`.
- Chipmunk 22.8–39.4 bits of core-SVP across 24 (τ, ρ, λ) cells (README.md:70–72) → `assessment/review.md:101, 416, 418–419, 447, 649, 849`.
- "~2.1× smaller at `N=2^{20}`" (README.md:75–76) → `assessment/review.md:99, 688, 844`.
- "4.64× at `N=2^{10}`, ~12.8× at `N=2^{20}`" KOTS shrink (README.md:98–99) → `assessment/review.md:109, 684–685, 860`.
- 52 cargo tests (README.md:82, table row L9) → `assessment/review.md:316, 329, 799, 989`.
- Python ↔ Rust byte-equivalent on `vectors` (README.md:82–83) → `assessment/review.md` §5.4 (`:382`).
- "11 threads vs paper's 24-thread baseline, within ±15 %" (README.md:91–93) → `assessment/review.md:467, 471, 489, 588`.
- Lemma 4.1 `N(Q+1)²` loss, `~2^140` at `N=2^{20}, Q_H=2^{60}`, ~268-bit hardness (README.md:110–113) → `assessment/review.md:193–194, 222–224, 716, 725`.

### "What the assessment found" coverage — every bullet has a §
- Bullet 1 (implementation matches) → `review.md` §4 (`:259`) and §5.1–5.4 (`:310, 331, 362, 382`).
- Bullet 2 (Sage reproduces) → §5.5–5.6 (`:410, 428, 439–440`).
- Bullet 3 (timings track) → §6 (`:465`).
- Bullet 4 caveats (Table 1 asymmetry, KOTS shrink corner, no runtime comparison) → §3 framing (`:105–113`), §5.3 (`:375–379`), §9.2 (`:681`).
- Bullet 5 (Aardal, Anada uncited) → §2 (`:153–160`), §9.1 (`:656`, `:661`).
- Bullet 6 (reduction loss not absorbed) → §3 (`:222–224`), §10 (`:716, :725, :744`).

### Color legend — every table row correctly tagged
README.md:9–19, against the legend at `README.md:5`:
- Toolchain (⬜) — setup, correct.
- Cargo build + tests (🟩) — Rust bench, correct.
- `lemur sizes` + `rice_sizes.py` (🟩) — benchmarks Rust + Python, correct.
- Byte-equivalence (🟩) — Python/Rust, correct.
- `bench --fast` (🟩) — Rust, correct.
- `bench_verify` (🟩) — Rust, correct.
- Sage `chipmunk_param`, `chipmunk_original`, `lemur_param` (🟦) — Sage, correct.
- Audit + fact-check subagents (🟪) — reviews, correct.

### Genealogy tree references — all real
Each name has a corroborating cite in `assessment/review.md`:
- BLS [3] → `review.md:646`.
- Drake '25 [7] → `review.md:647`.
- LeanSig 2025/1332 → `review.md:647` (footnote 45).
- HAPPIER LightSec'25 → `review.md:647` (footnote 46).
- Squirrel '22 [9] → `review.md:648`.
- Chipmunk '23 [8] → `review.md:649`.
- Anada et al. '24 ICISC (standard model) → `review.md:153–155, 879`.
- Boneh-Kim '20 [2] → `review.md:652`.
- Aardal et al. 2024/311 (Falcon + LaBRADOR) → `review.md:116, 661`.

### .gitignore content
- `.gitignore:1` `.gitignore` — odd (self-ignoring), but harmless.
- `.gitignore:2` `.DS_Store` — correct entry.

---

## DISCREPANCIES

### 1. Token-count tag is stale (README.md:3)
- README displays `🦞 712.3k tokens`.
- User's most recent `/context` reads **748.9k** — a +36.6k drift (+5.1 %).
- The number is presented as authoritative (top-of-page summary) and should be updated to the latest reading. **Recommend: bump to 748.9k, or rephrase as a snapshot ("as of <date>") so future drift doesn't silently invalidate it.**

### 2. Walltime table does not sum to the stated ~2 h 45 min (README.md:9–20)
Adding the rows literally:

| Row | Value | Minutes |
| --- | --- | ---: |
| Toolchain | ~15 min | 15.00 |
| Cargo build + tests | ~2 min | 2.00 |
| `lemur sizes` + `rice_sizes.py` | < 1 s | ~0 |
| Byte-equivalence | ~10 s | 0.17 |
| `bench --fast` (multiple runs) | ~50 min | 50.00 |
| `bench_verify` | ~5 min | 5.00 |
| Sage `chipmunk_param` | 13 s | 0.22 |
| Sage `chipmunk_original` | 2 min 45 s | 2.75 |
| Sage `lemur_param` | ~25 min | 25.00 |
| Audit subagents (6) | ~25 min | 25.00 |
| Fact-check subagents (6) | ~30 min | 30.00 |
| **Sum** | | **155.14 min ≈ 2 h 35 min** |

Stated total at `README.md:20`: **~2 h 45 min** (= 165 min). **Sum is ~10 min short of the stated total.**

Two ways to reconcile:
- The stated total is the real wallclock (parallel + idle gaps + retries not itemized), in which case the table is a *subset* — note this with "(activities itemized below; remainder is parallel-batch idle / context-switching)".
- Adjust the total to "~2 h 35 min". Less safe — the user has direct walltime evidence.

### 3. Stray `.DS_Store` files committed to the repo

```
/workspace/repo/.DS_Store          (10244 B, tracked by git? — NO, ignored)
/workspace/repo/submission/.DS_Store
/workspace/repo/submission/code/.DS_Store
```

Verified with `git ls-files | grep DS_Store` → **empty**, so they're correctly gitignored at the *repo* level. However, the file present on disk at the repo root is exactly the kind of artifact the legend tag "⬜ setup" / clean-repo audit notices. **Action: `rm` them; they bloat the working tree even though git ignores them.**

### 4. `__pycache__/` and `.venv/` are tracked in git (major)
`git ls-files | wc -l` → **4647 files tracked**. Breakdown:
- `.venv/` files (vendored pip packages incl. PyCryptodome, numpy, sage helpers, etc.): **3994**
- `__pycache__/*.pyc` files: **1551** (mostly inside `.venv`; **7** are real source caches under `submission/code/lemur-py/__pycache__/`)
- Real, intentional source files: **646**

This means **>85 % of the tracked tree is unwanted artefacts**. `.gitignore` does not list `.venv/`, `__pycache__/`, `*.pyc`, or `target/` (Rust build dir). The two-line `.gitignore` is grossly insufficient for a Rust + Python project. **This is a top-priority hygiene fix.** Recommended additions to `.gitignore`:

```
__pycache__/
*.py[cod]
.venv/
venv/
target/
*.pdf.bak
```

Then `git rm -r --cached submission/code/lemur-py/.venv submission/code/lemur-py/__pycache__` and commit.

### 5. Genealogy-tree alignment issue with the monkey emoji (README.md:50)
Line 50: `                              🐒 <b>Lemur '26</b>`
- 30 leading spaces, then `🐒` (multi-byte, renders as ~2 column-widths), then `<b>Lemur '26</b>`.
- Sibling node `Chipmunk '23` at `README.md:47` has 30 leading spaces.
- The `🐒` glyph pushes the visible "Lemur" label ~1 column right of "Chipmunk" / "Squirrel" in most monospace renderers (GitHub's CSS uses the system emoji font which is roughly 1.5–2 em wide vs. ASCII's 1 em). The vertical `│` at line 49 (column 37) hits between `m` and `u` of "Lemur" instead of under the `i` of "Chipmunk" (column 37 of L47 = the `i` in `Chipmunk`).
- Recent commits `0e18eba` and `44a8179` show the user has been swapping emojis on this node. The 🦞 (also wide) had the same alignment issue.
- **Fix options:** (a) drop the emoji on this line and keep the `<b>` bolding, or (b) widen the `│` indentation on lines 49 and below by 1 space to compensate. Cleanest: just remove the emoji or move it after the label — `<b>Lemur '26</b> 🐒`.

### 6. Pipe drift on the Synchronized/Non-sync subtree (README.md:40–41)
- `README.md:39` ends `Synchronized   Non-synchronized` — these are two sibling nodes.
- `README.md:40` `                                      (slot schedule)` — annotation for **Synchronized only**.
- `README.md:41` `                                            │            │` — pipe under "Synchronized" at col 45, pipe under "Non-synchronized" at col 58.
- "Synchronized" word starts at col 39 of L39, so its midpoint is ~col 45 — pipe at L41 col 45 is correct.
- "Non-synchronized" starts at col 54, midpoint ~col 62 — but pipe is at col 58, **off by ~4 columns to the left**. Boneh-Kim block at L42–43 is also indented relative to that pipe. Minor; readable but not crisp.

### 7. "(slot schedule)" annotation is asymmetric (README.md:40)
Annotation appears under `Synchronized` but **not** under `Non-synchronized`. Reader has to infer that `Non-synchronized` is the complement. Either add a parallel annotation (`(no schedule)`, `(any-time)`) or drop the one on `Synchronized` for symmetry.

### 8. Footnote/numbering style is inconsistent
Genealogy tree mixes numbered cites (`[3]`, `[7]`, `[9]`, `[8]`, `[2]`) with un-numbered names (Aardal et al., LeanSig, HAPPIER, Anada et al.). The numbered ones map to Lemur paper's bibliography; the others are the *missing* citations the assessment flags. A first-time reader sees no key explaining `[2]`/`[3]`/`[7]`/`[8]`/`[9]` — they could be footnotes, citations, or table refs. **Add a one-liner: "Bracketed numbers are Lemur paper bibliography entries; un-numbered names are the assessment's added context (see review §9).**

---

## MISSING / UNCLEAR

### 1. No top-level project title / one-sentence "what is this"
`README.md:26` is the first heading and reads `### Lemur — [paper], [code], and [review]`. It uses H3, not H1. A GitHub landing page typically wants:
- A single H1 `# Lemur` (or `# Lemur Assessment Repo`).
- One-line subtitle: "Audit and reproduction of the Lemur multi-signature paper (anonymous, 2026)."

Without this, someone landing from a link sees the `<details>` toggle first, then the genealogy tree, with no anchor for "what is this repo *for*."

### 2. No reproduction quickstart
The walltime table lists what *was* run but not *how to re-run*. `assessment/review.md` §11 (`:776`) has a full reproduction recipe — the README should link to it explicitly: e.g. `For a reproduction recipe, see [`assessment/review.md` §11](assessment/review.md#sec-11)`.

### 3. No directory map
The README never tells a new visitor what's where. Recommend an explicit tree:

```
submission/
  report.pdf           # the paper under audit
  code/                # author-provided implementation (lemur-py, lemur-rs, parameter/)
assessment/
  review.md            # the audit report
```

### 4. Date of audit / reviewer attribution
No date stamp, no attribution. `submission/report.pdf` exists, `assessment/review.md` exists, but the README doesn't say *who* wrote the review or *when*. The session-summary `<details>` block hints at compute but not at date. Recommend adding "Audit completed 2026-05-15" (today) plus reviewer handle/email near the top.

### 5. Paper bibliography reference for `[2]`, `[3]`, `[7]`, `[8]`, `[9]`
The genealogy tree uses these without a key. The reader can deduce them from context but should not have to. Either expand inline (`BLS '01 [3]`, `Boneh-Kim '20 [2]` ← already done) or drop them.

### 6. No mention of license
`submission/code/LICENSE` exists but the top-level repo has no LICENSE file and the README doesn't address licensing for the *audit* content. Reasonable to add a `LICENSE` at the root or at minimum a footer noting that the audit content is e.g. CC-BY-4.0 / MIT.

### 7. Section "What the assessment found" item 4, sub-bullet 3
README.md:101–102 says: `No runtime comparison vs Chipmunk (acknowledged in §1.1: "we are unable to provide a meaningful runtime comparison")`. The §1.1 reference is to the *paper*, not the assessment — slightly ambiguous since the surrounding bullets cite the assessment. Worth disambiguating: `(paper §1.1)`.

---

## SUGGESTIONS

### High-priority
1. **Fix `.gitignore`** (see Discrepancy #4). The repo is 85 % vendored Python venv. This is the single most important hygiene fix.
2. **Update the token tag** from `712.3k` to the latest `/context` reading (`748.9k`) and/or rephrase to a dated snapshot.
3. **Either fix the walltime sum** (~155 min) or add a footnote explaining what the unitemized ~10 min covers.
4. **Delete the `.DS_Store` files** present on disk (correctly gitignored but still cluttering the working tree).
5. **Add an H1 title and a one-line summary** so the GitHub landing page communicates "what is this" in the first viewport.

### Medium-priority
6. **Move the emoji on the Lemur node** to *after* the label (`<b>Lemur '26</b> 🐒`) — fixes the alignment without losing the visual marker.
7. **Add a directory map** under or near the `### Lemur — [paper], [code], [review]` line.
8. **Link to the reproduction recipe** in `assessment/review.md` §11.
9. **Add a key** for the `[2] [3] [7] [8] [9]` bibliography refs in the genealogy tree.
10. **Symmetrize the synchronized/non-sync annotations** (or remove `(slot schedule)`).

### Low-priority
11. **`.gitignore:1`** — listing `.gitignore` inside itself does nothing useful (file is already tracked); harmless but odd.
12. **Audit-date + reviewer attribution** somewhere near the top.
13. **`<details>` block** is currently the *first* thing on the page (line 3, before any title). Most repos lead with title/summary and put session-resource details further down. Recommend moving the `<details>` block below the title and the "What the paper delivers / What the assessment found" sections — it's reference material, not lead material.

### Stylistic
14. **Number consistency:** README uses `201 KB` (no decimal) but assessment uses `201.2 KB` (one decimal, matching `lemur sizes` output). The exactness is a *finding* of the assessment — the README's rounding obscures that. Consider promoting to `201.2 / 283.5 / 394.4 KB` to match the precision of the audit.
15. **Commit messages** (`git log --oneline -20`) are a coherent story of incremental README tweaks (color-coding, link fixes, details-toggle, emoji swaps). No surprises, no destructive operations. The recent two commits (`0e18eba` swap lobster for monkey; `44a8179` bold + lobster) suggest unresolved iteration on the Lemur node decoration — see Discrepancy #5 for the alignment fix.

---

## Summary

**Verdict:** README is substantively accurate — every numerical claim about the paper and assessment cross-checks against `assessment/review.md`, all four hyperlinks resolve, all named references in the genealogy tree are real, and the color legend is applied correctly.

**Top three things to fix before this lands publicly:**
1. **`.gitignore`** is two lines and the repo carries 4 000 vendored Python files — biggest single hygiene issue.
2. **Token-tag drift** (712.3k → 748.9k) — small but the kind of stale-claim a careful reader will notice and lose trust over.
3. **Genealogy-tree Lemur node** alignment is broken by the emoji — visible the moment anyone renders the README on GitHub.

**Things to add for a first-time visitor:** H1 title + one-line summary + directory map + reproduction-recipe link.
