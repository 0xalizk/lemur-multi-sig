# Lemur — theoretical / proof-side audit

Artifact: `/workspace/repo/report.pdf` (24 pp). Cross-references: line numbers
below refer to `/workspace/tmp/lemur.txt` (the layout-preserved text dump of
the PDF). Page numbers come from the running header in that dump.

The lens is assumptions, reductions, hybrids, and concrete tightness. Code
and benchmarks are out of scope here.

---

## 1. Definition 3.1 (Dual Hint-MLWE) — well-formed, but a comparison to
       Hint-MLWE [11] is missing and the parameter regime is restrictive

**Statement (paraphrased, p. 5, lines 390-413).** Fix
`q', d, m, n, k, ℓ ∈ N+`, distributions `χ` over R (for the secret) and `C`
over R (for the off-diagonal block of the hint). Sample
`A = [I_n; A_2] ∈ R_{q'}^{m×n}` with `A_2` uniform, `S ← χ^{k×m}`, and
`H = [I_ℓ | H'] ∈ R^{ℓ×k}` with `H' ← C^{ℓ×(k-ℓ)}`. The challenge is to
distinguish

  D_0 = (A, SA, H, HS)   vs.   D_1 = (A, (S+U)A, H, HS)

where each column `u_i ← ker(φ_{H,q'})`, independently uniform.

### 1.1 What is well-formed

- The two distributions agree on `A`, `H`, and the noise-free hint `HS`. The
  game is *exactly* the question "given the leakage `HS`, can one tell
  whether `SA` is the real LWE image or has been rerandomized by an element
  of the (image of the) kernel of `H` under `A`?". That is genuinely
  parallel to the Hint-MLWE intuition.

- The use of the systematic form `H = [I_ℓ | H']` and `A = [I_n; A_2]`
  pins down the basis representation needed in the kernel sampleability
  proof (Lemma 2.9 gives the explicit kernel basis `B = [-H'; I_{k-ℓ}]`),
  so the definition is computable.

### 1.2 What distinguishes it from Kim et al. Hint-MLWE [11]

The paper claims, p. 3, lines 182-188, two differences:

1. Lemur leakage is **noise-free** `HS`; Hint-MLWE leakage is **noisy**
   `z_i = c_i·s + y_i` for short `y_i`.
2. In Lemur the secret sits on **opposite sides** of the public-key
   equation `T = SA` and the signature equation `Z = HS`; in Kim et al.
   the secret sits on one side throughout.

Both differences are real. (1) is the more consequential one — noise-free
hints leak strictly more about `S` than noisy ones, and standard
Hint-MLWE reductions degrade by a factor that explodes as the hint
noise shrinks toward zero (see, in particular, Lemma A.4 / Lemma A.5,
which are the discrete-Gaussian smoothing bounds the paper invokes
through Peikert [17] / G+G [6]).

### 1.3 What is **not** discussed and ought to be

- **No direct reduction from Hint-MLWE to Dual Hint-MLWE.** Theorem 3.1
  reduces directly to *standard* MLWE; Hint-MLWE is cited only as
  motivation. That is actually a *strength* (Dual Hint-MLWE inherits
  no statistical-distance slack from Kim et al.'s reduction), but the
  paper would be cleaner if it stated this explicitly. As written, a
  casual reader might assume Hint-MLWE is being invoked anywhere in
  the security proof — it is not.
- **Parameter regime is narrower than Hint-MLWE.** The Lemur reduction
  needs `α^2 ≥ α_0^2 ‖B‖_2^2 + (1+ℓα_H) ln(2d(k-ℓ)(1+1/ε_2))/π`
  (eq. (4)–(5), lines 423-431). This is a strictly stronger condition on
  `α` than Kim et al.'s analogue (whose noise-floor scaling depends on
  the noisy-hint noise width). The paper never quantifies how much
  larger `α` has to be in Dual Hint-MLWE than in Hint-MLWE for a fixed
  underlying-MLWE level — concretely the paper picks `α = 61` for
  `d = 128, k = 5, α_H = 31`, where Hint-MLWE would let one go lower.
