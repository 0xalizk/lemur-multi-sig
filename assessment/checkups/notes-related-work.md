# Working notes — Lemur related work survey

Source: Lemur paper (`/workspace/repo/submission/report.pdf`) plus abstracts fetched
in May 2026. Used to draft the "background" skill.

## Per-reference taxonomy (Lemur's 17 cited works)

For each ref I note: **role in Lemur**, **paper kind**, **where it's
cited in Lemur's body**.

| # | Authors / Year | Lemur's body-text use | Role |
| - | --- | --- | --- |
| [1] | Albrecht-Player-Scott '15 | "lattice estimator [1]" §7.1 | Foundational tool — LWE/MLWE concrete hardness estimator |
| [2] | Boneh-Kim '20 | "Lyubashevsky-Micciancio [13] and Boneh-Kim [2]" §4 | Adjacent scheme — non-synchronized lattice aggregate sig |
| [3] | Boneh-Lynn-Shacham '01 | "BLS signature scheme [3]" §1 | Comparison baseline — pairing-based, deployed but pre-quantum |
| [4] | Buchmann-Dahmen-Schneider '08 (BDS08) | "BDS08 Merkle traversal algorithm [4]" §7.2 | Foundational tool — stateful signing optimization |
| [5] | Buser et al. '22 (PQ-VRF) | "As shown in [5]" §1.1 | Adjacent primitive — PQ VRF from sym primitives; future-work pointer |
| [6] | Devevey-Passelègue-Stehlé '23 (G+G) | Lemma 2.5 cites [6] | Foundational tool — full-rank Gaussian sampler |
| [7] | Drake-Khovratovich-Kudinov-Wagner '25 | "Existing proposals [7]" §1 | Competing paradigm — hash-based PQ multi-sig via SNARKs |
| [8] | Chipmunk (Fleischhacker-Herold-Simkin-Zhang '23) | Throughout: definitions, theorems, Prop B.2/B.3, etc. | **Direct predecessor** — comparison baseline |
| [9] | Squirrel (Fleischhacker-Simkin-Zhang '22) | "Squirrel [9] and its successor Chipmunk [8]" §1 | **Conceptual ancestor** — first synchronized lattice multi-sig |
| [10] | Itakura-Nakamura '83 | First reference at §1 opening | Historical — origin of "multi-signature" |
| [11] | Kim-Lee-Seo-Song '23 (Hint-MLWE) | "Kim et al. [11] introduced the Hint-MLWE…" §3 | **Theoretical ancestor** — Lemur's Dual Hint-MLWE generalises |
| [12] | Lyubashevsky '12 | Lemma A.5 | Foundational scheme — Fiat-Shamir with aborts framework |
| [13] | Lyubashevsky-Micciancio '08 | "KOTS… arising from [13]" §1.1 | **Conceptual ancestor** — first compact lattice OTS |
| [14] | Lyubashevsky-Seiler '18 | Lemma A.2 / Lemma 4.2 | Foundational tool — invertibility in partially splitting cyclotomics |
| [15] | Micali-Ohta-Reyzin '01 | (Not cited in body that I could find) | Adjacent model — accountable-subgroup multisig definition |
| [16] | Micciancio-Regev '04 | Lemmas 2.3, 2.4, Def 2.1 | Foundational tool — worst→avg-case via Gaussian smoothing parameter |
| [17] | Peikert '10 | Lemma 2.1, Lemma 2.5 | Foundational tool — efficient/parallel Gaussian sampler |

## Classification by role

- **Direct lineage (read these first when surveying the field):**
  - [13] Lyubashevsky-Micciancio '08 — KOTS conceptual root
  - [12] Lyubashevsky '12 — Fiat-Shamir with aborts (Dilithium ancestor)
  - [9] Squirrel '22 — first synchronized lattice multi-sig
  - [8] Chipmunk '23 — second-gen, Lemur's direct baseline
  - [11] Hint-MLWE '23 — Lemur's theoretical lever

- **Competing paradigms:**
  - [3] BLS '01 — pairing-based, deployed in Ethereum/Dfinity, not PQ
  - [7] Drake et al. '25 — hash-based with pqSNARK aggregation
  - [2] Boneh-Kim '20 — lattice aggregate (non-synchronized)

- **Foundational tools (cite-and-move-on; don't try to re-derive):**
  - [1] APS estimator
  - [4] BDS08 stateful traversal
  - [6] G+G Gaussian sampleability
  - [14] Lyub-Seiler invertibility
  - [16] Micciancio-Regev smoothing
  - [17] Peikert Gaussian sampler

- **Adjacent / historical:**
  - [5] PQ VRF (future-extension hook)
  - [10] Itakura-Nakamura '83 (origin)
  - [15] MOR '01 (accountable model — out of scope for Lemur)

## Key facts I verified

### Squirrel '22 [9] (eprint 2022/694, CCS '22)
- First synchronized lattice multi-sig.
- Ring-SIS based.
- Bounded number of time slots `2^τ`.
- Non-interactive aggregation, ROM, rogue-key safe.
- Lemur quotes: "Squirrel and its successor Chipmunk."

### Chipmunk '23 [8] (eprint 2023/1820, CCS '23)
- Improved Squirrel: **5.6× smaller** aggregate signature at the same
  parameters.
- 8192 signatures aggregate to **~136 KB at 112-bit security** (per Chipmunk's
  authors' claim).
- **BUT** Lemur's recomputation under Dilithium-style estimation puts
  the *actual* security at ~40 bits — see
  `parameter/chipmunk_original_security_summary.txt`.
- Still Ring-SIS based.
- Defines the KOTS/HVC primitives Lemur reuses (Def 4.1, Def B.2-B.5,
  Prop B.2, Thm 25 — all cited by Lemur).

### Drake et al. '25 [7] (eprint 2025/055, CiC '25)
- The competing PQ paradigm: hash-based.
- **Components:** Winternitz One-Time Signatures (WOTS+ chains) +
  Merkle trees + post-quantum SNARK (pqSNARK).
- The SNARK is what enables non-interactive aggregation — without it
  hash-based signatures cannot natively aggregate.
- Targets Ethereum proof-of-stake validator signatures (~10⁶ signers).
- Hard target: signatures under 4 KiB per slot.
- Has a successor "LeanSig" (eprint 2025/1332).
- Lemur's critique: "heavy proof machinery and substantial
  computational overhead, which makes it difficult to deploy in
  latency-sensitive consensus settings."

### Hint-MLWE '23 [11] (Kim-Lee-Seo-Song, CRYPTO '23, eprint 2023/623)
- Variant of MLWE where the distinguisher additionally sees a
  bounded number of *noisy linear hints* about the secret.
- Has an efficient reduction *to* standard MLWE — i.e., Hint-MLWE
  hardness follows from MLWE hardness, with a quantitative loss.
- Originally introduced for zero-knowledge proof-of-knowledge
  protocols for RLWE encryption and BDLOP commitments without
  noise flooding or rejection sampling.
- **Lemur's twist (Dual Hint-MLWE):** the hints in Lemur's KOTS are
  *noise-free* linear images of the secret (not Kim et al.'s noisy
  hints), AND the secret appears on opposite sides of the public key
  and signature computation (`T = SA` vs `Z = HS`). This structural
  mismatch is why Lemur defines a *new* assumption (Dual Hint-MLWE)
  and provides a fresh reduction from MLWE in Theorem 3.1.

### Boneh-Kim '20 [2] (Stanford TR / Semantic Scholar 515b4a…)
- Two schemes, both lattice-based on SIS:
  1. Public-aggregation OTS (anyone can aggregate; based on
     Lyubashevsky-Micciancio one-time-sig structure).
  2. Interactive many-time aggregate (signers participate in agg).
- Aggregate size is at most **logarithmic** in number of signatures.
- *Not synchronized* — distinct paradigm from Squirrel/Chipmunk/Lemur.
- Lemur cites [2] only as conceptual context for the KOTS, not as a
  baseline.

### Lyubashevsky-Micciancio '08 [13] (TCC '08)
- Asymptotically efficient lattice-based digital signatures.
- The compact-OTS line that Chipmunk's (and Lemur's) KOTS descends from.
- Important: Lemur's KOTS *deviates* from this line by using a
  computational (Dual Hint-MLWE) argument, where [13] used statistical.

### Lyubashevsky '12 [12] (EUROCRYPT '12)
- "Lattice signatures without trapdoors" — the Fiat-Shamir with aborts
  framework underpinning Dilithium and most modern lattice signatures.
- Used by Lemur only in a tail bound (Lemma A.5).

### BDS08 [4] (Buchmann-Dahmen-Schneider, PQCrypto '08)
- Standard textbook Merkle tree traversal optimization.
- Signer keeps `O(τ)` state (the "authentication path" + treehash
  workers) instead of `O(2^τ)` (whole tree).
- Used by Lemur's stateful signer (default mode in `lemur-py`/`lemur-rs`).

### BLS '01 [3]
- Boneh-Lynn-Shacham short signatures from the Weil pairing.
- 48-byte aggregates, currently deployed in Ethereum and Dfinity for
  PoS validator aggregation.
- Broken by Shor's algorithm (discrete log in pairing-friendly group).

## Paradigm map (consolidated)

```
                       Multi-signatures
                              │
        ┌─────────────────────┼─────────────────────────────┐
        │                     │                             │
  Pairing-based         Hash-based + SNARKs            Lattice-native
        │                     │                             │
       BLS [3]          Drake et al. [7]                    │
   (Ethereum,          (PQ Ethereum,                        │
    deployed,           Winternitz +                        │
    not PQ)             Merkle + pqSNARK)                   │
                                                            │
                              ┌─────────────────────────────┤
                              │                             │
                   Synchronized (slot schedule)    Non-synchronized
                              │                             │
                              ▼                             ▼
                       Squirrel '22 [9]              Boneh-Kim '20 [2]
                              │                       (OTS + interactive)
                              ▼
                       Chipmunk '23 [8]
                              │
                              ▼
                          Lemur '26
```

Within the synchronized lattice column, the build-blocks are always:

- **KOTS** (one-time signature, key-homomorphic)
- **HVC** (homomorphic vector commitment, a Merkle tree of KOTS pks)
- **Aggregation** (rogue-key-safe weighted sum with RO-derived weights)
- **Stateful signing** (BDS08, amortizes the O(2^τ) cost)

## Themes that recur across the lineage

1. **The trade between statistical and computational arguments.**
   Squirrel and Chipmunk's KOTS used statistical
   indistinguishability, which forced large parameters. Lemur swaps
   to a computational argument (Dual Hint-MLWE) for ~10× smaller
   KOTS. Future schemes in this line will likely continue this
   shift; any "we beat Lemur" claim that does *not* address the
   underlying assumption is suspect.

2. **Ring-SIS vs Module-SIS.** Squirrel/Chipmunk are Ring-SIS;
   Lemur moves the HVC to Module-SIS, gaining parameter flexibility
   without changing the asymptotic security argument. The cost is
   slightly more complex implementation (matrix HVC ops vs vector).

3. **Security estimation as a moving target.** Chipmunk's parameters
   look fine under the estimator the authors used; under modern
   Dilithium-style estimation they collapse to ~40 bits. Any future
   paper in this lineage should be evaluated under the *latest*
   estimator [1], not whichever one the authors chose.

4. **Aggregation always has a probabilistic correctness gap.** The
   weighted-sum may exceed norm bounds with probability `ε_hom`;
   the standard fix is `γ` retries with fresh randomizer draws.
   Squirrel, Chipmunk, and Lemur all do this; only the constants
   differ.
