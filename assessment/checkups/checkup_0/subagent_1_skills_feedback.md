# Skills feedback — theory-audit lens

I was asked to audit Lemur's theoretical content (Definition 3.1, Theorem 3.1,
Lemma 3.5, Theorem 4.1 / Lemma 4.1, the "~40-bit Chipmunk" claim). I used
both `lemur-paper-analysis/SKILL.md` and `lemur-prior-work-survey/SKILL.md`.
Below is a section-by-section critique with concrete proposed edits.

---

## A. Overall verdict

- The two skills are **well above baseline**. They are written for an
  experienced reader of lattice-signature papers, not a generic LLM auditor.
  The advice on "front-load the right pages" (Step 1, `lemur-paper-analysis`)
  was correct: I went straight to §1.1 → §1.2 → §3 → §4 → Appendix D and
  did not waste time on §2 notation.
- They are **calibrated for a fast generalist survey**, not for a deep
  proof audit. About 60% of the content in `lemur-paper-analysis` is
  optimized for summarizing or comparing schemes — only Step 3 and Step 4
  speak directly to theory work. This is the right ratio for a broad
  audience, but a theory-focused user (my brief) would benefit from
  a more proof-centric branch.
- The two skills overlap on the KOTS+HVC blueprint (Step 2 of
  paper-analysis vs. Step 4 of prior-work-survey). One of them should
  be the canonical source.

---

## B. `lemur-paper-analysis/SKILL.md` — section-by-section

### B.1 What worked

- **Step 1, "Branch by goal: Audit the proof"** (lines 47-55) was
  exactly the right entry point. The four-line dependency graph
  pointed me at §3 → Theorem 3.1 → §4 → Theorem 4.1 → Lemma 3.5 with
  zero wasted reads. This is the most valuable item in the skill.

- **Step 4, "paper-internal notation traps"** (lines 121-156) saved me
  twice: once on `q` vs `q'` (the HVC vs KOTS modulus distinction is
  critical for understanding why `q' ≡ 17 mod 32` matters), and once
  on `α` vs `σ = α / √(2π)`. Without this I would have spent ten
  minutes confused on Lemma D.1's `α ≥ √2 · η_{ε_1}(Z)` versus the
  sampler's standard deviation.

- **Step 6, "what the paper does *not* prove"** (lines 184-199) is
  defensively useful but did not come up for the theory lens. It's
  more relevant when someone asks "can Lemur do X?" for `X` not in
  the threat model.

### B.2 What was missing or wrong for a theory audit

1. **No explicit map of the security proof structure.** Step 1 says
   "audit the proof — §3, Theorem 3.1, then §4 (Theorem 4.1, the
   EUF-RK reduction). Lemma 3.5 (sampleability over non-full-rank
   lattices) is the technically novel proof step." That's correct
   but minimal. A theory user would benefit from a *dependency
   diagram* of the lemmas:

   ```
   Theorem 4.1 (KOTS EUF-RK)
     ├── Lemma 4.1 (main reduction)
     │    ├── Lemma 4.2 (rank deficiency)
     │    ├── Lemma 4.3 (intersection-kernel sampling)
     │    └── Theorem 3.1 (Dual Hint-MLWE → MLWE)
     │         ├── Lemma 3.2 (PSD + smoothing)
     │         ├── Lemma 3.3 (real branch)
     │         ├── Lemma 3.4 (uniform branch)
     │         └── Lemma 3.5 (Sampleability) ← novel
     │              └── Lemma 2.6 (rank-r → full-rank sampler reduction)
     │                   └── Appendix D proof (orthonormal U trick)
     ├── Lemma D.1 (individual correctness)
     ├── Lemma D.2 (probabilistic homomorphism)
     └── Lemma D.3 (robust homomorphism)
   ```

   This would have saved me the 20-minute exercise of grepping for
   lemma forward-references.

   **Proposed edit:** Insert this diagram in Step 1, immediately after
   the existing dependency graph. Section to edit: "Step 1 —
   front-load the right pages", new sub-paragraph at the end of the
   bullet "**Audit the proof**".