- **No discussion of `C = T_{α_H}`'s structure.** The paper instantiates
  `C = T_{α_H}` (the ternary, weight-α_H polynomials, p. 5, line 420).
  The reduction proof never uses anything beyond an upper bound on
  `‖B‖_2` (numerically `≈ 33.36`, line 965). It would be honest to
  flag that the result *qualitatively* holds for other `C`, but the
  numerical bound is `T_{α_H}`-specific.

### 1.4 Suspicion: the "dual" framing makes the leakage stronger, not weaker

This is worth flagging explicitly. In Kim et al. [11], `s` is on one side
of `As + e`; the hints `c_i·s + y_i` are noisy linear combinations. In
Lemur, `T = SA` reveals `S` linearly through `A` (modular linear), and
`HS` reveals `S` linearly through `H` (no modular reduction, exact).
The combination `(SA, HS)` over the **same** `S` with **two different**
mixing matrices (`A` on the right, `H` on the left) is, structurally,
a strictly richer leakage profile than Kim et al.'s. The hardness claim
therefore relies on a structural argument: `H` is short and `A` is
uniform, so projecting `S` through `H` leaks the small-norm directions
while projecting through `A` is uniformly mixed. The reduction works
because the rerandomization mask `U` lives in `ker(φ_{H,q'})` — i.e.
exactly the directions of `S` that `HS` *cannot* see. This is the
correct intuition, but the paper would benefit from one paragraph
spelling it out.

---

## 2. Theorem 3.1 — sequence-of-hybrids reduction, with a `8mε_2` loss

**Statement (p. 5, lines 423-434).** Under stated norm-bound conditions on
`α, α_0, ‖B‖_2`,

  Adv_DualHintMLWE_{q',m,n,k,ℓ,α,T_{α_H}}(A) ≤ Adv_MLWE_{q',m,n,k-ℓ,α_0}(B) + 8mε_2.

### 2.1 Hybrid structure (good)

The proof, lines 436-483, uses three games:

- G_0: real (A, SA, H, HS).
- G_1: replace `s_i` by a resample `s̃_i ← D_{ker(φ_H)+s_i, α}`. This is
  *identical in distribution* to G_0 by the coset-resampling Lemma 2.7.
- G_2: replace `SA` by `(S+U)A` with `u_i ← ker(φ_{H,q'})` uniform. This
  is the Dual Hint-MLWE side-distribution.

The reduction B receives an MLWE_{q',m,n,k-ℓ,α_0} instance `(A, T')`,
samples `S ← D_{R^k×m, α}`, samples `r_i ← D_{ker(φ_H)+s_i, √Σ_1}` using
Lemma 3.5, sets `T := B T' + R A` and outputs `(A, T, H, HS)`.

- If `T' = S' A`, the convolution-of-Gaussians argument (Lemma 3.3 +
  Lemma 2.1 via Peikert [17] Theorem 3.1) shows `T` is `8mε_2`-close
  to G_1.
- If `T'` is uniform, Lemma 3.4 shows `T` is *exactly* distributed as
  G_2.

The composition is tight: one Hint-MLWE-style hybrid → uniform.

### 2.2 Loss factor (mild, but worth pinning down)

