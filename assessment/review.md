## Lemur — Analysis & Correctness Review

<a id="toc"></a>

<sub><sub>

**Contents**

<table><tr><td valign="top">

<a href="#sec-1" style="text-decoration:none">1. TLDR</a><br>
<a href="#sec-2" style="text-decoration:none">2. Field placement and paradigm</a><br>
<a href="#sec-3" style="text-decoration:none">3. What the paper proves and what it does not</a><br>
<a href="#sec-4" style="text-decoration:none">4. Correctness audit — Python reference vs. paper</a><br>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#sec-4-1" style="text-decoration:none">4.1 KOTS — paper Figure 3</a><br>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#sec-4-2" style="text-decoration:none">4.2 HVC — paper Figure 4 + Appendix B</a><br>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#sec-4-3" style="text-decoration:none">4.3 Lemur composition — paper Figure 5</a><br>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#sec-4-4" style="text-decoration:none">4.4 Audit checkpoints</a><br>
<a href="#sec-5" style="text-decoration:none">5. Implementation testing</a><br>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#sec-5-1" style="text-decoration:none">5.1 Rust test suite</a><br>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#sec-5-2" style="text-decoration:none">5.2 Aggregate-signature size — `lemur sizes`</a><br>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#sec-5-3" style="text-decoration:none">5.3 Rice-encoded sizes for larger N</a><br>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#sec-5-4" style="text-decoration:none">5.4 Python ↔ Rust byte-equivalence</a><br>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#sec-5-5" style="text-decoration:none">5.5 Chipmunk security recomputation</a><br>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#sec-5-6" style="text-decoration:none">5.6 Sage parameter-estimator outputs</a>

</td><td valign="top">

<a href="#sec-6" style="text-decoration:none">6. Benchmark measurements vs paper</a><br>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#sec-6-1" style="text-decoration:none">6.1 What I could not run</a><br>
<a href="#sec-7" style="text-decoration:none">7. Performance internals</a><br>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#sec-7-1" style="text-decoration:none">7.1 Gaussian CDT sampler</a><br>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#sec-7-2" style="text-decoration:none">7.2 NTT — two backends</a><br>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#sec-7-3" style="text-decoration:none">7.3 Rayon parallelism</a><br>
<a href="#sec-8" style="text-decoration:none">8. Parameter regeneration flow</a><br>
<a href="#sec-9" style="text-decoration:none">9. Related work — verified facts</a><br>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#sec-9-1" style="text-decoration:none">9.1 Missing references</a><br>
&nbsp;&nbsp;&nbsp;&nbsp;<a href="#sec-9-2" style="text-decoration:none">9.2 Notable framing softening</a><br>
<a href="#sec-10" style="text-decoration:none">10. Open questions and limitations</a><br>
<a href="#sec-11" style="text-decoration:none">11. Reproduction recipe</a><br>
<a href="#footnotes" style="text-decoration:none">Footnotes</a>

</td></tr></table>

</sub></sub>


**This is a technical assessment of the
[Lemur paper](../submission/report.pdf) and independent validation
of its [implementation](../submission/code/) and
[parameter estimator](../submission/code/parameter/).**

<details>
<summary><strong>Notes</strong></summary>

**Paper:** [`submission/report.pdf`](../submission/report.pdf) —
*Lemur: Scalable Post-Quantum Synchronized Multi-Signatures*,
24 pages, anonymous ACM CCS submission for 2026.

**Artifact:** [`submission/code/`](../submission/code/) — Python
reference, Rust performance port, Sage parameter estimator. All paths
in this document are relative to the repo root.

**Hardware:** aarch64 Linux, **11 CPU threads, 8 GiB RAM**
(paper baseline: 24 threads).

**Source of truth.** Three things, in priority order:

1. **The paper** itself — for what is *claimed*.
2. **The code** — for what is *implemented*.
3. **Independent execution** — running the code (and the Sage estimator
   scripts) locally and observing the outputs. The reproduction
   recipe is in §11; the Sage re-runs are in §5.6.

The committed `.txt` files in [`submission/code/parameter/`](../submission/code/parameter/)
are *derived* outputs of the Sage scripts there; they're convenient
quick references, not primary sources. The body of this review cites
the paper by section/theorem/figure, the code by `file:line`, and
empirical measurements by the command that produced them.

</details>


### <a id="sec-1"></a>1. TLDR [↑](#toc)