2. **No mention of the `ℓ = 1` restriction.** Lemma 4.1's proof is
   only valid for `ℓ = 1`, but Theorem 4.1 is stated more generally.
   This kind of "stated more generally than proved" is a common
   source of audit findings; the skill should call it out as a
   pattern to look for. Specifically, the skill should advise:

   > **When reading Theorem 4.1 and its dependencies, check which
   > parameter regime the proof actually covers. The published
   > parameters use `ℓ = 1`; the proof is restricted to `ℓ = 1`. If
   > you see a future paper claiming general `ℓ`, verify the proof
   > generalization.**

   Section to edit: Step 3 (trap claims to scrutinize), add a new
   bullet under "Security claims" or as a fourth top-level item.

3. **No advice on reduction loss / tightness assessment.** Step 3
   talks about security claims being estimator-bound, but does not
   warn the reader about *reduction loss factors*. For Lemur this is
   the `N(Q+1)^2` factor in Lemma 4.1; for any other paper in this
   lineage there is an analogous loss. A theory user has to look at
   the reduction concrete-tightness, not just the asymptotic
   hardness assumption. **Proposed edit:** in Step 3, add a fourth
   category beside size / timing / security:

   > 4. **Reduction tightness** — multiplicative `N · Q^k` factors
   >    in the bound from EUF security to underlying lattice
   >    hardness must be absorbed by parameter selection. Lemur loses
   >    `N(Q+1)^2`; Chipmunk's analogue is similar. A claim of
   >    "λ-bit security" against `Q` queries on `N` users requires
   >    the underlying lattice problem to provide
   >    `λ + log_2(N Q^2)` bit hardness. Verify this in the
   >    parameter analysis. The paper rarely shows this calculation;
   >    if absent, it is a real (and shared-with-Dilithium) gap.

4. **No callout on the difference between Lemma 3.5 and its
   ancestors.** The skill says "Lemma 3.5 is the technically novel
   proof step", but doesn't explain *why* the standard G+G '23 [6]
   sampler doesn't suffice. The reason is that `ker(φ_H)` is a
   rank-`(k-ℓ)d` lattice in the ambient `R^k ≡ Z^{kd}` space, so
   the lattice is not full-rank. Most existing samplers (Peikert
   '10, G+G '23) only handle full-rank lattices. Lemma 2.6
   bridges via an orthonormal change-of-basis trick.

   **Proposed edit:** In Step 1 "Audit the proof" bullet, replace
   "Lemma 3.5 (sampleability over non-full-rank lattices) is the
   technically novel proof step" with:

   > Lemma 3.5 (sampleability over non-full-rank lattices) is the
   > technically novel proof step. Standard samplers (Peikert '10
   > [17], G+G '23 [6]) work only over full-rank lattices, but
   > `ker(φ_H) ⊆ R^k` has rank `(k-ℓ)d < kd` over `Z`. Lemma 2.6
   > bridges via orthonormal change of basis, and the actual
   > Gaussian-resampling-into-the-coset is the central technical
   > content. The reduction in Theorem 3.1 (Lemma 3.3, Lemma 3.4)
   > combines this sampler with Peikert's convolution Lemma 2.1 to
   > get a `8mε_2`-tight reduction to standard MLWE.

5. **The `~40-bit Chipmunk` headline number is not pinned to the
   estimator output.** Step 3 (lines 105-114) says the security
   claim rests on the lattice estimator, and the closing paragraph
   tells you to read `chipmunk_original_security_summary.txt`. But
   the skill never says *what the numbers in that file mean*. The
   columns `RSIS1 rop, RSIS2 rop, RSIS3 rop` are core-SVP
   log_2-operations values (i.e. roughly bit-security). A theory
   reader does not necessarily know that off the bat.

   **Proposed edit:** In Step 3, after the third bullet on security
   claims, add:

   > **The `chipmunk_original_security_summary.txt` columns
   > `RSIS{1,2,3} rop` are base-2 log of estimated SVP-attack
   > operation cost — i.e. bit-security in core-SVP. Chipmunk's
   > published parameters yield values in `[22.8, 39.4]`. The
   > "~40-bit" claim in Lemur's §1 is the maximum over the three
   > RSIS instances and over τ; a stricter security-as-min-over-
   > instances reading would put Chipmunk at ~23 bits. Both
   > readings support Lemur's directional claim (Chipmunk's
   > 112-bit number is overstated by ~70 bits); the precise number
   > depends on the worst-case reading.**