- Statistical slack: `8mε_2`. With `m = 20` and `ε_2 = 2^{-136}`
  (the paper's choice, line 965), this is `≈ 2^{-130}` — well below
  `2^{-λ} = 2^{-128}` so the slack is fine.
- **Multiplicative loss: none.** This is genuinely a tight reduction
  in the cryptographic sense — no `Q` factor, no `N` factor, no
  `1/Q^2` factor; advantage in MLWE is at least advantage in
  Dual Hint-MLWE minus a statistical slack term.

This is the rare best-case scenario for a hybrid reduction. The
multi-user / EUF-RK losses kick in only in Theorem 4.1 (Lemma 4.1).

### 2.3 What's hand-wavy in the proof

- **Lemma 3.2 (line 485)** asserts `Σ_1 = α^2 I_k − α_0^2 B B^⊤` is
  PSD and `Σ_1 ⪰ η_{ε_2}(ker(φ_H))^2`. The proof bounds `λ_min(Σ_1) =
  α^2 − α_0^2 ‖B‖_2^2` (correct by Rayleigh quotient), then upper-bounds
  the smoothing parameter via Lemma 2.4. But Lemma 2.4 needs a **basis**
  whose columns are the generators of the lattice; the paper uses
  `B = [-H'; I_{k-ℓ}]` and bounds `max‖b_i‖_2 = √(1 + ℓα_H)` (eq. 5).
  This is OK because every column of `B` has at most one `±1` from
  the identity part and one column of `H'` (each entry of `H'` is in
  `T_{α_H}`, so ℓ_2 norm of that column ≤ √(ℓα_H)). The reasoning is
  correct but the paper just writes `√(1 + ℓα_H)` without justification.
  Reviewers would want a single sentence here.

- **`α ≥ √2 · η_{ε_1}(Z)` in Lemma D.1 (line 2106)** is stated as a
  precondition without proof. Standard, and the constant is right
  for the discrete-Gaussian regime, but it is shadowed by a *second*
  bound on `α` coming from Lemma 4.2 (`α ≥ p √(2α_H ln(2(r-1)d(1+1/ε_3))/π)`,
  line 740). The paper has *three* constraints on `α` (Constraints 5,
  12, 15 in Table 4) and resolves them by taking the max numerically
  (`α = 61`, line 976). Concretely fine; theoretically opaque.

- **Lemma 3.5 (Sampleability) — see §3 below**, which is genuinely novel.

---

## 3. Lemma 3.5 (Sampleability over non-full-rank lattices) — adapted, not
       invented, but the reduction is non-trivial

**Statement (p. 6, lines 514-534).** Given a basis `B = [-H'; I_{k-ℓ}] ∈
R^{k×(k-ℓ)}` of `Λ = ker(φ_H)` (which has Z-rank `n = (k-ℓ)d`, a
non-full-rank lattice in `R^k`), and parameters satisfying
`α^2 ≥ α_0^2 ‖B‖_2^2 + (1+ℓα_H) ln(2(k-ℓ)d+4)/π`, there is a PPT algorithm
to sample `r_i ← D_{Λ+s_i, √Σ_1}` for any `s_i ∈ R^k`.

### 3.1 Provenance

The lemma reduces to **G+G '23 Lemma 4 (= Lemma 2.5 in the paper,
line 270)**, which is a *full-rank* Gaussian sampler. The novelty of
Lemma 3.5 is that the lattice `ker(φ_H)` is rank `(k-ℓ)d < kd = n`
in `R^k = Z^{kd}` (the ambient space), so the G+G sampler does not
directly apply.

