# Skills feedback — empirical-verification lens

Two skills were available:

  - `/workspace/.claude/skills/lemur-paper-analysis/SKILL.md`
  - `/workspace/.claude/skills/lemur-prior-work-survey/SKILL.md`

Below is what helped, what was unhelpful or missing for the
specific task of doing an empirical / implementation audit
(Python ↔ Rust byte-equivalence, four-checkpoint code audit,
Table 2 ratio reproduction).

---

## 1. lemur-paper-analysis — overall verdict: 70 % useful

### What helped

  - **Step 1 reading order branched by goal** ("Summarize for someone
    else — §1.1, §1.2, §7, and Tables 2/5") let me go straight to §7
    plus Tables 2 and 5 without sinking time in §2–3. This is the
    single biggest win the skill delivered.
  - **Step 4 notation traps** caught one mistake before I made it:
    I would have read α_w = 31 (§7.1 worked example) into the
    "implementation" mental model, but the skill explicitly flags
    "three different α's" so I checked α_w = 23 against the
    profile and immediately spotted the §7.1-vs-implementation gap.
  - **Step 3 categories** (size deterministic, timing
    machine-dependent, security estimator-bound) gave me the right
    order to run things in: size first (`lemur sizes`), then
    timing ratios (batch verify on 11 threads vs 24), then I left
    security to the committed summary.txt rather than try to
    reinstall Sage.
  - **Step 3 timing ratios** ("~ 19:1 secure-aggregate to batch-
    verify at fixed N") was the right framing — I didn't try to
    match 30.1 ms exactly.

### What was unhelpful or missing

  - **No mention of the "implementation cell vs worked example"
    gap.** §7.1 walks `d=128, N=2^20, τ=24, α=61`; the implementation
    is `d=256, k=4, τ=20, N=1024, α=87`. These do not overlap on a
    single parameter. The skill says "the paper switches between
    Gaussian width α and σ" but does not say "the §7.1 worked example
    is not the cell the implementation ships". That's the largest
    gotcha for anyone trying to verify the empirical claims and
    deserves a step of its own.
  - **No `bench --fast` time budget.** "Several minutes per stage,
    --fast 10–20 min" lives only in CLAUDE.md. The skill doesn't
    mention that the `bench --fast` Full Signing stage (seed-only
    rederive) can take 1–2 minutes alone, which can blow a time
    budget if you didn't know. I cancelled mine after 8 minutes
    because the full-sign step seemed to hang; it wasn't, it was
    just slow. A one-line "bench --fast: budget 20+ min, the
    full-sign step alone is 1–2 min" would have saved that
    decision.
  - **The four audit checkpoints from CLAUDE.md are not in the
    skill.** The skill says "Theorem 4.1, the EUF-RK reduction"
    and mentions the figures, but the four foot-gun checkpoints
    that consume "90 % of audit time" are CLAUDE.md content, not
    skill content. If the goal of the skill is to reduce
    onboarding for paper audits, these belong in Step 2 or 3.
  - **No mention of Rice-encoding mechanics.** Step 3 says
    "size disputes always reduce to which parameter cell you're in"
    but does not say "Lemur reports two size numbers per cell —
    a worst-case bound and a Rice-encoded one — and Table 5 only
    shows the worst case." That was a notable trap when reading
    Table 5: the 457 KB total at N=2^20 isn't comparable to the
    "394.4 KB" headline because they're different encodings.

### Concrete edits

**Skill:** `lemur-paper-analysis`

**Section:** Step 1 — front-load the right pages

**Add after the dependency-graph diagram, before the bullet list:**

> Before reading §7, note that §7.1 walks the parameter-selection
> process for a worked example `(d=128, N=2²⁰, τ=24)` that is **not**
> the cell the artifact actually implements. The implementation
> ships `(d=256, k=4, τ=20, N=1024)` (see `parameter/summary.txt`
> line 11, also baked into `lemur-py/profiles.py:106-113`). When
> verifying any §7.1 numerical statement against code, first ask
> "which cell am I in?" — the worked-example α=61 and β_z=8185
> will never appear in the binary; you will see α=87 and β_z=14046.

**Section:** Step 3 — the trap claims to scrutinize

**Add a fourth category after Security:**

> 4. **Encoding-noise size claims.** Lemur reports two size numbers
>    per cell: a worst-case bound (Table 5 totals) and a Rice/Golomb-
>    encoded estimate (Tables 1 and 2). Table 5 totals exceed the
>    Rice estimates by ~14 % — they are not comparable line by line.
>    When checking a "we beat Lemur" claim, first verify which
>    column is being cited.

**Section:** Step 1 — front-load the right pages

**Add a sentence to "Understand the engineering":**

> §7.2 is essential — it is the only place the
> "two interoperable implementations" and the
> "byte-equivalent test vectors" claim is explicit. The
> CLAUDE.md "four checkpoints" list is the empirical version of
> §7.2's correctness story.

**Section:** Step 5 — when comparing schemes within the paradigm

**Add as the fourth bullet (between "Same recomputed security
level?" and "Rice/Golomb-coded sizes or raw?"):**

> - **Same parameter cell?** The Lemur Python and Rust artifacts
>   ship a single concrete `(d, k, τ, N)` cell (currently
>   `(256, 4, 20, 1024)`). Larger-N Table entries are
>   formula-driven extrapolations (sizes via Rice formula, timings
>   via linear extrapolation from N=8192). A "we run at N=2²⁰"
>   comparison must specify whether the comparison is against the
>   measured N=1024 row or the extrapolated N=2²⁰ row.

---

## 2. lemur-prior-work-survey — overall verdict: 50 % useful for *this* task

The skill is good at its stated purpose (surveying the field for a
related-work paragraph) but is misapplied to an empirical audit.
I read it because the user asked me to, and it did not materially
help me check whether the code reproduces Tables 2 and 5.

### What helped

  - **Step 4 KOTS/HVC/Aggregation/BDS08 blueprint** is the same as
    in the paper-analysis skill (Step 2) and lets me triage code
    files quickly: kots.py → KOTS, hvc.py → HVC + BDS08,
    lemur.py → aggregation + composition.
  - **Step 7 red-flag #2** ("Rice/Golomb-coded sizes or raw?")
    is the same trap I flagged independently in the size audit.

### What was unhelpful or wrong for this task

  - **Step 5 ("read the three flagship abstracts, then stop")** sends
    you to Squirrel/Chipmunk/Hint-MLWE abstracts. For an empirical
    audit those are useless — what I needed was the **Chipmunk
    estimator output** (`parameter/chipmunk_original_security_summary.txt`)
    and not the Chipmunk abstract. The skill mentions
    `chipmunk_original_security_summary.txt` in Step 8's files list
    but does not call it out as the empirical artifact for the
    "Chipmunk ~40-bit" claim.
  - **Step 6 BLS/hash-SNARK competitors** is irrelevant for an
    empirical Lemur audit. None of those papers' numbers are
    being compared against in Tables 2 / 5.
  - **The user task brief said "fetch Chipmunk's published sizes
    (eprint 2023/1820 — IACR returns 403, try hackmd/researchgate
    summaries instead)"** but neither skill points at where to
    look for prior-paper size tables. The skill could note that
    the Chipmunk Table 5 reproduction in `report.pdf` itself is
    the authoritative numerical comparison and external lookup
    is unnecessary.
  - **Step 1 bucket table** lists 17 refs; for an empirical audit
    only one matters (Chipmunk [8], because Table 1 / 5 directly
    compares against its numbers). The skill doesn't surface that
    asymmetry.

### Concrete edits

**Skill:** `lemur-prior-work-survey`

**Section:** Step 1 — triage the reference list into four buckets

**Add at end:**

> If your task is **empirical verification** (not paper writing or
> related-work surveying), this skill is over-scoped. You probably
> need only Chipmunk's Table 1/5 comparison values — and those are
> reproduced inside the Lemur PDF itself (paper Table 5 has the
> full side-by-side). External Chipmunk-paper fetches are
> unnecessary for size cross-checks. Skip Steps 5–6 and go
> straight to the comparison column.

**Section:** Step 5 — read the three flagship abstracts, then stop

**Add a paragraph before "The pattern: ..."**

> For empirical verification (rather than related-work prose),
> the more useful artifact than the three abstracts is the
> committed estimator output:
> `parameter/chipmunk_original_security_summary.txt` re-evaluates
> Chipmunk's claimed parameters under the modern Dilithium-style
> estimator and shows RSIS rop in the 26–39 bit range. That file
> is the data behind the "Chipmunk's published parameters
> realise ~40-bit security" claim and is more directly usable
> than the Chipmunk paper text.

**Section:** Step 8 — when summarising the field for someone else

**Add a paragraph titled "When *not* to apply this skill":**

> This skill is for related-work surveying and paradigm
> placement. It is **not** the right tool for:
>   - Code audits (use the four CLAUDE.md checkpoints).
>   - Reproducing Table 2/5 numbers (use `lemur sizes`,
>     `parameter/rice_sizes.py`, and the byte-equivalence
>     vectors check).
>   - Recomputing the security estimator output (needs Sage,
>     and the artifact ships the committed output already).
> If your task is one of those three, stop reading this skill
> and switch to the paper-analysis skill plus CLAUDE.md.

**Section:** Step 7 — red flags

**Edit red flag #1:**

Old text:
> 1. **Same recomputed security level?** Chipmunk's published 112-bit
>    parameters actually realize ~40 bits under modern estimation.

New text:
> 1. **Same recomputed security level?** Chipmunk's published 112-bit
>    parameters actually realize ~40 bits (RSIS rop in 26–39 range)
>    under modern estimation. The grounding artifact is
>    `parameter/chipmunk_original_security_summary.txt` — *committed*
>    estimator output, not a paper claim. If a "we beat
>    Lemur/Chipmunk" submission cites Chipmunk's 112-bit row, ask
>    for the modern-estimator recomputation; in the Lemur artifact
>    one column shows 38.8-bit rop for `(τ=21, N=1024, λ_claimed=128)`.

---

## 3. Cross-cutting gaps both skills share

### 3.1 No mention of the Rice-encoding gap between Table 5 and Table 2

Table 5 reports worst-case bounds (e.g. 457 KB at τ=20, N=2²⁰).
Table 2 reports Rice-encoded sizes (394.4 KB at the same cell).
Neither skill explains that these are the same parameter, encoded
differently. Both should call this out explicitly.

### 3.2 No mention of what the artifact does *not* ship

Both skills' "Files this skill uses" sections list the right
artefacts (`summary.txt`, the estimator outputs, the paper) but
neither says "the binaries only run at N=1024; larger-N rows are
extrapolations". That's the single biggest empirical-verification
trap and should be in Step 3 of the paper-analysis skill.

### 3.3 No timing-budget guidance for the actual benches

The paper-analysis skill says "exact commands live in CLAUDE.md".
That's fair, but the actual cost of running each benchmark is
worth surfacing once:

  - `lemur sizes` — instant.
  - `cargo test --release` — 1 min on 11 threads, 10 s on 24+.
  - `cargo run vectors` — seconds.
  - `bench_verify --zero-fixture --n 1024 --reps 5` — ~20 s on 11
    threads.
  - `bench_verify --zero-fixture --n 32768 --reps 3` — ~3 min.
  - `bench_verify --zero-fixture --n 1048576 --reps 1` — ~9 GiB RAM.
  - `bench --fast` — keygen alone is `2^τ` KOTS keygens ≈ 1.8 min
    on 11 threads; full run is dominated by the seed-only
    sign rederive (1–2 min) plus aggregation (~10 min at N=8192);
    budget at least 30 min and **do not cancel before 30 min**.

The "do not cancel before 30 min" line is what I needed and didn't
have — I cancelled at 8 min because the Full Signing line had no
progress indicator. Add it.

### 3.4 Both skills assume Sage is unavailable; could say so once

Both list `parameter/lemur_param.sage` and
`chipmunk_original.sage` without noting that the typical container
won't have SageMath and the committed `.txt` summaries are the
practical source of truth. CLAUDE.md says this; the skills could
inherit one sentence.

---

## 4. What I would *not* change

  - **Step 2 KOTS+HVC blueprint** in both skills is genuinely
    useful and well-organised. Leave it.
  - **Step 7 red flags 3, 4, 5, 6, 7** in the prior-work skill are
    well-targeted to the "we beat Lemur" review use case. Keep.
  - **Notation-trap step** in the paper-analysis skill (q vs q',
    α vs σ, three different α's) is high-yield and would have
    saved me time even just on the §7.1 walk. Keep verbatim.

---

## 5. Summary of edits, by skill and section

  | Skill                  | Section           | Edit                                                  |
  | ---------------------- | ----------------- | ----------------------------------------------------- |
  | lemur-paper-analysis   | Step 1            | Add paragraph: §7.1 worked example ≠ implementation cell |
  | lemur-paper-analysis   | Step 1            | One-liner: §7.2 is the empirical-claims load-bearing section |
  | lemur-paper-analysis   | Step 3            | Add 4th category: encoding-noise size claims (Rice vs worst-case) |
  | lemur-paper-analysis   | Step 5            | Add bullet: "Same parameter cell?" — Table-2 extrapolation gotcha |
  | lemur-paper-analysis   | (new) Step 3.5    | Bench time budgets + "don't cancel before 30 min" rule |
  | lemur-prior-work-survey| Step 1            | Add note: empirical verification is over-scope; skip 5–6 |
  | lemur-prior-work-survey| Step 5            | Add paragraph: estimator-output file > Chipmunk abstract |
  | lemur-prior-work-survey| Step 7 #1         | Add the 26–39 bit rop number from `chipmunk_*_summary.txt` |
  | lemur-prior-work-survey| Step 8            | Add "When *not* to apply this skill" paragraph        |