### B.3 What was misleading

- **Step 2's KOTS+HVC blueprint table** (lines 67-77) is duplicated
  near-verbatim in `lemur-prior-work-survey` Step 4. This is
  redundant. The two skills should pick one to be canonical.
  Recommendation: keep it in `lemur-prior-work-survey` (the survey
  skill is about placing the paper in context, so the blueprint
  belongs there). In `lemur-paper-analysis`, replace with a
  one-line cross-reference: "For the KOTS+HVC blueprint shared by
  Squirrel/Chipmunk/Lemur, see `lemur-prior-work-survey` Step 4."

- **The "ratios beat absolutes" advice in Step 3** (lines 96-105)
  is timing-specific and does not transfer to proof audits. This
  is a non-issue for the theory lens but it does take up real
  estate in Step 3. If you want to keep the skill compact for a
  theory user, the timing section could be moved to a sub-skill
  or appendix.

### B.4 What I would add as new content

A new **Step 8** titled "When auditing a security proof", with the
following structure:

> 1. Locate the **main theorem** (typically the EUF/UF reduction).
>    Note its preconditions: which underlying assumptions does it
>    invoke? Find their formal statements (typically in §2 / §3).
> 2. Build the **lemma dependency graph** for the main theorem.
>    For Lemur, the graph is given above.
> 3. For each lemma, verify:
>    - **Preconditions** are listed in the theorem statement.
>    - **Computability / PPT** is asserted (or, ideally, derived).
>    - **Where the lemma is used in the main proof**.
> 4. Identify **the load-bearing novel step**. For Lemur this is
>    Lemma 3.5 (and behind it, Lemma 2.6's orthonormal change of
>    basis).
> 5. **Check parameter regime restrictions.** A theorem stated
>    generally may have a proof restricted to a specific regime
>    (e.g. `ℓ = 1`). The parameter selection in §7 should use
>    only the *proved* regime.
> 6. **Estimate reduction loss.** Multiply through any guessing
>    factors (target user index, RO query position, etc.). Report
>    the loss explicitly. A 128-bit signature scheme with a
>    `2^{140}` loss to the underlying problem needs the underlying
>    problem to be ~268-bit hard.
> 7. **Estimate statistical slack.** Hybrid arguments accumulate
>    `ε` terms; sum them and check `8mε_2 < 2^{-λ}`.
> 8. **Sanity-check the assumption itself.** Is the assumption new?
>    If so, is there a reduction to a standard assumption? For
>    Lemur, Dual Hint-MLWE has a tight reduction to MLWE in
>    Theorem 3.1 — this is what gives confidence.

This would have shortened my audit from "read in linear order and
take notes" to "fill in the eight bullets".

---

## C. `lemur-prior-work-survey/SKILL.md` — section-by-section

### C.1 What worked

- **Step 1, "triage the reference list into four buckets"** (lines
  32-62) is the most useful single block in either skill. For my
  brief (theory lens), the relevant buckets were [11] Kim et al.
  Hint-MLWE (theoretical lever), [8] Chipmunk + [9] Squirrel +
  [13] LM '08 (direct lineage), and [6, 14, 16, 17] (foundational
  tools). The skill correctly identified [11] as the single most
  important paper to read in depth.

- **Step 2, "find the body-text use of each citation"** (lines
  65-93) gave me the right trick — find where [11] appears in the
  body. The instruction `grep -nE '\[N\]'` is concretely usable.
  When I followed it, I found that [11] appears *only* in §3 and
  §1.2, both of which set up the assumption — confirming it is a
  theoretical lever, not a tool.

### C.2 What was missing for a theory audit