The paper bridges via **Lemma 2.6** (line 286, "Reduction to full-rank
sampler"), whose proof is deferred to Appendix D (lines 1981-2101).
This appendix gives an orthonormal change of basis `U` from the
rank-`r` subspace, projects the center `c` onto the span, samples in
the smaller full-rank lattice using G+G, and unprojects. The proof is
clean: equations (47)-(53) and (54)-(57) verify the resulting
distribution is exactly `D_{Λ_0+c_0, √Σ_0}`.

### 3.2 Is it novel?

- The **technique** (Gram-Schmidt orthogonalization + sample in the
  span + add the orthogonal-complement offset) is folklore; e.g.
  Micciancio-Regev '04 (the ancestor of Lemma 2.3) and Peikert '10
  (Lemma 2.1, the convolution lemma) use the same toolbox.
- The **packaging** as a clean black-box "given a full-rank sampler,
  one can sample over any rank-`r` sublattice" is new in this lineage.
  The paper is honest about this: line 280, "the lemma below the
  extension to the general rank-`r` case", and explicitly defers proof
  to Appendix D.
- I have not seen this exact statement in print elsewhere in the
  lattice-multi-sig literature; it is plausibly novel, though
  borderline-folklore. Calling it "may be of independent interest"
  (line 188) is well-calibrated.

### 3.3 Where it could be tighter

- The verification condition `η(r) · max_i ‖Σ_0^{-1/2} b_{0,i}‖_2 ≤ 1`
  (eq. 3, line 299) ties together the rank-`r` smoothing parameter
  with the rescaled basis. The paper picks `η(r) = √(ln(2n+4)/π)`
  (line 528). This is the tail-cut constant from G+G; using the
  tighter Brakerski-Langlois-Peikert-Regev-Stehlé style bound would
  shave logarithmic factors. Probably not worth the complication for
  the paper's headline numbers (`α = 61`, generous margin).
- The PPT time bound is asserted but not analyzed. Standard.

### 3.4 Verdict

Lemma 3.5 is **the genuinely novel proof step in the paper**, even
though it is built on standard ingredients. The statement is correct
and the reduction in Lemma 2.6 / Appendix D is rigorous. A careful
reviewer would still verify that the orthonormal `U` is computable in
poly time over `R` (it is, by Gram-Schmidt over the column embedding,
but the paper just asserts it line 1989), and that the rescaling
`x_0 := S_0 · x` preserves the Gaussian (which the equations (54)-(57)
do show, but with somewhat informal `α'` normalisation).

---

## 4. Theorem 4.1 / Lemma 4.1 (KOTS EUF-RK) — handles multi-user, but the
       reduction loss is dominated by `N(Q+1)^2`

**Statement (Theorem 4.1, p. 7, lines 561-582 + Lemma 4.1, p. 7-10,
lines 619-732).** Under Dual Hint-MLWE + MSIS in ROM,

  Adv_EUF-RK_{KOTS}(A) ≤ N(Q+1)^2 · Adv_DualHintMLWE(B) + Adv_MSIS(D)
                          + Q_H^2 / |T_{α_H}|^{k-2ℓ-1} + negl(λ),

where `Q = Q_H + N` and `Q_H` is the number of RO queries.

### 4.1 Multi-user handling — present but lossy

The reduction handles multi-user via two layers of guessing in G_2
(p. 8, lines 615-617, Claim 4.2):

- Guess the target user `i_0 ← [N]` with probability `1/N`.
- Guess the hash-query position `j_1` for the queried signing message
  and `j_2` for the forged message, with probability `1/(Q+1)^2`.

Multiplying: `p_2 ≥ p_1 / (N(Q+1)^2)`.

This is **the standard Forking / Coron-style reduction** for KOTS in
multi-user. It is rigorous but loose:

- `N` factor is intrinsic (no other way to embed an MLWE/MSIS challenge
  into one of `N` keys without telling the adversary which).
- `(Q+1)^2` is the price of two query-position guesses; could in
  principle be brought to `(Q+1)` if the construction supported
  programming the challenge at a single, adversary-chosen position
  (e.g. via a re-randomization technique like Boyen). Lemur's KOTS
  signs a single message per user, so the second `(Q+1)` is genuine.

### 4.2 Concrete loss

For `N = 2^{20}`, `Q = 2^{60}` (a generous RO bound), the prefactor is
`≈ 2^{20} · 2^{120} = 2^{140}`. To absorb a `Q^2`-loss and still
deliver `128-bit` security against `2^{60} · 2^{20}` queries you need
the underlying Dual Hint-MLWE assumption to provide *at least* `268-bit`
hardness in the asymptotic sense. The paper's parameter selection
implicitly assumes the underlying problem is, well, hard at
`λ = 128` — but the reduction is far from tight, and the paper does
**not** discuss whether the picked parameters actually compensate for
the `N(Q+1)^2` loss. This is a real omission.

The Chipmunk paper had the same loss profile, but Chipmunk's other
parameters were also off by ~70 bits per Lemur's own recomputation,
so the question is more pressing here: does the corrected estimator
output for Lemur's parameters give the ~270 bits of "underlying"
Dual Hint-MLWE hardness needed to absorb the reduction loss? The
`summary.txt` numbers (`RHF_LWE_KOTS ≈ 1.0042`) translate to
core-SVP bit security in the range `120-130` for the *primary* MLWE
instance, **not** for any reduction-loss-adjusted level.

### 4.3 Game G_3 hop: Dual Hint-MLWE switch (clean)

Claim 4.3 (line 631) switches `T = SA` for `T = (S+U)A` with
`u_i ← ker(φ_{[H_1; H_2], q'})` via a single Dual Hint-MLWE invocation.
The intersection-kernel sampling (Lemma 4.3, lines 802-841) gives an
explicit basis `B_{q'}` for `ker(φ_{[H_1;H_2], q'})` via an explicit
formula. Provided `X = H_1^{(1)} - H_2^{(1)}` is invertible in `R_{q'}`
(which Lemma A.2 supplies for the parameter choice `q' ≡ 17 mod 32`,
`t_2 = 8`), the construction is correct.

The `negl(λ)` in Claim 4.3 collects the abort probability from
Claim 4.1 (`Q^2 / |T_{α_H}|^{r-1}` term, fine).

### 4.4 MSIS extraction in Claim 4.4 (line 645) — the load-bearing step

Once T = (S+U)A and the forgery `(Z*, w*)` arrives, the extractor
forms `X := Z* - w* H_2 S`. Then `XA = 0 mod q'` follows from
`H_2 U = 0 mod q'` plus `Z* A = H_2 (w* T) mod q'` (the verification
equation for the forgery). The norm bound `‖X‖_max ≤ 2β_σ + 2α_w β_z`
gives the MSIS solution width.

The non-trivial step is to show `X ≠ 0` with high probability. The
paper's argument (lines 676-720) is:

- Partition `S = [S^{(0)}; S^{(1)}; S^{(2)}]` with `S^{(2)} ∈ R^{(k-2ℓ)×m}`.
- Argue that the last `k-2ℓ` rows of `U` (from the Lemma-4.3 intersection
  kernel) are uniform over `R_{q'}^{(k-2ℓ)}` and hence statistically
  mask `S^{(2)}` given `T`.
- Lemma 4.2 then bounds the conditional probability that
  `H_2^{(2)} S^{(2)} = (w*)^{-1} Z* - H_2^{(0,1)} S^{(0,1)}` by
  `((1+ε_3)/(1-ε_3))^{2m} · p^{-dm/t_1}`.

This argument uses the `Lemma A.2` invertibility result twice — once
for `q'` (to make `w*` a unit), once for `p` (to make
`H_1^{(1)}, H_2^{(1)}` invertible). The `r ≥ 2ℓ` requirement comes
from needing `H^{(2)}` to have rank-deficiency-free image (the
`Im(ψ')` cardinality argument, lines 740-777, which lower-bounds the
image cardinality by `p^{d/t_1}`).

**This is a careful, well-grounded MSIS extraction.** No obvious holes.

### 4.5 Subtler issues

1. **The `ℓ = 1` requirement (line 545, line 620).** Lemma 4.1 is proved
   only for `ℓ = 1`. The paper says "all our parameter settings have
   `ℓ = 1`" — true, but it means Theorem 4.1 as stated for general `ℓ`
   does not have a proof for `ℓ > 1`. Pragmatically OK; theoretically a
   loose end. A clearer statement of Theorem 4.1 would simply restrict
   to `ℓ = 1`.

2. **The role of `r = k - 2ℓ` in Constraint 14, `|T_{α_H}|^{r-1} ≥ 2^{2λ}`
   (line 909).** For `k = 5, ℓ = 1, r = 3`, this requires
   `|T_{α_H}|^2 ≥ 2^{256}`, i.e. `|T_{α_H}| ≥ 2^{128}`. Counting:
   `|T_{α_H}| = (d choose α_H) · 2^{α_H}`. For `d = 128, α_H = 31`:
   log_2 `|T_31| = log_2 (C(128, 31)) + 31 ≈ 73 + 31 = 104`. **104 ≱ 128**.
   So `|T_{α_H}|^2 ≈ 2^{208} ≥ 2^{256}` would need `|T_{α_H}| ≥ 2^{128}`,
   which **fails** at `(d, α_H) = (128, 31)`. The paper says
   Constraint 14 is satisfied; check it numerically. Either I'm
   misreading the symbols (likely α_H is the *count* of nonzero coefs
   and the bound is approximate) or the constraint is in fact tight
   and the paper's parameter generator has a subtle margin. This
   deserves a sanity check against the parameter script.

3. **Robust homomorphism for KOTS (Lemma D.3, line 2092).** The proof
   is one line and correct, but only because the construction outputs
   linear `Z = HS`. The robust homomorphism for HVC (Definition B.4,
   Lemma B.6 / Lemma B.7) is more involved because the HVC opening
   includes the projection/decomposition machinery.

---

## 5. The "Chipmunk ≈ 40-bit secure" claim — well-supported by the
       committed estimator output, but the claim should be sharpened

### 5.1 What the evidence says

`/workspace/repo/code/parameter/chipmunk_original_security_summary.txt`
(25 rows) reports `RSIS rop` columns ranging from **22.8 to 39.4** for
Chipmunk's published parameter sets (both `secpar=112` rows, lines 3-14,
and `secpar=128` rows, lines 15-26).

