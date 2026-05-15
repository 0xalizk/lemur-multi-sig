<br>

<details><summary><b> &nbsp; 788k tokens 🦞  &nbsp;&nbsp;&nbsp;  ~2h45m 🕒 </b></summary>
<br>
🟩 benchmarks rust/py · 🟦 Sage estimator · 🟪 audits & reviews · ⬜ setup <br><br>

| |  |  |
| :-: | --- | ---: |
| ⬜ | Toolchain install (rustup + Python + Sage incl. retries) | ~15 min |
| 🟩 | Cargo build + 52-test suite | ~2 min |
| 🟩 | `lemur sizes` + `rice_sizes.py` | < 1 s |
| 🟩 | Python ↔ Rust `vectors` byte-equivalence check | ~10 s |
| 🟩 | `bench --fast` (multiple runs across session) | ~50 min |
| 🟩 | `bench_verify --zero-fixture` at N ∈ {1024, 8192, 32768} | ~5 min |
| 🟦 | Sage `chipmunk_param.sage` | 13 s |
| 🟦 | Sage `chipmunk_original.sage` | 2 min 45 s |
| 🟦 | Sage `lemur_param.sage` (chunked by 16 (τ, N) cells) | ~25 min |
| 🟪 | 6 audit subagents (3 audit + 3 review, run in parallel batches) | ~25 min |
| 🟪 | 6 fact-check subagents (3 FC + 3 review, run in parallel batches) | ~30 min |
|   | **Total compute walltime** | **~2 h 45 min** |

</details>

<br>

### Lemur — [paper](submission/report.pdf), [code](submission/code/), and [review](assessment/review.md)

<br>

<pre>
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
                              🐒 <b>Lemur '26</b>
</pre>

### What the paper delivers: 

1. **Lemur is a synchronized post-quantum multi-signature** that
   non-interactively aggregates `N` signatures generated for the same
   message at the same time slot, with `N` up to 2²⁰ (≈ 1 M signers).
2. **Aggregate-signature sizes:** 201 KB at `N = 2¹⁰`, 284 KB at
   `N = 2¹⁵`, **394 KB at `N = 2²⁰`** — Rice-encoded; the worst-case
   theoretical bound is 239 / 331 / 457 KB respectively.
3. **Timings (24-thread baseline):** stateful sign 4.1 ms, batch
   verification 30.1 ms at `N = 2¹⁰`, 812 ms at `N = 2¹⁵`,
   25.3 s at `N = 2²⁰`; secure aggregation 567 ms at `N = 2¹⁰`,
   linear-extrapolated to ≈ 12 min at `N = 2²⁰`.
4. **New computational assumption:** Dual Hint-MLWE, with a tight
   reduction from standard MLWE (`8m·ε₂` statistical slack, no
   multiplicative loss). Lemur's KOTS unforgeability rests on
   Dual Hint-MLWE + MSIS in the ROM.
5. **Security recomputation of Chipmunk:** under modern
   Dilithium-style lattice estimation, Chipmunk's published 112-bit
   parameters realize **22.8–39.4 bits of core-SVP security** across
   the 24 (τ, ρ, λ) cells in its parameter table — the paper's
   "approximately 40-bit" headline is the max-over-instances reading.
   At a corrected 128-bit security level, Chipmunk's aggregate would
   roughly double in size; Lemur at the same target is ~2.1× smaller
   at `N = 2²⁰`.

### What the assessment found: 

1. **Implementation matches the paper.** Every deterministic claim
   reproduces: [aggregate sizes (201.2 / 283.5 / 394.4 KB exact)](assessment/review.md#sec-5-3),
   algorithm-level structure ([Python ↔ Rust byte-equivalent on
   `vectors`](assessment/review.md#sec-5-4)),
   [all 52 cargo tests pass](assessment/review.md#sec-5-1).
   [Audit checkpoints](assessment/review.md#sec-4-4) in
   `kots.py` / `hvc.py` / `lemur.py` all correctly implemented.
2. **[Sage estimator outputs reproduce from source.](assessment/review.md#sec-5-6)**
   `chipmunk_original.sage` and `chipmunk_param.sage` produce
   byte-identical outputs to the committed files; `lemur_param.sage`
   (chunked by `(τ, N)` to fit 8 GiB) produces a `summary.txt`
   field-identical to the committed file across all 16 cells. The
   ["40-bit Chipmunk" finding](assessment/review.md#sec-5-5) is independently reproducible.
3. **[Timings track the paper proportionally.](assessment/review.md#sec-6)**
   On 11 threads versus the paper's 24-thread baseline, every measured operation lands
   within ±15 % of the expected linear thread-count slowdown.
4. **[Three framing caveats](assessment/review.md#sec-9-2)** worth flagging:
   - Table 1 places Rice-encoded Lemur next to raw-encoded Chipmunk
     (~14 % asymmetry, disclosed in the caption but not column
     headers).
   - "Order of magnitude" KOTS shrink against Chipmunk is 4.64× at
     `N = 2¹⁰`, ~12.8× at `N = 2²⁰` — only the favorable corner
     reaches the headline.
   - [No runtime comparison vs Chipmunk](assessment/review.md#sec-10) (acknowledged in §1.1: "we
     are unable to provide a meaningful runtime comparison").
5. **[Two adjacent paradigms uncited](assessment/review.md#sec-9-1):**
   Aardal et al. *"Aggregating
   Falcon Signatures with LaBRADOR"* (CRYPTO 2024) — a PQ-sig +
   lattice-PoK paradigm Lemur's introduction trichotomy does not
   contain; and Anada-Fukumitsu-Hasegawa *"Tightly Secure
   Lattice-Based Synchronized Aggregate Signature in Standard Model"*
   (ICISC 2024) — a direct sibling in the synchronized lattice column
   with a strictly stronger security model.
6. **[Reduction loss not absorbed in parameter selection.](assessment/review.md#sec-10)**
   Lemma 4.1's
   `N(Q+1)²` multi-user loss is `~2¹⁴⁰` at `N = 2²⁰, Q_H = 2⁶⁰`;
   absorbing it would require ~268-bit core hardness. Lemur picks
   parameters at the 128-bit core-SVP level. Community-standard
   practice for lattice multi-sigs (Dilithium does the same) but
   worth flagging.
<br>

## [`Full Review`](assessment/review.md)

<br>