1. **No advice on how to read a "predecessor assumption" paper
   under time pressure.** The skill says "fetch the abstracts of
   the three direct-lineage / theoretical-lever papers". For [11]
   that's Kim et al. CRYPTO '23, eprint 2023/623. The skill notes
   ePrint returns 403 to direct fetch — true and unhelpful. For
   a theory lens, the *specific* content one needs from the
   predecessor paper is:
   - The exact formal definition of the assumption (Hint-MLWE).
   - The form of the reduction to MLWE (loss factor, statistical
     slack, parameter regime).
   - Any restrictions (e.g. number of hints, hint width).

   The current skill does not say what to extract; it implies the
   abstract is sufficient, but for a theory audit the abstract is
   not enough. **Proposed edit:** in Step 5, expand the bullet on
   [11] to say:

   > **Hint-MLWE '23 [11]** — eprint 2023/623. For a theory
   > audit, you need: (a) the formal definition (find Definition
   > 1 or 3.1 in the original paper); (b) the reduction's loss
   > factor (typically an exponential or polynomial loss in the
   > number-of-hints `Q`); (c) the parameter regime
   > restrictions on hint width. If ePrint is blocked, check
   > IACR archives, Semantic Scholar paper pages (often have
   > full-text or extracted abstract+intro), or the authors'
   > university page. Failing that, work from what the *current*
   > paper says about the assumption — Lemur's §1.2 lines
   > 182-188 give a 3-sentence summary that is enough to compare
   > to Dual Hint-MLWE.

2. **No structural map of what changes between Hint-MLWE and Dual
   Hint-MLWE.** The skill notes the difference exists (Step 5,
   line 169-171: "Lemur generalizes to *Dual* Hint-MLWE
   (noise-free hints, secret on opposite sides of public-key vs
   signature equation)") but does not enumerate. A theory user
   needs:
   - Hint form: noisy (Hint-MLWE) vs noise-free (Dual).
   - Secret placement: one side (Hint-MLWE) vs two sides (Dual,
     T = SA *and* Z = HS).
   - Reduction granularity: Hint-MLWE → MLWE has known loss
     scaling with hint count and hint width; Dual Hint-MLWE →
     MLWE has 8mε_2 loss with no multiplicative term.

   **Proposed edit:** Add a comparison table in Step 5 immediately
   after the Hint-MLWE bullet:

   | Property | Hint-MLWE [11] | Dual Hint-MLWE (Lemur §3) |
   |----------|----------------|----------------------------|
   | Hint form | `z_i = c_i·s + y_i` (noisy) | `HS` (noise-free) |
   | Number of hints | `Q` (bounded) | `m` (one per column of `S`) |
   | Secret placement | `s` on one side | `S` on two sides (`SA` and `HS`) |
   | Reduction to MLWE | Loss scales with hint width and `Q` | `8mε_2` statistical only, no mult. loss |
   | Smoothing precondition | hint width ≥ `√Σ` | `α^2 ≥ α_0^2‖B‖_2^2 + (1+ℓα_H)·L` |

### C.3 What was misleading or redundant

- **Steps 4 and 7 overlap on "red flags for comparison claims".**
  Step 4 talks about the four building blocks; Step 7 lists red
  flags when reviewing a "we beat Lemur/Chipmunk" claim. Both
  mention same-security-level, same-N, same-coding. Recommendation:
  consolidate red-flag content in Step 7 only.

- **Step 6's coverage of BLS / Drake et al. / Boneh-Kim** is correct
  but generalist. A theory-lens user does not care about deployment
  numbers; they care about whether the *assumptions* used in those
  papers are reducible to Lemur's. A theory-flavor sub-section
  would be:

  > **For theory users:** the assumptions used by each paradigm
  > are not reducible to each other. BLS rests on co-CDH in
  > pairing groups; Drake et al. on CRHF + pqSNARK arguments;
  > Boneh-Kim on Module-SIS + interactive setups. So there is no
  > "shared assumption baseline" for comparing post-quantum
  > multi-sigs theoretically — the comparison is necessarily
  > on different threat models. This is worth flagging in any
  > "X beats Lemur" claim.

### C.4 What I would add as new content

A new **Step 9** titled "When the theoretical lever is novel":

> If a paper introduces a *new* assumption (as Lemur does with
> Dual Hint-MLWE), the credibility test is:
>
> 1. Is there a **formal reduction to a standard assumption**? If
>    yes, what is the loss factor? (Lemur: Theorem 3.1 → standard
>    MLWE, `8mε_2` statistical only.)
> 2. What does the **predecessor assumption** [11] look like? Why
>    doesn't the predecessor suffice for the current paper? (Lemur:
>    Hint-MLWE has noisy hints, Lemur needs noise-free; Hint-MLWE
>    has one-sided secret, Lemur is two-sided.)
> 3. Is the **parameter regime** in the assumption matched by the
>    parameter selection in §7? (Lemur: `α ≥ 60.93` from
>    Constraint 12, `α = 61` selected — matches.)
> 4. Is there any **subtler reduction loss** (multi-user, RO
>    programming)? (Lemur: `N(Q+1)^2` in Lemma 4.1 — not absorbed
>    in parameter selection, see §B.4 above.)