These are core-SVP bit-security estimates (from `SIS.estimate.rough(..)`
in the lattice-estimator, see `chipmunk_original.sage` lines 28-40).
The lowest values are around **22.8 bits** (the `secpar=112, τ=21,
ρ=131072` family); the highest is **39.4 bits** (the
`secpar=112, τ=21, ρ=1024` family). For the *128-bit-claimed* parameter
sets specifically, the max is **38.8 bits** (lines 15, 18, 21, 24) and
the min is **23.1 bits** (lines 17, 20, 23, 26).

### 5.2 Is the "~40-bit" headline well-calibrated?

- The paper says "approximately 40-bit instead of the claimed 112-bit"
  (line 120). Strictly, the estimator gives **~23 to 39 bits**
  depending on which RSIS instance is the bottleneck. The headline
  number is the **maximum** of the three RSIS instances. That is the
  honest summary if you assume the security level is the *easiest*
  attack on the *hardest* instance — but a more conservative reading
  (security = min over instances) would put Chipmunk at **~23 bits**,
  which is even worse.
- Per the contract here: the "~40-bit" claim is **mildly optimistic
  on Chipmunk's behalf** — the data actually says "between 23 and 39
  bits depending on which instance and which τ". The paper takes
  the most charitable read.
- This is a fair criticism of Chipmunk and the conclusion is correct
  in direction and rough magnitude. The 70-bit gap between claimed
  and recomputed is real.