**The implementation matches the paper.** Every deterministic
quantity I checked (aggregate-signature sizes at N ∈ {2¹⁰, 2¹⁵, 2²⁰}<sup id="ref-1">[1](#fn-1)</sup>,
KOTS/HVC/Lemur algorithm-level structure, Python ↔ Rust byte-level
agreement) reproduces exactly. Every machine-dependent quantity
(timings) tracks the paper to within ~15 % of the expected
thread-count slowdown.

**The paradigm contribution is modest but the engineering payoff is
real.** Lemur stays inside the Squirrel/Chipmunk KOTS+HVC blueprint<sup id="ref-2">[2](#fn-2)</sup>;
it changes two of the four blocks (KOTS and HVC). At paper-scale
`N=2²⁰` the aggregate is ~2.1× smaller than Chipmunk *at corrected
128-bit security*; at `N=2¹⁰` the ratio shrinks to 1.18×<sup id="ref-3">[3](#fn-3)</sup>. The
security-estimation rectification of Chipmunk (from claimed 112-bit
to actual 22.8–39.4-bit range under modern lattice estimation)<sup id="ref-4">[4](#fn-4)</sup> is,
on its own, a useful service to the field.

**Compelling results with three caveats.** (1) The Table 1 comparison
puts Rice-encoded Lemur against raw-encoded Chipmunk, a ~14%
asymmetry disclosed in the caption but not on the column headers<sup id="ref-5">[5](#fn-5)</sup>.
(2) The "order of magnitude" KOTS shrink against Chipmunk is
actually 4.64× at `N=2¹⁰` and ~12.8× at `N=2²⁰`<sup id="ref-6">[6](#fn-6)</sup> — only the favorable
corner reaches the headline. (3) The paper provides **no runtime
comparison vs Chipmunk** (acknowledged in §1.1: "we are unable to
provide a meaningful runtime comparison")<sup id="ref-7">[7](#fn-7)</sup>. For a CCS submission the
substantive contribution survives all three caveats; for production
deployment in Ethereum-style consensus, the absence of timing
comparison and the lack of comparison against
post-2024 paradigms ([Aardal et al.](https://eprint.iacr.org/2024/311)
CRYPTO '24 Falcon+LaBRADOR<sup id="ref-8">[8](#fn-8)</sup>,
[Anada et al.](https://link.springer.com/chapter/10.1007/978-981-96-5566-3_4)
ICISC '24 standard-model lattice synchronized<sup id="ref-9">[9](#fn-9)</sup>) leaves
gaps a reviewer should flag.


### <a id="sec-2"></a>2. Field placement and paradigm [↑](#toc)

The taxonomy of post-quantum multi-signatures the paper sits inside,
extended after the field-lens audit to include the
PQ-sig+lattice-PoK paradigm Lemur does not engage with:

```
                            Multi-signatures
                                  │
   ┌────────────────┬─────────────┴──────────────┬─────────────────┐
   │                │                            │                 │
 Pairing-based   Hash-based +              Lattice-native    PQ-sig + lattice-PoK
 BLS [3]           SNARKs                                      (Aardal et al.
 (Ethereum,     [7] Drake '25              ┌──────┴─────┐        2024/311
  deployed,     LeanSig 2025/1332          │            │      Falcon + LaBRADOR,
  not PQ)       HAPPIER LightSec'25    Synchronized   Non-synchronized
                                      (slot schedule)
                                            │            │
                                    ┌───────┴────┐   Boneh-Kim '20 [2]
                                    │            │  (OTS + interactive,
                              Squirrel '22  Anada et al.   log-size agg)
                                  [9]       '24 ICISC
                                    │      (standard model)
                              Chipmunk '23
                                  [8]
                                    │
                                Lemur '26
```

Lemur sits firmly in the **lattice-native synchronized ROM** column.
Within the synchronized lattice column, Lemur and Anada et al. (ICISC
2024<sup id="ref-9b">[9](#fn-9)</sup>) are siblings, not ancestor/descendant — Anada targets the
*standard model*, a strictly stronger security setting.

Lemur's stated paradigm trichotomy (BLS / hash+SNARK / lattice-
synchronized in §1<sup id="ref-10">[10](#fn-10)</sup>) is **incomplete**: it sidesteps both the
PQ-sig+lattice-PoK bucket (Aardal et al.<sup id="ref-8b">[8](#fn-8)</sup>) and the standard-model
sibling (Anada et al.<sup id="ref-9c">[9](#fn-9)</sup>). Whether deliberate or oversight, this affects
the credibility of "no other PQ aggregate at this scale" framing.

The four building blocks of any scheme in the synchronized lattice
column<sup id="ref-2b">[2](#fn-2)</sup>:

| Block | Lemur's choice | Vs. Chipmunk |
| --- | --- | --- |
| KOTS | Computational unforgeability via Dual Hint-MLWE<sup id="ref-11">[11](#fn-11)</sup> | Chipmunk: statistical (forces larger params) |
| HVC | Module-SIS, matrix commitment domain<sup id="ref-12">[12](#fn-12)</sup> | Chipmunk: Ring-SIS, vector commitment |
| Aggregation | Weighted random sum, γ retries | Inherited unchanged |
| Stateful sign | BDS08 [4]<sup id="ref-13">[13](#fn-13)</sup> | Inherited unchanged |


### <a id="sec-3"></a>3. What the paper proves and what it does not [↑](#toc)

**Proves:**

- A new lattice problem, **Dual Hint-MLWE** (Def 3.1), is at least
  as hard as standard MLWE up to **8m·ε₂ statistical slack with no
  multiplicative loss** (Theorem 3.1<sup id="ref-14">[14](#fn-14)</sup>, proof via four hybrid hops
  Lemmas 3.1–3.4 plus the sampleability lemma 3.5). With the §7.1
  illustrative parameters (`d=128, m=20, ε₂=2^{-136}`), the slack is
  `8·20·2^{-136} ≈ 2^{-128.7}` — narrowly below `2^{-128}`. The
  *shipped* `d=256` cell has `m ∈ {9,10,10,11}` across the four τ
  rows (`parameter/summary.txt`)<sup id="ref-15">[15](#fn-15)</sup>; the worst-case slack at `m=11` is
  `≈ 2^{-129.5}`, slightly more comfortable than the illustration.
- Lemma 3.5 (sampleability over non-full-rank lattices) bridges to
  the G+G '23 [6] full-rank sampler via Lemma 2.6 (rank-r → full-rank
  reduction) using an orthonormal change-of-basis (Appendix D,
  `U^T U = I`)<sup id="ref-16">[16](#fn-16)</sup>. Built on standard ingredients but the packaging is
  the load-bearing technical novelty.
- A KOTS construction (Fig 3) is `W'`-EUF-RK secure in the ROM under
  Dual Hint-MLWE + MSIS (Theorem 4.1, proof via Lemma 4.1 with a
  **multi-user reduction loss of `N(Q+1)²`** where `Q := Q_H + N`)<sup id="ref-17">[17](#fn-17)</sup>.
- An HVC construction (Fig 4) over Module-SIS satisfies individual
  correctness, probabilistic homomorphism, robust homomorphism, and
  position binding (Theorem B.1)<sup id="ref-18">[18](#fn-18)</sup>.
- Composition: a synchronized multi-signature (Fig 5) inherits
  unforgeability and aggregation properties from the components<sup id="ref-19">[19](#fn-19)</sup>.

**Does not prove (and does not claim to):**

- Accountable aggregation (signer identification beyond the public
  list P). The Micali-Ohta-Reyzin model [15] is adjacent but unused;
  in fact [15] appears only in the `[10, 15]` opening pair and the
  bibliography — bibliography filler, not load-bearing<sup id="ref-20">[20](#fn-20)</sup>.
- Multi-message aggregation. Every signer must sign the same message
  at the same slot.
- Security after slot reuse. KOTS is one-time *per slot*; reusing a
  slot leaks the secret by construction.

**Loose ends in the proof structure:**

- **Lemma 4.1 is restricted to ℓ=1**<sup id="ref-21">[21](#fn-21)</sup>. The paper itself
  acknowledges this in the §4.1 preface to Theorem 4.1's proof:
  "Some of our proofs for the KOTS require ℓ = 1, which is already
  always the case in our parameter setting. However, we keep the
  description general by using the ℓ parameter." All shipped
  parameters use `ℓ=1`, so this is pragmatically fine — but
  Theorem 4.1 is stated for general `ℓ`, and the restriction is one
  layer down at Lemma 4.1.
- **The `N(Q+1)²` reduction loss is not absorbed in parameter
  selection.** At `N=2²⁰`, `Q_H=2⁶⁰`, the loss is `~2¹⁴⁰`, which
  would require ~268-bit core hardness to absorb fully. Lemur picks
  parameters at the 128-bit core-SVP level. This is community-
  standard practice for lattice multi-sigs (Dilithium does the same)
  but represents a real gap a strict reviewer would flag.
- **Lemma C.3 EUF composition is sketched, not fully proved**,
  with explicit "follows the same idea as [8]" disclaimer<sup id="ref-22">[22](#fn-22)</sup>.

**Headline-claim category map** (per the paper-analysis skill):

- **Size claims** (deterministic): 201.2 KB at `N=2¹⁰`, 283.5 KB at
  `N=2¹⁵`, 394.4 KB at `N=2²⁰` — all reproduced exactly (§5 below)<sup id="ref-1b">[1](#fn-1)</sup>.
- **Encoding claims** (deterministic, two columns): Tables 1 and 2
  both report Rice-encoded sizes (Table 1 is the Chipmunk comparison;
  Table 2 has an "Agg. signature size" row in an otherwise-timing
  table), while Table 5 reports worst-case theoretical bounds before
  Rice coding<sup id="ref-23">[23](#fn-23)</sup>. The two columns differ by 14–16% per cell
  (15.8/14.4/13.7% at `N=2^{10,15,20}`). Table 1's Lemur-vs-Chipmunk
  comparison further mixes Rice-encoded Lemur with raw-encoded
  Chipmunk; the caption discloses this asymmetry but the column
  headers do not flag it<sup id="ref-5b">[5](#fn-5)</sup>.
- **Timing claims** (machine-dependent): 4.1 ms stateful sign,
  30.1 ms batch verify at `N=2¹⁰` at 24 threads<sup id="ref-24">[24](#fn-24)</sup> — reproduced within
  ~10–15 % of the linear thread-count scaling (§6 below).
- **Security claims** (estimator-bound): λ = 128 under MLWE + MSIS
  via APS-style lattice estimation<sup id="ref-25">[25](#fn-25)</sup>. **Independently re-run** via
  SageMath 9.5 installed during this review; all three estimator
  outputs
  ([`summary.txt`](../submission/code/parameter/summary.txt),
  [`chipmunk_summary.txt`](../submission/code/parameter/chipmunk_summary.txt),
  [`chipmunk_original_security_summary.txt`](../submission/code/parameter/chipmunk_original_security_summary.txt))
  reproduce against the committed files — bit-identical for the two
  chipmunk files, and field-identical row-by-row across all 16
  (τ, N) cells for `summary.txt`<sup id="ref-51">[51](#fn-51)</sup>.


### <a id="sec-4"></a>4. Correctness audit — Python reference vs. paper [↑](#toc)

The Python reference (`lemur-py/`, ~3.6 kLOC) is the spec. Variable
naming matches the paper to the symbol. The Rust port (`lemur-rs/`,
~9 kLOC) is byte-equal to it on test vectors (§5).

#### <a id="sec-4-1"></a>4.1 KOTS — `lemur-py/kots.py` ↔ paper Figure 3 [↑](#toc)

| Paper algorithm | Code location | Match |
| --- | --- | --- |
| `Setup: A₂ ← R_{q'}^{(m-n)×n}, return A = [I_n; A₂]` | `kots.py:setup` (lines 105–118); structured form stored as A₂ only, A reconstructed in `_matmul_with_A` | ✓ |
| `KGen: S ← D^{k×m}_{R,α}, T = SA mod q'` | `kots.py:keygen` (lines 126–138); Gaussian via CDT with `σ = α / √(2π)` | ✓ |
| `Sign: H = H(μ) = [I_ℓ | H'] with H' ← T_{α_H}, Z = HS` | `kots.py:_build_H` (lines 90–99) for H; `kots.py:sign` (lines 140–152) for Z; identity block encoded as `H[i,i,0]=1` (constant polynomial 1) | ✓ |
| `iVrfy bound β_z` / `sVrfy bound β_σ` / `wVrfy bound 2β_σ` | Three thin wrappers `ivrfy`/`svrfy`/`wvrfy` (lines 163–170) over `vrfy(beta)` (lines 154–161) | ✓ |

#### <a id="sec-4-2"></a>4.2 HVC — `lemur-py/hvc.py` ↔ paper Figure 4 + Appendix B [↑](#toc)

| Paper algorithm | Code location | Match |
| --- | --- | --- |
| `Setup: B ← R_q^{ω×ρν·κ'}, A_0, A_1 ← R_q^{ω×ω·κ}` | `hvc.py:setup` (lines 438–452) | ✓ |
| Leaf label `c_i = dec_q(B · vec(M_i))` | `hvc.py:_leaf_label` (lines 403–406) | ✓ |
| Internal label `c_j = dec_q(A_0·left + A_1·right)` | `hvc.py:_internal_label` (lines 408–412); reduces mod `self.q` (HVC modulus), **not** `self.q_prime` (KOTS modulus) | ✓ |
| Balanced `(2η+1)`-ary decomposition (Def B.6) | `hvc.py:_decompose_coeff` / `_dec_poly` / `_proj_poly` (lines 190–235) | ✓ |
| Babai encode/decode for compressed openings (Fig 6, Lemma B.1) | `hvc.py:_decompose_coeff_zz` / `babai_encode_block` / `babai_decode_block` (lines 241–340) | ✓ |
| BDS08 traversal [4]: auth path + treehash + retain | `hvc.py:bds_init` / `bds_advance` / `bds_opening` (lines 467–635); `phi` counter is the leaf cursor | ✓ |

#### <a id="sec-4-3"></a>4.3 Lemur composition — `lemur-py/lemur.py` ↔ paper Figure 5 [↑](#toc)

| Paper algorithm | Code location | Match |
| --- | --- | --- |
| `Setup, Sign, iVrfy, aVrfy` interface | `lemur.py:setup` / `sign_seed` / `sign_stateful` / `ivrfy` / `avrfy` (lines 94–321) | ✓ |
| `Aggregate`: pre-verify every input, then do-while with `H(t,μ,P,j)` for up to γ attempts | `lemur.py:aggregate` (lines 264–297); randomizer oracle at `_hash_to_randomizers` (lines 44–58); break on `avrfy` passing (line 294), **not** on local norm checks alone | ✓ |
| Weighted-sum on Z and on opening | `lemur.py:_scale_opening` + `_add_openings` + per-signer `kots.ring.scale_mat` in aggregate loop | ✓ |

#### <a id="sec-4-4"></a>4.4 Audit checkpoints — none of the foot-guns triggered [↑](#toc)

Four spots that consume 90 % of audit time in this class of scheme.
All four are correctly implemented (independently verified by the
implementation-lens audit and its reviewer):

- `kots.py:_build_H` line 95: identity block is `H[i,i,0]=1`
  (constant polynomial 1), not the polynomial `1+x+x²+…`<sup id="ref-26">[26](#fn-26)</sup>. ✓
- `kots.py:vrfy` lines 154–170: `β_z`, `β_σ`, `2·β_σ` bounds exactly<sup id="ref-27">[27](#fn-27)</sup>. ✓
- `hvc.py:_internal_label` line 411: reduction is `% self.q`
  (HVC modulus), not `q'` (KOTS modulus)<sup id="ref-28">[28](#fn-28)</sup>. ✓
- `lemur.py:aggregate` line 294: breaks via `avrfy` (norm +
  algebraic identity), not on a partial check<sup id="ref-29">[29](#fn-29)</sup>. ✓


### <a id="sec-5"></a>5. Implementation testing — deterministic checks [↑](#toc)

#### <a id="sec-5-1"></a>5.1 Rust test suite [↑](#toc)

```
cargo test --release
```

**Result:** 52/52 tests pass across 6 test binaries (1 lib unit-test
harness + 5 integration-test files)<sup id="ref-30">[30](#fn-30)</sup>:

| Test binary | Tests | Coverage |
| ---: | ---: | --- |
| unit tests in `lemur_rs` | 25 | poly arithmetic, NTT, sampler, codec, profile invariants |
| `bds_stateful` | 8 | BDS08 state advance/opening consistency |
| `gauss_ctx` | 5 | CDT correctness, narrower-CDT compatibility |
| `materialized_tree` | 5 | Tree build/sign matches BDS path every slot |
| `profile_pipeline` | 4 | Full pipeline, encoding snapshots, runtime HVC bounds |
| `robustness` | 5 | Malformed-input rejection (truncation, bitflip, wrong context) |
| doctests | 0 | — |

Total: 52 tests. Time: ~1 minute on 11 threads.

#### <a id="sec-5-2"></a>5.2 Aggregate-signature size — `lemur sizes` [↑](#toc)

```
cargo run --release --bin lemur -- sizes
```

| Object | Paper Table 5 / Table 1 | Measured | Match |
| --- | ---: | ---: | --- |
| `pp` (seeds + τ) | — | 65 B | — |
| `sk` master seed | — | 32 B | — |
| `sk.state` (fresh BDS) | 134.4 KB | 134.4 KB | ✓ |
| `pk` (HVC commitment) | 3.4 KB | 3.4 KB | ✓ |
| Individual sig | 89.5 KB | 89.5 KB | ✓ |
| ↳ Z (KOTS sig, individual) | — | 4.2 KB | — |
| ↳ sibling labels | — | 68.8 KB | — |
| ↳ u | — | 16.5 KB | — |
| **Aggregated sig, N=2¹⁰** | **201.2 KB** | **201.2 KB** | **✓ exact** |
| ↳ Z_agg (5.6 KB, wider bit-width than Z) | — | 5.6 KB | — |
| ↳ Babai path (Rice k=5) | — | 50.8 KB | — |
| ↳ sibling labels (Rice k=15) | — | 116.8 KB | — |
| ↳ u (Rice k=15) | — | 28.0 KB | — |

Sub-component sum: 5.6 + 50.8 + 116.8 + 28.0 = **201.2 KB**<sup id="ref-31">[31](#fn-31)</sup>. Note:
the aggregated signature uses **Z_agg** (5.6 KB) at a wider bit-width
tuned to `β_σ`, not the individual KOTS sig's **Z** (4.2 KB) — these
are different objects despite similar names.

`bench --fast` realized aggregate-size: **195.8 KB** (formula
predicts 201.2 KB; documented ~3 % Rice-coding variance, per
`code/README.md` last paragraph)<sup id="ref-32">[32](#fn-32)</sup>.

#### <a id="sec-5-3"></a>5.3 Rice-encoded sizes for larger N — `rice_sizes.py` [↑](#toc)

```
cd parameter && python3 rice_sizes.py
```

| N | Worst-case (Table 5) | Rice-encoded (Tables 1 & 2) | Measured Rice | Match |
| ---: | ---: | ---: | ---: | --- |
| 2¹⁰ | 239 KB | 201.2 KB | 201.2 KB | ✓ |
| 2¹⁵ | 331 KB | 283.5 KB | 283.5 KB | ✓ exact |
| 2²⁰ | 457 KB | 394.4 KB | 394.4 KB | ✓ exact |

The Rice/worst gap is 13.7–15.8 % across cells. Tables 1 and 2 use
Rice-encoded sizes; Table 5 uses worst-case theoretical bounds<sup id="ref-23b">[23](#fn-23)</sup>. Table 1's Lemur-vs-Chipmunk
comparison places Rice-encoded Lemur against raw Chipmunk, an
asymmetry disclosed in the caption ("Lemur: measured in
implementation for `N=2¹⁰`; Rice-encoded on the corresponding cells
of Table 5 for `N ∈ {2¹⁵, 2²⁰}`. Chipmunk: theoretical, from their
scripts").

#### <a id="sec-5-4"></a>5.4 Python ↔ Rust byte-equivalence — `vectors` [↑](#toc)

```
python3 cli.py vectors --tau 3 --signers 2 --slot 0 --msg "artifact check" --out /tmp/lemur-py-vectors.json
lemur vectors --tau 3 --signers 2 --slot 0 --msg "artifact check" --out /tmp/lemur-rs-vectors.json
diff <(jq -S . py.json) <(jq -S . rs.json)
```

**Result:** all keys match byte-for-byte:

```
pp:           MATCH
signatures:   MATCH
ivrfy:        MATCH
avrfy:        MATCH
agg_attempt:  MATCH
aggregate:    MATCH
pk 0:         MATCH
pk 1:         MATCH
```

This is the strongest evidence that both implementations realize the
*same* scheme<sup id="ref-33">[33](#fn-33)</sup>. Particularly noteworthy: the **Python uses schoolbook
multiplication for the KOTS modulus `q'`** while the **Rust uses
CRT-via-two-aux-primes NTT** (`aux_ntt.rs`)<sup id="ref-34">[34](#fn-34)</sup>, and they agree
byte-for-byte on `Z`. The schoolbook and CRT implementations of the
same KOTS multiplication produce identical signed Z coefficients.

#### <a id="sec-5-5"></a>5.5 Chipmunk security recomputation — committed estimator output [↑](#toc)

Inspected [`parameter/chipmunk_original_security_summary.txt`](../submission/code/parameter/chipmunk_original_security_summary.txt)
directly. Independent verification by the field-lens audit and its
reviewer:

- **RSIS rop range across all 24 rows × 3 columns: [22.8, 39.4]**
  bits of core-SVP security<sup id="ref-4c">[4](#fn-4)</sup>.
- Min 22.8: RSIS3 column at `secpar=112, τ ∈ {21,23,24,26}, N=131072`.
- Max 39.4: RSIS1/RSIS2 at `secpar=112, τ=21, N=1024`.
- For `secpar=128` rows specifically: range `[23.1, 38.8]`.

Lemur's headline "approximately 40-bit instead of the claimed 112-bit"
(§1.1)<sup id="ref-35">[35](#fn-35)</sup> is the **max-over-instances** reading; a worst-cell reading
gives ~23 bits. Either way the gap from the 112/128-bit claim is real
and large. The data is convincing as evidence Chipmunk's parameters
are far below their claimed security level, just not "exactly 40."

#### <a id="sec-5-6"></a>5.6 Sage parameter-estimator outputs — reproduced from source [↑](#toc)

Earlier review iterations could not validate the parameter-estimator
outputs because SageMath was not installed. SageMath 9.5 was
installed for this round (`apt-get install --no-install-recommends
sagemath`), all three Sage scripts re-run from
`submission/code/parameter/`, and the freshly-generated outputs
compared against the committed files:

| Script | Output file | Runtime | Result |
| --- | --- | ---: | --- |
| [`chipmunk_original.sage`](../submission/code/parameter/chipmunk_original.sage) | [`chipmunk_original_security_summary.txt`](../submission/code/parameter/chipmunk_original_security_summary.txt) | 2m45s | **byte-identical** to committed |
| [`chipmunk_param.sage`](../submission/code/parameter/chipmunk_param.sage) | [`chipmunk_summary.txt`](../submission/code/parameter/chipmunk_summary.txt) | 13s | **byte-identical** to committed |
| [`lemur_param.sage`](../submission/code/parameter/lemur_param.sage) | [`summary.txt`](../submission/code/parameter/summary.txt) | ~25 min (chunked) | **field-identical** row-by-row across all 16 `(τ, N)` cells |

Notes:

- **`chipmunk_original.sage`** searches `secpars × taus × rhos =
  2 × 4 × 3 = 24` cells under the original Dilithium-style estimator.
  Diff against the committed file is empty — the `22.8 → 39.4` RSIS
  rop range and every other column reproduce exactly.
- **`chipmunk_param.sage`** searches the same Chipmunk space under
  the *MSIS-optimised* estimator; finishes in 13 s and the output
  is bit-identical to committed.
- **`lemur_param.sage`** was OOM-killed at ~3 min on a single
  end-to-end run (the lattice-estimator allocates heavily during
  the higher-β MLWE estimates). Re-running in 16 sub-runs partitioned
  by `(τ, N)` succeeded in ~25 min total; parsed back into rows the
  resulting `summary.txt` matches the committed file field-by-field
  across all 16 cells (verified by a per-cell Python diff keyed on
  `(τ, N)`). The wall-time gap vs `chipmunk_param.sage` reflects
  Lemur's broader parameter search and heavier MSIS-vs-MLWE
  cross-checks per cell.
- Committed outputs are correctly produced from the committed source
  scripts; no hidden inputs, no manual edits.


### <a id="sec-6"></a>6. Benchmark measurements vs paper [↑](#toc)

Hardware: ARM aarch64 Linux, 11 threads, 8 GiB RAM. Paper baseline:
24 threads.

**Expected slowdown vs. baseline**: ~2.2× linear thread-count ratio
(24/11). Anything within ±15 % of that ratio counts as a match.

**Important:** the baseline column below mixes two distinct
24-thread sources. Paper Table 2 reports only **stateful sign,
tree-in-memory sign, aggregation, batch verification, and the agg.
signature size**. Everything else (key generation, online sign, full
sign, individual pre-verify, aggregate-after-verified-inputs,
4.13 ms vs the paper's rounded 4.1 ms) comes from
`submission/code/README.md`, a 24-thread artifact-only run on the
same machine. Both sources are independently reproducible; just
don't quote a README row as a "paper claim".

| Operation | Baseline (24 thr) | Source | Measured (11 thr) | Ratio | Verdict |
| --- | ---: | --- | ---: | ---: | --- |
| Key generation | 1.3 min | README | 1.7 min | 1.3× | ✓ better than expected |
| Online sign (KOTS only) | (not reported) | — | 304 µs | — | implementation-internal |
| Full sign | 1.3 min | README | 1.7 min | 1.3× | ✓ |
| Stateful sign (BDS08) | 4.1 ms (paper) / 4.13 ms (README) | Table 2 / README | 3.91 ms | **0.95×** | ✓ serial-dominated |
| Individual pre-verify, N=2¹⁰ | 1.67 s | README | 4.40 s | 2.6× | ✓ within ±15 % of 2.2× |
| Aggregate after verified inputs, N=2¹⁰ | 2.40 s | README | 5.74 s | 2.4× | ✓ |
| **Secure aggregation, N=2¹⁰** | **567 ms** | **Table 2** | **1.41 s** | 2.5× | ✓ |
| **Batch verification, N=2¹⁰** | **30.1 ms** | **Table 2** | **79.31 ms** | 2.6× | ✓ |
| Batch verify, N=2¹³ (zero-fixture) | — | — | 589 ms | — | — |
| Batch verify, N=2¹⁵ (zero-fixture) | 812 ms | Table 2 | 1.22 s | 1.5× | ✓ better than expected |

**Key ratios** (machine-independent, more diagnostic than absolutes):

| Ratio | Baseline | Measured | Verdict |
| --- | ---: | ---: | --- |
| Stateful sign : full sign | 1 : 19 000 (4.1 ms vs ~1.3 min — paper × README) | 1 : 26 000 | ✓ (BDS cache speedup) |
| Secure aggregation : batch verify (N=2¹⁰) | 19 : 1 (both paper Table 2) | 18 : 1 | ✓ |

The "Online sign : stateful sign ≈ 1 : 12" ratio cited in earlier
review iterations was implementation-internal (not in Table 2);
the **paper does not report online-sign time**.

Sampler microbench (`bench --sampler-only`):

| Path | µs / poly (256 coeffs) |
| --- | ---: |
| Profile-specialised (indexed CDT) | **1.7 µs** |
| Generic `GaussCtx` (runtime CDT) | 3.8 µs |
| Const-folded per-sample read (32-bit) | 4.1 µs |
| Const-folded batched read (32-bit) | 3.1 µs |

Indexed-CDT bucket dispatch gives a **~2.2× speedup** over the
generic context path, confirming the layered optimization claim<sup id="ref-36">[36](#fn-36)</sup>.

#### <a id="sec-6-1"></a>6.1 What I could not run [↑](#toc)

- **N=2²⁰ batch verify (`bench_verify --n 1048576`)** — needs ~9 GiB
  RAM; container has 8 GiB<sup id="ref-37">[37](#fn-37)</sup>.
- **N=8192 aggregation end-to-end** — OOM-killed during signer
  replication (~9 GiB peak working set).
- **`--with-tree` materialised-tree signing** — needs 8 GiB just for
  the HVC tree at τ=20.
- ~~**Sage-based parameter regeneration** — no SageMath installed.~~
  **Resolved during this review.** SageMath 9.5 installed via
  `apt-get install --no-install-recommends sagemath`; all three
  Sage scripts re-run with outputs matching the committed
  [`summary.txt`](../submission/code/parameter/summary.txt) /
  [`chipmunk_summary.txt`](../submission/code/parameter/chipmunk_summary.txt) /
  [`chipmunk_original_security_summary.txt`](../submission/code/parameter/chipmunk_original_security_summary.txt)
  (see §5.6 below)<sup id="ref-51b">[51](#fn-51)</sup>.

None of these gaps undermine the central claim. Every measurement
that fits in 8 GiB tracks the paper to within a thread-count factor,
and every deterministic size figure reproduces exactly.


### <a id="sec-7"></a>7. Performance internals — what is actually fast [↑](#toc)

The Rust port has three identifiable layers of careful optimization,
all of them benchmarkable in isolation.

#### <a id="sec-7-1"></a>7.1 Gaussian CDT sampler (`lemur-rs/src/sample.rs`) [↑](#toc)

1. **Indexed bucket dispatch** (`sample_cdt_indexed` + `cdt_hi` table
   emitted by `gen_tables.py`): the top 9 bits of a 32-bit sample
   `u` index into a small `[lo, hi]` window within the CDT, cutting
   binary-search depth from ~log₂(175) ≈ 8 levels to ~1–2<sup id="ref-38">[38](#fn-38)</sup>.
   **Measured speedup vs. generic GaussCtx: 2.2×.**
2. **Batched XOF reads**: one `xof.read(d · cdt_bytes)` of ≤ 1024
   bytes per poly, instead of `d` per-coefficient reads. Same
   Keccak-f count (SHAKE buffers a rate block internally); just
   amortises per-call dispatch overhead. ~30 % win on its own.
3. **Stack-allocated buffers** sized to `MAX_D = 256`, zero-alloc
   on the hot path.

#### <a id="sec-7-2"></a>7.2 NTT — two backends (`ntt.rs` + `aux_ntt.rs`) [↑](#toc)

Lemur has two ring moduli with different NTT properties:

| Modulus | Property | Multiplication backend |
| --- | --- | --- |
| HVC `q ≈ 2⁵³` | NTT-friendly (`q ≡ 1 mod 2d`) | `ntt.rs` u64 Montgomery: Cooley-Tukey forward / Gentleman-Sande inverse, constant-time signed-mask reduction |
| KOTS `q' ≡ 17 (mod 32)` | **Not** NTT-friendly at length d=256 | `aux_ntt.rs` CRT-over-two-aux-primes: lift coeffs to (-q/2, q/2], NTT under two primes near 2⁴⁸, pointwise multiply, inverse NTT, Garner-CRT back, reduce mod q' |

The CRT primes are compile-time constants (`AUX_P1`, `AUX_P2`) so
LLVM specialises `% p`<sup id="ref-39">[39](#fn-39)</sup>. The Python reference uses schoolbook for q'
multiplication — that's the only place Python is not a faithful
performance reference, and the byte-equality on `vectors` proves the
two compute the same signed coefficients.

#### <a id="sec-7-3"></a>7.3 Rayon parallelism (`lemur-rs/src/lemur.rs:lemur_aggregate`) [↑](#toc)

Three concurrent map-reduce passes<sup id="ref-40">[40](#fn-40)</sup>:

- **Individual pre-verification**:
  `pks.par_iter().zip(sigs.par_iter()).try_for_each(lemur_ivrfy)`
- **Weighted Z aggregation**: per-signer `scale_mat_crt` in parallel,
  reduced with commutative add. Three branches in the source for
  CRT / u64-NTT / u32-NTT backends; production path is CRT.
- **Weighted opening aggregation**: same pattern with `add_openings`.

`par_iter().reduce_with` is safe here because addition is
commutative. Thread scaling is near-ideal: 24-thread paper timings
become ~2.5× longer on 11 threads, within 15 % of linear.


### <a id="sec-8"></a>8. Parameter regeneration flow [↑](#toc)

Source of truth: [`parameter/lemur_param.sage`](../submission/code/parameter/lemur_param.sage)
→ outputs [`parameter/summary.txt`](../submission/code/parameter/summary.txt).
The flow downstream is **manual**:

```
lemur_param.sage           (Sage; uses estimator/ + msis_estimator/)
   │
   ▼
summary.txt                (committed; one row per (τ,N,d,k) cell)
   │ hand-transcribe ONE row into:
   ├──────────────►  lemur-py/profiles.py        (LemurProfile dataclass)
   │                       │ imported by:
   │                       ▼
   │                 lemur-rs/gen_tables.py      (emits Rust tables)
   │                       │
   │                       ▼
   │                 lemur-rs/src/tables_*.rs    (NTT zetas, CDT, CDT_HI, …)
   │
   └──────────────►  lemur-rs/src/profile.rs     (Profile static, references tables_*.rs)
```

`gen_tables.py` imports from `lemur-py`, so the Python profile is
the actual hub. The Rust profile + tables are downstream. For the
KOTS modulus, since it's not NTT-friendly, only the HVC tables are
emitted; KOTS multiplication routes through the compile-time-fixed
aux primes in `aux_ntt.rs` and does not need regenerating.

**§7.1 trap:** the paper walks the parameter-selection methodology on
a cell `(d=128, N=2²⁰, τ=24, α_w=31, k=5, r=3, α=61, β_z=8185)`<sup id="ref-41">[41](#fn-41)</sup> that
**is not the cell the implementation ships**. The shipped cell is
`(d=256, k=4, τ=20, N=1024, α=87, α_H=60, α_w=23, β_z=14046)`<sup id="ref-42">[42](#fn-42)</sup>. Every
numeric value differs. The §7.1 walk is pedagogical methodology, not
a derivation of the shipped numbers; the shipped numbers come from
`parameter/summary.txt` line 11 (τ=20, N=1024 row).

Also note: `parameter/summary.txt`'s **first column is `secpar`**
(security parameter λ, always 128) and the *fourth* column is `d`
(ring dimension, always 256 in the shipped family). A reader
scanning the file may see "128" in column 1 and conflate it with
`d=128`; this is the same trap the §7.1 worked example sets.


### <a id="sec-9"></a>9. Related work — verified facts [↑](#toc)

The per-reference taxonomy that backs this section was assembled
during the original review session (notes file lives outside the
repo, under the workspace's `tmp/` scratch area; the verdicts below
restate its conclusions). Verified from the cited papers'
abstracts/intros (fetched May 2026) plus the field-lens audit's
additional discoveries:

| Ref | Paper | Verified claim |
| --- | --- | --- |
| [3] | [BLS '01](https://link.springer.com/chapter/10.1007/3-540-45682-1_30) | 48-byte sigs, 96-byte aggregates; pairing-based; deployed in Ethereum/Dfinity; not PQ<sup id="ref-43">[43](#fn-43)</sup> |
| [7] | [Drake et al. '25](https://eprint.iacr.org/2025/055)<sup id="ref-44">[44](#fn-44)</sup> | Hash-based: Winternitz + Merkle + pqSNARK. Targets < 4 KiB sigs for Ethereum PQ. SNARK is the aggregation mechanism. **Has 2025+ successors not in Lemur's bibliography:** [LeanSig](https://eprint.iacr.org/2025/1332)<sup id="ref-45">[45](#fn-45)</sup>, [HAPPIER](https://link.springer.com/chapter/10.1007/978-3-032-15541-2_1)<sup id="ref-46">[46](#fn-46)</sup>. |
| [9] | [Squirrel '22](https://eprint.iacr.org/2022/694)<sup id="ref-47">[47](#fn-47)</sup> | First synchronized lattice multi-sig. Ring-SIS. ROM, rogue-key safe. |
| [8] | [Chipmunk '23](https://eprint.iacr.org/2023/1820)<sup id="ref-48">[48](#fn-48)</sup> | 5.6× smaller aggregate than Squirrel. **Claimed** ~136 KB for 8192 signers at 112-bit security. Lemur's recomputation: actually 22.8–39.4 bits of core-SVP security across cells<sup id="ref-4d">[4](#fn-4)</sup>; the "~40-bit" headline is max-over-instances. |
| [11] | [Hint-MLWE '23](https://eprint.iacr.org/2023/623)<sup id="ref-49">[49](#fn-49)</sup> | MLWE with bounded noisy hints; efficient reduction to standard MLWE. Originally for ZK proofs; Lemur extends to **Dual** Hint-MLWE with two real differences: (a) noise-free hints (vs `z_i = c_i·s + y_i` in Kim et al.) and (b) dual-sided secret placement (`T = SA`, `Z = HS` over the *same* `S`). |
| [13] | Lyubashevsky-Micciancio '08 | Compact lattice OTS — the conceptual root of Chipmunk's KOTS. Statistical unforgeability argument. Lemur swaps to computational. |
| [2] | [Boneh-Kim '20](https://crypto.stanford.edu/~skim13/agg_ots.pdf)<sup id="ref-50">[50](#fn-50)</sup> | Lattice aggregate sigs based on standard SIS. Two variants: public-agg OTS (logarithmic aggregate) + interactive many-time. **Non-synchronized — a competing paradigm, not an ancestor.** Lemur §1.2 places it in the KOTS ancestor line, which is a placement the reader should question. |
| [4] | BDS08 '08 | Standard Merkle traversal: `O(τ)` state vs `O(2^τ)`. Used unchanged by Lemur's stateful signer. |
| [15] | MOR '01 (accountable-subgroup multisig) | Appears only in the `[10, 15]` opening pair and bibliography. **Bibliography filler**, not load-bearing for Lemur<sup id="ref-20b">[20](#fn-20)</sup>. |

#### <a id="sec-9-1"></a>9.1 Missing references the paper *should* have engaged with [↑](#toc)

Three works exist in adjacent paradigms, all dated 2024 (well before
Lemur's 2026 submission), all uncited:

- [**Aardal-Aranha-Boudgoust-Kolby-Takahashi, "Aggregating Falcon
  Signatures with LaBRADOR"**](https://eprint.iacr.org/2024/311),
  **CRYPTO 2024**<sup id="ref-8c">[8](#fn-8)</sup>. Constructs aggregation of standard Falcon
  signatures using LaBRADOR (a lattice-based proof system). Defines
  an entire paradigm Lemur's §1 trichotomy does not contain — PQ-sig
  + lattice-PoK.
- [**Anada-Fukumitsu-Hasegawa, "Tightly Secure Lattice-Based
  Synchronized Aggregate Signature in Standard Model"**](https://link.springer.com/chapter/10.1007/978-981-96-5566-3_4),
  **ICISC 2024**<sup id="ref-9d">[9](#fn-9)</sup>. A direct sibling of Lemur in the
  synchronized lattice column, with a **strictly stronger security
  model** (standard, not ROM). Apples-to-apples comparison would
  require either Lemur lifting to standard-model security or
  acknowledging that Anada is tighter on that axis.
- **Hash+SNARK ecosystem successors to [7]:**
  [LeanSig](https://eprint.iacr.org/2025/1332)<sup id="ref-45b">[45](#fn-45)</sup>,
  [HAPPIER](https://link.springer.com/chapter/10.1007/978-3-032-15541-2_1)<sup id="ref-46b">[46](#fn-46)</sup>. Both extend Drake et al.'s
  framework. Their existence post-dates Lemur's submission but
  matters for assessing whether the field has moved past Drake et
  al.'s "heavy proof machinery" framing.

#### <a id="sec-9-2"></a>9.2 Notable framing softening worth flagging [↑](#toc)

- **"Order of magnitude smaller KOTS" (§1.1)** — actual KOTS shrink
  vs Chipmunk is **4.64× at `N=2¹⁰`** (26 KB → 5.6 KB in Table 1)
  and ~**12.8× at `N=2²⁰`** (>110 KB → 8.6 KB). "An order of
  magnitude" is on the favorable end of this range.
- **"2× smaller than Chipmunk" headline** — per-cell ratios are
  **1.18× at `N=2¹⁰`, 1.47× at `N=2¹⁵`, 2.1× at `N=2²⁰`**. The "2×"
  is the favorable-corner reading.
- **Table 1 Rice-vs-raw asymmetry** — Lemur is Rice-encoded;
  Chipmunk is raw. Disclosed in the caption but not the column
  header. ~14% asymmetry.
- **Drake et al. dismissal** — the paper devotes one sentence to
  the hash+SNARK paradigm ("heavy proof machinery") with no
  benchmarked numbers and no engagement with 2025 successors. A
  reviewer wanting to know "is Lemur actually better than LeanSig
  for Ethereum?" gets no answer.

The "Chipmunk security recomputation to 22.8–39.4 bits" is the
central field-level finding of this paper. **Independently
reproduced during this review:**
[`sage chipmunk_original.sage`](../submission/code/parameter/chipmunk_original.sage)
completes in ~2m45s on this host and produces a
[`chipmunk_original_security_summary.txt`](../submission/code/parameter/chipmunk_original_security_summary.txt)
that is **byte-identical** to the committed file<sup id="ref-51c">[51](#fn-51)</sup>. The data is convincing as evidence
Chipmunk's parameters miss their claimed security level by 70+ bits
in the median case.


### <a id="sec-10"></a>10. Open questions and limitations of this review [↑](#toc)

- **Security proof not formally audited.** The 128-bit claim rests
  on the lattice estimator [1] and the MSIS estimator. I did not
  re-run either. The theory-lens audit verified the proof structure
  (Theorem 3.1's tight `8m·ε₂` bound, Lemma 3.5's reduction chain,
  Lemma 4.1's `N(Q+1)²` loss) at the lemma-statement level but did
  not check every hybrid hop. The implementation correctness is
  independent of this (it relies on the assumption, not the
  reduction), but the *security* of the implementation rests on
  Theorem 4.1 being correct.

- **Reduction loss not absorbed.** `N(Q+1)² ≈ 2¹⁴⁰` at
  `N=2²⁰, Q_H=2⁶⁰`. Lemur picks MLWE/MSIS parameters at the 128-bit
  core-SVP level; absorbing the multi-user loss would require
  ~268-bit hardness. Community-standard practice for lattice
  multi-sigs (Dilithium does the same) but a real gap a strict
  reviewer should flag.

- **No runtime comparison vs Chipmunk.** §1.1 acknowledges: "Since
  Chipmunk's implemented parameters are at a substantially lower
  security level (around 40 bits) than ours (128 bits), we are
  unable to provide a meaningful runtime comparison against
  Chipmunk."<sup id="ref-7b">[7](#fn-7)</sup> Reasonable but worth noting — a reader who wants to
  know "how fast is verification at scale vs Chipmunk?" gets no
  answer. The "comparison" is purely size-based.

- **N=2²⁰ end-to-end not measured.** Only the batch-verify path at
  `N=2¹⁵` via `bench_verify --zero-fixture`. The aggregation timing
  at `N=2²⁰` is *extrapolated* in the paper from the `N=8192`
  measurement via linear scaling; the paper acknowledges this. I did
  not even measure `N=8192` aggregation here (OOM). A reviewer with
  16+ GiB RAM should confirm linearity holds.

- **Lemma 4.1 ℓ=1 restriction.** Proved only for `ℓ=1`; all shipped
  parameters use `ℓ=1` so this is pragmatically fine, but Theorem
  4.1 is stated for general `ℓ` and the restriction enters one layer
  down. The paper acknowledges this (PDF line 545).

- **Side-channel resistance not assessed.** The code claims
  constant-time discipline in `ntt.rs` (signed-mask reduction,
  branch-free conditional subtract). Not verified with a
  side-channel testing tool (e.g., `dudect`). The KOTS Gaussian
  sampler is constant-time by construction (every coefficient
  consumes exactly `cdt_bytes` of XOF output regardless of value).

- **Comparison vs. Drake et al. is qualitative only.** The
  field-lens audit notes the paper's one-sentence dismissal of
  hash+SNARK aggregation does not cite benchmarked numbers and
  ignores LeanSig / HAPPIER. The Lemur paper itself reports
  ~12 minutes for aggregation at `N=2²⁰`, which is *within the
  same order of magnitude* as the "seconds-to-minutes" range cited
  for pqSNARK proof generation. A more careful comparison would
  acknowledge this.

- **No comparison vs Aardal et al. (Falcon+LaBRADOR) or Anada et
  al. (standard-model lattice synchronized).** Both are 2024 and
  uncited. The PQ-sig+lattice-PoK paradigm is genuinely missing
  from Lemur's §1 trichotomy.

- **Implementation profile coverage.** Both implementations ship
  only the `d=256, k=4, τ=20, N=1024` cell. Other rows in
  `parameter/summary.txt` (τ ∈ {12, 16, 24}, N ∈ {2¹⁵, 2¹⁷, 2²⁰})
  are predicted, not exercised end-to-end.


### <a id="sec-11"></a>11. Reproduction recipe [↑](#toc)

Reproduced on a Debian bookworm container (Node 22 base image),
aarch64 Linux, 11 CPU threads, 8 GiB RAM; passwordless `sudo`. No
Rust, Python, or SageMath in the base image — the first block
installs them. A similar Debian/Ubuntu host with `curl`, `git`,
`sudo`, and ~12 GiB free disk should reproduce the recipe identically.

```sh
# 1. Toolchain (one-time)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y \
    --default-toolchain stable --profile minimal
sudo apt-get install -y build-essential
uv python install 3.11
ln -s ~/.local/bin/python3.11 ~/.local/bin/python3
export PATH="$HOME/.local/bin:$HOME/.cargo/bin:$PATH"

cd submission/code/lemur-py
python3 -m venv .venv && source .venv/bin/activate
pip install numpy pycryptodome

# 2. Cheap checks (~2 min, decisive)
cd submission/code/lemur-rs
cargo test --release            # 52 tests
cargo run --release --bin lemur -- sizes
cd ../parameter && python3 rice_sizes.py

# 3. Python ↔ Rust byte-equivalence (~10 s)
cd ../lemur-py && python3 cli.py vectors --tau 3 --signers 2 --slot 0 \
    --msg "artifact check" --out /tmp/lemur-py-vectors.json
cd ../lemur-rs && cargo run --release --bin lemur -- vectors \
    --tau 3 --signers 2 --slot 0 --msg "artifact check" --out /tmp/lemur-rs-vectors.json

# 4. Main benchmark (~15–30 min, machine-dependent)
cargo run --release --bin bench -- --fast

# 5. Large-N verify (needs ~9 GiB for N=2²⁰; skip on small machines)
cargo run --release --bin bench_verify -- --zero-fixture --n 32768 --reps 3
cargo run --release --bin bench_verify -- --zero-fixture --n 1048576 --reps 1
```

Items 2–3 are sufficient to confirm the *correctness* claim. Item 4
is needed to confirm the *timing* claim. Item 5 is needed only for
the `N=2²⁰` verify cell.


### <a id="footnotes"></a>Footnotes [↑](#toc)

Each footnote cites the source verified during the fact-checking
round. Sources are either a paper location (cited by section,
theorem/lemma number, definition, figure, or table caption — the
identifiers the paper uses), an in-repo `file:line` for the code,
or an external IACR / Springer URL for related work. Some footnotes
quote historical "PDF line N" attributions from `pdftotext` dumps
used by earlier audits — newer work should not rely on those line
numbers; refer to the citation's section/theorem ID instead.

<a id="fn-1"></a>**1.** Reproduced by `submission/code/parameter/rice_sizes.py`
output (Table 2 column / Table 5 corresponding cells); also exact
in `cargo run --release --bin lemur -- sizes` at `N=2¹⁰`. Paper
Table 2 rows "Agg. signature size" = 201.2 / 283.5 / 394.4 KB. [↩](#ref-1) [↩](#ref-1b)

<a id="fn-2"></a>**2.** Lemur paper §1.2 "Technical Overview" (PDF line 189+)
and Figure 1; Squirrel/Chipmunk lineage cited at PDF line 84–85
and §1.1 line 124. The four-block KOTS+HVC+aggregation+BDS08
decomposition is explicit in §6 (Lemur composition) and Figure 5. [↩](#ref-2) [↩](#ref-2b)

<a id="fn-3"></a>**3.** Computed from paper Table 1 cells: 237/201 = 1.179× at
`N=2¹⁰`; (>418)/284 = 1.472× at `N=2¹⁵`; (>831)/394 = 2.109× at
`N=2²⁰`. Independently verified by fact-checker FC1 and reviewer R1. [↩](#ref-3)

<a id="fn-4"></a>**4.** `submission/code/parameter/chipmunk_original_security_summary.txt`
contains 24 rows × 3 RSIS columns. Direct file inspection: min
22.8 (RSIS3, secpar=112, τ ∈ {21,23,24,26}, N=131072); max 39.4
(RSIS1/RSIS2, secpar=112, τ=21, N=1024). For secpar=128 rows
specifically: [23.1, 38.8]. [↩](#ref-4) [↩](#ref-4b) [↩](#ref-4c) [↩](#ref-4d)

<a id="fn-5"></a>**5.** Paper Table 1 caption (PDF line ~63): "Chipmunk:
theoretical, from their scripts. Lemur: measured in implementation
for N=2¹⁰; Rice-encoded on the corresponding cells of Table 5 for
N ∈ {2¹⁵, 2²⁰}." No equivalent disclosure for Chipmunk's encoding
(which is the raw worst-case from Chipmunk's own scripts). [↩](#ref-5) [↩](#ref-5b)

<a id="fn-6"></a>**6.** Paper Table 1, "One-Time Sig" columns. Chipmunk 26 KB
vs Lemur 5.6 KB at `N=2¹⁰` (=4.64×). Chipmunk >110 KB vs Lemur
8.6 KB at `N=2²⁰` (>12.79×). [↩](#ref-6)

<a id="fn-7"></a>**7.** Lemur paper §1.1 (PDF lines 137–139): "Since Chipmunk's
implemented parameters are at a substantially lower security level
(around 40 bits) than ours (128 bits), we are unable to provide a
meaningful runtime comparison against Chipmunk." [↩](#ref-7) [↩](#ref-7b)

<a id="fn-8"></a>**8.** Aardal, Aranha, Boudgoust, Kolby, Takahashi.
"Aggregating Falcon Signatures with LaBRADOR." CRYPTO 2024,
Springer LNCS. eprint [2024/311](https://eprint.iacr.org/2024/311).
Constructs non-interactive aggregation of standard Falcon
signatures via the LaBRADOR lattice proof system. [↩](#ref-8) [↩](#ref-8b) [↩](#ref-8c)

<a id="fn-9"></a>**9.** Anada, Fukumitsu, Hasegawa. "Tightly Secure Lattice-Based
Synchronized Aggregate Signature in Standard Model." ICISC 2024,
Springer LNCS 15596. DOI
[10.1007/978-981-96-5566-3_4](https://link.springer.com/chapter/10.1007/978-981-96-5566-3_4).
First lattice-based synchronized aggregate signature secure in the
standard model (not ROM). [↩](#ref-9) [↩](#ref-9b) [↩](#ref-9c) [↩](#ref-9d)

<a id="fn-10"></a>**10.** Lemur paper §1 (PDF lines 57–90). Names only BLS [3],
hash+SNARK [7], and the lattice-synchronized lineage. Falcon+LaBRADOR
(Aardal et al.) and standard-model lattice synchronized (Anada et al.)
are absent. [↩](#ref-10)

<a id="fn-11"></a>**11.** Lemur paper §3 (Definition 3.1, PDF lines 390–413)
introduces Dual Hint-MLWE; §4 (Figure 3) gives the KOTS construction;
the swap from Chipmunk's *statistical* unforgeability to a
*computational* argument is announced in §1.1 (PDF lines 145–150). [↩](#ref-11)

<a id="fn-12"></a>**12.** Lemur paper §1.1 (PDF lines 155–162) and §5: "we extend
the underlying homomorphic vector commitment (HVC) scheme used to
compress the KOTS public keys. We transition the foundation of the
HVC from the Ring-SIS setting to the Module-SIS setting." [↩](#ref-12)

<a id="fn-13"></a>**13.** Reference [4]: Buchmann, Dahmen, Schneider. "Merkle Tree
Traversal Revisited." PQCrypto 2008, LNCS 5299, 63–78. Cited by
Lemur §7.2 (PDF line 1019) for the stateful signer. [↩](#ref-13)

<a id="fn-14"></a>**14.** Lemur paper Theorem 3.1 statement at PDF line 434:
`Adv^DualHintMLWE ≤ Adv^MLWE + 8m·ε₂`. Proof via Lemmas 3.1–3.4
(four hybrid hops). The `8m·ε₂` is the total statistical slack;
no multiplicative loss. [↩](#ref-14)

<a id="fn-15"></a>**15.** `submission/code/parameter/summary.txt` rows 1–16 at
`d=256`. The `m` column reads 9 (τ=12), 10 (τ=16), 10 (τ=20), 11
(τ=24) for the shipped k=4 family. Row 11 is the implemented cell
(τ=20, N=1024). The paper's §7.1 (PDF lines 933–949) walks an
illustrative d=128 cell with `m=20`. [↩](#ref-15)

<a id="fn-16"></a>**16.** Lemur paper Lemma 3.5 (PDF line 514) reduces to Lemma 2.6
(PDF line 286, titled "Reduction to full-rank sampler"), which in
turn reduces to Lemma 2.5 (PDF line 270, the G+G '23 full-rank
sampler from reference [6]). Appendix D proof (PDF line 2030+)
uses `U^T U = I` (line 2034: "orthonormal columns of U"). [↩](#ref-16)

<a id="fn-17"></a>**17.** Lemur paper Theorem 4.1 (PDF line 561) statement; Lemma
4.1 statement at PDF line 620 with explicit
`Adv^EUF-RK ≤ N(Q+1)² · Adv^DualHintMLWE + Adv^MSIS + …`. The
`Q := Q_H + N` definition is at PDF line 647. Claim 4.2 at PDF line
608 makes the `1/(N(Q+1)²)` guess explicit in the reduction. [↩](#ref-17)

<a id="fn-18"></a>**18.** Lemur paper Theorem B.1 (Appendix B, PDF line ~1380),
combining individual correctness (Lemma B.4), probabilistic
homomorphism (Lemma B.5), robust homomorphism (Lemma B.6), and
position binding (Lemma B.7). [↩](#ref-18)

<a id="fn-19"></a>**19.** Lemur paper §6 (PDF lines 845+) gives the composition;
Figure 5 has the algorithm-level construction; Lemma C.3 covers
unforgeability. [↩](#ref-19)

<a id="fn-20"></a>**20.** Reference [15]: Micali, Ohta, Reyzin. "Accountable-subgroup
multisignatures." CCS 2001, 245–254. Body-text grep confirms [15]
appears only in the `[10, 15]` opening citation (PDF line ~50)
and the bibliography — no role in any theorem, definition, or
construction. [↩](#ref-20) [↩](#ref-20b)

<a id="fn-21"></a>**21.** Lemur paper Lemma 4.1 statement at PDF line 620
includes "such that ℓ = 1, k ≥ 4ℓ". The acknowledgement at PDF
line 545: "Some of our proofs for the KOTS require ℓ = 1, which
is already always the case in our parameter setting. However, we
keep the description general by using the ℓ parameter." [↩](#ref-21)

<a id="fn-22"></a>**22.** Lemur paper Appendix C, Lemma C.3: "Proof. The proof
follows the same idea as in [8]." (PDF line ~1981). [↩](#ref-22)

<a id="fn-23"></a>**23.** Paper Table 2 caption (PDF lines ~67–69): "Representative
Rust implementation performance using (τ=20, N=2¹⁰) parameter
setting; 24 CPU threads." Includes an "Agg. signature size" row
at 201.2/283.5/394.4 KB (Rice-encoded). Paper Table 5 caption (PDF
line ~944): "Comparison of aggregated one-time signature, opening,
and total aggregated sizes (KB) for λ = 128 and root Hermite factor
RHF ≤ 1.0045." Table 5 lists `Sig`, `Open`, `Total` columns at the
worst-case theoretical bound (239 / 331 / 457 KB at τ=20). [↩](#ref-23) [↩](#ref-23b)

<a id="fn-24"></a>**24.** Lemur paper Table 2: "Signing (BDS08)" row = 4.1 ms
(all three N columns); "Batch verification" row = 30.1 ms at
`N=2¹⁰`. [↩](#ref-24)

<a id="fn-25"></a>**25.** Reference [1]: Albrecht, Player, Scott. "On the concrete
hardness of Learning with Errors." J. Math. Cryptol. 9(3) (2015)
169–203. Cited by Lemur §7.1 (PDF line 933) as "the lattice
estimator." [↩](#ref-25)

<a id="fn-26"></a>**26.** `submission/code/lemur-py/kots.py` line 95:
`H[i, i, 0] = 1` inside `_build_H`. The 3-index assignment encodes
the constant polynomial 1 at coordinate `(i,i)` of the `(ℓ × k × d)`
H-tensor. Verified by direct file read. [↩](#ref-26)

<a id="fn-27"></a>**27.** `submission/code/lemur-py/kots.py` lines 154–170:
`vrfy(...)` takes a `beta` parameter; `ivrfy` passes `self.beta_z`,
`svrfy` passes `self.beta_sigma`, `wvrfy` passes `2*self.beta_sigma`. [↩](#ref-27)

<a id="fn-28"></a>**28.** `submission/code/lemur-py/hvc.py` lines 408–412:
`_internal_label` computes `(self.ring.mat_vec(A0, left) +
self.ring.mat_vec(A1, right)) % self.q`. `self.q` is the HVC
modulus (e.g. 9007199254746113 for the shipped cell); the KOTS
modulus is `self.q_prime`. [↩](#ref-28)

<a id="fn-29"></a>**29.** `submission/code/lemur-py/lemur.py` line 294:
`if self.avrfy(pp, pks, t, m, (Z_agg, d_agg, attempt)): return ...`.
The do-while loop body always falls through to `avrfy`, which
verifies both norm bound and the algebraic identity. [↩](#ref-29)

<a id="fn-30"></a>**30.** `submission/code/lemur-rs/tests/` has 5 files:
`bds_stateful.rs` (8 tests), `gauss_ctx.rs` (5), `materialized_tree.rs`
(5), `profile_pipeline.rs` (4), `robustness.rs` (5). Total
integration = 27. Unit tests inside `src/` tally to 25. Doctests = 0.
Grand total: 25 + 27 = 52 across 6 test binaries (the lib's unit
harness + 5 integration files). [↩](#ref-30)

<a id="fn-31"></a>**31.** `cargo run --release --bin lemur -- sizes` output: the
aggregated-sig component lines read `Z_agg (20b, bound=378933) =
5761 B`, `Babai path (Rice k=5) = 52000 B`, `sibling labels (Rice
k=15) = 119600 B`, `u (Rice k=15) = 28704 B`. Sum 206065 B
= 201.236 KB. [↩](#ref-31)

<a id="fn-32"></a>**32.** `submission/code/README.md` last paragraph: "the `lemur
sizes` numbers in the serialized-size table above (e.g. 201.2 KB
at `N=1024`) are the **predicted** encoding lengths. For Rice-coded
components — Babai path, sibling labels, and `u` — the per-coefficient
cost is estimated… `bench --fast` also prints an `Agg Sig Size`
line, but that is the **realised** length of one specific encoded
aggregate, which fluctuates by a few percent around the formula." [↩](#ref-32)

<a id="fn-33"></a>**33.** Reproduced live during the implementation-lens audit's
reviewer pass: `python3 cli.py vectors --tau 3 --signers 2 --slot
0 --msg "artifact check"` on both implementations produces JSON
files that match on every cryptographic key (`pp`, `signatures`,
`ivrfy`, `avrfy`, `agg_attempt`, `aggregate`, per-signer `pk`).
Only the `"implementation"` metadata string differs. [↩](#ref-33)

<a id="fn-34"></a>**34.** `submission/code/lemur-rs/src/aux_ntt.rs` lines 1–45,
including the module docstring describing the CRT-over-two-aux-primes
pipeline. `AUX_P1 = 281_474_976_694_273` and
`AUX_P2 = 281_474_976_690_689` (both near 2⁴⁸) at lines 34, 36. The
Python uses `ring.py`'s schoolbook path when the modulus fails the
NTT condition (top-of-file comment in `lemur-py/profiles.py` explains
the `q' ≡ 17 (mod 32)` splitting condition). [↩](#ref-34)

<a id="fn-35"></a>**35.** Lemur paper §1.1 PDF line 119: "the proposed parameters
correspond to substantially lower concrete security (approximately
40-bit instead of the claimed 112-bit)." [↩](#ref-35)

<a id="fn-36"></a>**36.** `cargo run --release --bin bench -- --sampler-only`
output (measured during the implementation-lens audit): production
indexed CDT 1.7 µs/poly vs generic `GaussCtx` 3.8 µs/poly. Ratio
3.8/1.7 = 2.24×. [↩](#ref-36)

<a id="fn-37"></a>**37.** `submission/code/lemur-rs/src/bin/bench_verify.rs` module
docstring (lines 8–16): "Memory budget at `--n 1048576` under
`D256_K4` (peaks): replicated `pks` Vec ≈ 4 GiB, `concat_pk_bytes(pks)`
≈ 4 GiB, one chunk of randomizers ≈ 8 MiB, per-thread scaled scratch
≈ a few MiB. Peak working set ≈ 9 GiB; budget for ~12 GiB free RAM." [↩](#ref-37)

<a id="fn-38"></a>**38.** `submission/code/lemur-rs/src/sample.rs:sample_cdt_indexed`
(lines 44–65): top-`prefix_bits` of the 32-bit sample index into
`cdt_hi[]` to retrieve `(lo, hi)` window, then binary-search within
that window. `D256_K4_CDT_PREFIX_BITS = 9` per
`src/tables_d256_k4.rs`. [↩](#ref-38)

<a id="fn-39"></a>**39.** `submission/code/lemur-rs/src/aux_ntt.rs` lines 57–69
define `AuxPrime` trait with `const P: u64` and `const C: u64`
and the marker types `P1Marker`, `P2Marker`. Compile-time constant
moduli let LLVM specialise the modular reduction. [↩](#ref-39)

<a id="fn-40"></a>**40.** `submission/code/lemur-rs/src/lemur.rs:lemur_aggregate`
(lines 413–507): `pks.par_iter().zip(sigs.par_iter()).try_for_each
(...lemur_ivrfy...)` at line 423; weighted-Z map-reduce at
lines 446–485; weighted-opening map-reduce at lines 488–493. [↩](#ref-40)

<a id="fn-41"></a>**41.** Lemur paper §7.1 (PDF lines 933–949): "Once `d` and `λ`
are fixed, Constraint 18 (from Table 4) determines `α_w = 31`… We
set the probabilistic homomorphism error to ε_hom = 2⁻¹⁵, which in
turn fixes the number of aggregation attempts to γ = 10 via
Constraint 17… we set `ℓ = 1`, `k = 5`, and `r = 3`… Constraint 14
fixes `α_H = 31`… Constraint 6 then determines individual-signature
norm bound `β_z = 8185`." Walk is for `d=128, N=2²⁰, τ=24`. [↩](#ref-41)

<a id="fn-42"></a>**42.** `submission/code/lemur-py/profiles.py` lines 106–113
(`LemurProfile(name="d256_k4", d=256, tau=20, n_signers=1024,
k=4, ell=1, m=9, n=4, q_prime=3_469_416_721, q=9_007_199_254_746_113,
alpha=87, alpha_h=60, alpha_w=23, beta_z=14_046, ...)`) and the
matching Rust `D256_K4: Profile` static in
`submission/code/lemur-rs/src/profile.rs` lines 300–341. [↩](#ref-42)

<a id="fn-43"></a>**43.** Reference [3]: Boneh, Lynn, Shacham. "Short Signatures
from the Weil Pairing." ASIACRYPT 2001, LNCS, 514–532. BLS12-381
deployment in Ethereum 2 / consensus layer (CL); 48-byte compressed
G1 signatures, 96-byte compressed G2 aggregate. [↩](#ref-43)

<a id="fn-44"></a>**44.** Drake, Khovratovich, Kudinov, Wagner. "Hash-Based
Multi-Signatures for Post-Quantum Ethereum." IACR Comm. Cryptol.
2(1) (2025), 13. eprint
[2025/055](https://eprint.iacr.org/2025/055). [↩](#ref-44)

<a id="fn-45"></a>**45.** Drake et al. "Technical Note: LeanSig for Post-Quantum
Ethereum." eprint
[2025/1332](https://eprint.iacr.org/2025/1332). Extends the Drake
et al. framework. [↩](#ref-45) [↩](#ref-45b)

<a id="fn-46"></a>**46.** Saygan, Gündoğan, Arslan, Gönen. "HAPPIER: Hash-Based,
Aggregatable, Practical Post-quantum Signatures Implemented
Efficiently with Risc0." LightSec 2025, Springer. DOI
[10.1007/978-3-032-15541-2_1](https://link.springer.com/chapter/10.1007/978-3-032-15541-2_1). [↩](#ref-46) [↩](#ref-46b)

<a id="fn-47"></a>**47.** Fleischhacker, Simkin, Zhang. "Squirrel: Efficient
Synchronized Multi-Signatures from Lattices." CCS 2022, 1109–1123.
eprint [2022/694](https://eprint.iacr.org/2022/694). [↩](#ref-47)

<a id="fn-48"></a>**48.** Fleischhacker, Herold, Simkin, Zhang. "Chipmunk: Better
Synchronized Multi-Signatures from Lattices." CCS 2023, 386–400.
eprint [2023/1820](https://eprint.iacr.org/2023/1820). The "5.6×
smaller than Squirrel" claim is from the Chipmunk abstract. [↩](#ref-48)

<a id="fn-49"></a>**49.** Kim, Lee, Seo, Song. "Toward Practical Lattice-Based
Proof of Knowledge from Hint-MLWE." CRYPTO 2023, LNCS 14085, 549–580.
eprint [2023/623](https://eprint.iacr.org/2023/623). Introduces the
Hint-MLWE assumption (MLWE with noisy linear hints `z_i = c_i·s + y_i`)
and reduces it to standard MLWE. [↩](#ref-49)

<a id="fn-50"></a>**50.** Boneh, Kim. "One-Time and Interactive Aggregate Signatures
from Lattices." Stanford TR 2020.
[crypto.stanford.edu/~skim13/agg_ots.pdf](https://crypto.stanford.edu/~skim13/agg_ots.pdf).
Two schemes: public-aggregation OTS (anyone can aggregate; aggregate
size *logarithmic* in number of sigs) and interactive many-time. [↩](#ref-50)

<a id="fn-51"></a>**51.** Re-run on this host with SageMath 9.5
(`sudo apt-get install -y --no-install-recommends sagemath`). All
three estimator scripts in `submission/code/parameter/` produce
outputs that match the committed `.txt` files. (a)
`chipmunk_original.sage` → `chipmunk_original_security_summary.txt`:
byte-identical (`diff` empty), 2m45s. (b) `chipmunk_param.sage` →
`chipmunk_summary.txt`: byte-identical, 13s. (c) `lemur_param.sage`
→ `summary.txt`: OOM in a single end-to-end run on 8 GiB; ran
chunked as 16 sub-runs over `(τ, N) ∈ {12,16,20,24} × {1024, 32768,
131072, 1048576}`, ~25 min total; resulting 16 data rows are
field-identical to the committed `summary.txt` (verified by Python
diff keyed on `(τ, N)`). [↩](#ref-51) [↩](#ref-51b) [↩](#ref-51c)


*Footnotes were generated by a fact-check round (three lens-specific
fact-checker subagents plus three reviewer subagents), then extended
with footnote 51 after the Sage parameter-estimator outputs were
independently reproduced from source. For the per-claim ledger, see
`tmp/fact-checking/fact_checker_{1,2,3}.md` and the reviewer reports
in the same directory. 197 atomic claims were checked; this footnote
set covers the load-bearing source citations.*