---

## D. Specific proposed edits, compactly

| Skill | Section | Edit |
|-------|---------|------|
| `lemur-paper-analysis` | Step 1, "Audit the proof" bullet | Add the lemma-dependency diagram for Theorem 4.1 ← Lemma 4.1 ← Theorem 3.1 ← Lemma 3.5 ← Lemma 2.6. |
| `lemur-paper-analysis` | Step 1, "Audit the proof" bullet | Expand the Lemma 3.5 sentence to explain *why* it is novel: standard samplers handle only full-rank lattices, but `ker(φ_H)` is rank-`(k-ℓ)d` in `Z^{kd}`. |
| `lemur-paper-analysis` | Step 2, KOTS+HVC table | Delete (redundant with `lemur-prior-work-survey` Step 4); replace with one-line cross-reference. |
| `lemur-paper-analysis` | Step 3 | Add a fourth category "Reduction tightness" with concrete advice on multiplying through `N · Q^2` factors. |
| `lemur-paper-analysis` | Step 3 | Add a paragraph explaining what `RSIS rop` columns mean in `chipmunk_original_security_summary.txt`. |
| `lemur-paper-analysis` | New Step 8 | "When auditing a security proof" — 8-bullet checklist (see §B.4 above). |
| `lemur-prior-work-survey` | Step 5, [11] bullet | Expand to say what specific items a theory-lens reader needs from the predecessor paper (formal def, reduction loss, parameter regime). |
| `lemur-prior-work-survey` | Step 5 | Add the Hint-MLWE vs Dual Hint-MLWE comparison table (see §C.2 above). |
| `lemur-prior-work-survey` | Step 7 | Consolidate red-flag content here; remove duplication with Step 4. |
| `lemur-prior-work-survey` | New Step 9 | "When the theoretical lever is novel" — 4-bullet credibility test (see §C.4 above). |
| Both | "Files this skill assumes" | Both skills already point at `chipmunk_original_security_summary.txt`. Add a one-line note that columns named `RSIS{1,2,3} rop` are base-2 log of estimated SVP-attack cost. |

---

## E. What I did *not* find useful

- **`lemur-paper-analysis` Step 5** ("when comparing schemes within
  the paradigm") was not relevant to a single-paper theory audit.
  It's relevant if a reviewer is comparing two papers.

- **`lemur-paper-analysis` Step 7** ("when summarizing for someone
  else") is also reviewer/summarizer-oriented, not auditor.

- **`lemur-prior-work-survey` Step 3** (paradigm map for PQ
  multi-sigs) is well-drawn but a theory user already knows the
  landscape. A more concise version would suffice.

None of these are wrong; they just aren't the load-bearing content
for the theory lens. Refactoring the skill into "general reading"
+ "summarizer branch" + "theory-audit branch" would let each user
focus.

---

## F. Single most important addition

If I could only make one change to the skills, it would be in
`lemur-paper-analysis` Step 1: **the lemma dependency diagram
for Theorem 4.1's proof**. That single image would let any
future theory-lens reader build a mental model of the proof
in 30 seconds, instead of grepping for 20 minutes. It is the
*one* artifact that would have saved me the most time and is
not currently in either skill.