### 5.3 Why the gap exists

Chipmunk's `get_root_hermite_factor(secpar)` is a hand-tuned
table — RHF=1.005 for `secpar=112`, RHF=1.004 for `secpar=128`
(`chipmunk_original.sage` lines 134-140). This is the *2015-era*
heuristic. Modern Dilithium / APS-style estimators do not work this
way; they compute the BKZ block size achieving a target root Hermite
factor, then convert via core-SVP. Chipmunk's heuristic
overestimates the achievable RHF for the given dimension.

This critique is fair, well-supported, and reproducible from the
committed sage scripts. It would be even more persuasive if the
paper included a one-paragraph technical explanation of *why*
RHF=1.005 underestimates the attack cost — currently the paper
just asserts the discrepancy.

### 5.4 What this implies for Lemur's own security claim

- Lemur uses the lattice-estimator (APS '15 [1]) for MLWE and the
  Dilithium framework for MSIS — both modern. `summary.txt` reports
  `RHF_LWE_KOTS ∈ [1.0039, 1.0042]` and `RHF_SIS_* ≤ 1.0045` across
  the parameter cells. Working back to bit security via the standard
  conversion: RHF=1.0042 ⇔ ~128-bit core-SVP. So Lemur's `α = 61`,
  `q' ≈ 1.2-54 × 10^9` choice is at least *internally consistent* at
  the 128-bit core-SVP level for the underlying MLWE/MSIS problems.
- **But** see §4.2: the reduction loss factor of `N(Q+1)^2` is not
  absorbed in this 128-bit level. The paper's effective EUF-RK
  security level, accounting for the reduction loss, is closer to
  `128 − log_2(N(Q+1)^2)`. At `N=2^{20}, Q=2^{60}`:
  `128 − 20 − 120 = -12 bits`. To have *meaningful* asymptotic
  security guarantees one needs the underlying lattice problems to
  be hard at the `λ + log_2(N(Q+1)^2)` level. This is a tension that
  the paper does not address.

  (In practice, the consensus in the post-quantum signature
  community is to live with this loss because the reduction is loose
  and concrete attacks do not match the reduction; the same critique
  applies to Dilithium. But the paper should at least mention it.)

---

## 6. EUF (multi-sig) → EUF-RK (KOTS) composition — present but proof
       sketch is borrowed from [8] Chipmunk

The Lemur EUF reduction (Theorem C.1, line 1886, Lemma C.3, line 1962)
distinguishes two cases:

- Case 1: HVC binding break → reduction to position-binding [8].
- Case 2: KOTS forgery → reduction to MU-EUF-RK [Lemma 4.1].

Adv_EUF_Lemur ≤ Adv_binding_HVC + Adv_MU-EUF-RK_KOTS.

The bound is additive, no further multi-user loss at this layer (the
multi-user loss is already absorbed in Lemma 4.1).

**Issue:** Lemma C.3 says (line 1981) "The proof follows the same idea
as in [8]. For completeness, we still draft the proof idea in here."
This is the *only* proof of the EUF composition for Lemur, and it is
a sketch borrowed from Chipmunk. Reviewers would want either a more
careful argument or an explicit citation to the formal Chipmunk proof
being inherited. The HVC position-binding reduction (Lemma B.7, line
1774) is proved separately, which is good.

---

## 7. Suspect, hand-wavy, or under-justified items (summary)

| # | Location | Issue | Severity |
|---|----------|-------|----------|
| 1 | Def 3.1 (line 390) | No formal comparison to Hint-MLWE [11]; the paper's narrative claim that the problem is "new" should be supported with a one-line lemma "Hint-MLWE → Dual Hint-MLWE is open" or similar. | Low |
| 2 | Thm 3.1 (line 423) | Lemma 3.2's column-norm bound `‖b_i‖_2 ≤ √(1+ℓα_H)` is asserted without proof. | Low |
| 3 | Lemma 3.5 (line 528) | Proof relies on Lemma 2.6 in App. D, which uses orthonormal `U`. The computability of `U` in PPT over `R` is asserted, not derived. | Low |
| 4 | Lemma 4.1, ℓ=1 restriction (line 620) | Thm 4.1 statement is for general ℓ; proof is only for ℓ=1. Statement should be sharpened. | Medium |
| 5 | Lemma 4.1 (line 640) | `N(Q+1)^2` reduction loss is not absorbed in parameter selection; the paper provides 128-bit MLWE/MSIS hardness but the EUF-RK level is reduction-loss-adjusted. | Medium-High |
| 6 | Constraint 14 (line 909) | `|T_{α_H}|^{r-1} ≥ 2^{2λ}` with `(d, α_H, r) = (128, 31, 3)` gives `≈ 2^{208}`, which clears `2^{256}` only marginally; should double-check parameter generator margin. | Low (verify) |
| 7 | "~40-bit Chipmunk" (line 120) | Data actually says 23-39 bits depending on instance/τ; the paper uses the most charitable read. The narrative is correct but the precise number deserves a footnote. | Low |
| 8 | Lemma C.3 (line 1981) | Proof of Lemur EUF composition is "same idea as in [8]" sketch only. Borrowed proof. | Medium |
| 9 | Robust homomorphism (Lemma D.3) | One-line proof — correct but should explicitly note that this is *only* because KOTS Sign is purely linear. | Low |
| 10 | KOTS verification has *no* rejection sampling | The construction `Z = HS` is fully linear and does not include the Lyubashevsky '12 rejection-sampling step. This is intentional (and is what enables Dual Hint-MLWE), but it means classical Fiat-Shamir lineage analysis does not directly apply, and the paper should justify why it is safe in this regime. The Dual Hint-MLWE assumption is the answer, but the discussion in §1.2 (line 172) is brief. | Low |

---

## 8. Bottom line

- Definition 3.1 is well-formed and the "noise-free + dual-sided"
  innovation over Hint-MLWE [11] is real and non-trivial.
- Theorem 3.1 is a *tight* reduction in the cryptographic sense
  (`8mε_2` statistical slack, no multiplicative loss), modulo the
  sampleability requirement of Lemma 3.5.
- Lemma 3.5 is the genuinely novel proof step; the Appendix-D
  decomposition into a full-rank G+G sampler is clean.
- Theorem 4.1 / Lemma 4.1 is rigorous but loses `N(Q+1)^2` to
  multi-user + RO programming. This is standard but unaddressed in
  the parameter analysis.
- The "~40-bit Chipmunk" claim is well-grounded in the committed
  estimator output, though the headline number is the most
  charitable read of the data (which actually says 23-39 bits).
- The EUF composition (Theorem C.1, Lemma C.3) inherits its
  argument structure from Chipmunk [8] and is sketched rather than
  re-proved.

The paper's theoretical content is mostly solid. The main missed
opportunity is a discussion of how the reduction loss in Lemma 4.1
interacts with the parameter selection — this is the only gap
between "120-bit lattice problem" and "128-bit signature scheme"
that the paper does not bridge.
