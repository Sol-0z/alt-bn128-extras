# Changelog

All notable changes to `alt-bn128-extras` (reference implementations
+ benches for two proposed Solana syscalls) will be documented here.

## [0.1.0] — 2026-05-11

Initial public release. Reference impls and Mollusk bench grids
backing a SIMD-XXXX proposal for two new BN254 syscalls. Status:
research / proposal companion; not deployed on-chain.

### Reference implementations

- `crates/g1-msm-ref` — `alt_bn128_g1_msm(scalars, points) →
  bn254::G1`. Pippenger algorithm; window size auto-chosen by
  scalar count. Pure-Rust no_std-compatible reference for the
  proposed syscall semantics.
- `crates/fr-batch-inv-ref` — `alt_bn128_fr_batch_inverse(scalars)
  → Vec<Fr>`. Montgomery batched-inverse trick; same one-inverse-plus-
  3N-mults complexity used inside SHPLONK's rotation-set evaluation
  in `halo2-solana-verifier`.

### Bench grids

- `programs/g1-msm-bench` — Mollusk SVM bench BPF program. Sweeps
  MSM size {2, 4, 8, 16, 32, 64, 128} and reports CU under the
  current syscall surface (manual G1 add + scalar-mul loop) vs the
  proposed batched syscall. Numbers feed the SIMD proposal's
  motivation section.

### SIMD proposal companion

The `docs/SIMD-XXXX.md` draft (in this repo) outlines the proposed
syscall semantics, gas pricing arguments, and reference vectors. Both
reference impls are bit-for-bit deterministic against arkworks 0.5
output and intended as conformance fixtures.

### Known gaps

- Not a Solana feature proposal yet — pending Solana Foundation
  feedback before opening a SIMD PR upstream.
- `fr-batch-inv` reference covers only Fr (scalar field). Fq (base
  field) variant deferred — no current verifier path needs it.

[0.1.0]: https://github.com/nzengi/alt-bn128-extras/releases/tag/v0.1.0
