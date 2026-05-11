# alt-bn128-extras

Reference implementations + Mollusk bench grids for two proposed
Solana BPF syscalls. Part of the [Sol-0z Protocol](../) workspace,
but **independent of the verifier** — these are direct contributions
to Solana's BN254 syscall surface.

```
crates/
  g1-msm-ref/             Pippenger reference impl for the G1 MSM SIMD oracle
  fr-batch-inv-ref/       Montgomery batch-inverse reference (60 lines, 7/7 tests)

programs/
  g1-msm-bench/           Mollusk CU bench grid:
                            sequential syscall vs proposed-SIMD model
                            vs pure-BPF Pippenger vs naive

docs/
  simd-proposals/
    simd-XXXX-alt-bn128-g1-msm.md           formal SIMD draft
    simd-XXXX-alt-bn128-fr-batch-inverse.md formal SIMD draft
    PR-BODY.md                              PR body for the g1-msm submission
```

## Status

Both SIMD drafts follow the structure of [SIMD-0302
(BN254 G2 syscalls)](https://github.com/solana-foundation/solana-improvement-documents/blob/main/proposals/0302-bn254-g2-syscalls.md).
Discussion thread for the G1 MSM draft is at
[solana-improvement-documents#535](https://github.com/solana-foundation/solana-improvement-documents/discussions/535).

Neither has been filed as a formal SIMD-XXXX PR yet.

## How the case relates to the verifier

The verifier in [`halo2-solana-verifier`](../halo2-solana-verifier/)
ships a **software** Montgomery batch inverse in `kzg::shplonk` (the
v2.1 "Path B" refactor) — the same algorithm the
`alt_bn128_fr_batch_inverse` draft proposes, but in pure BPF Fr math.
This already let every reference circuit fit the 1.4 M per-tx cap.

The native syscall versions would still help:

- `alt_bn128_g1_msm` — replaces N sequential
  `alt_bn128_g1_multiplication_be` calls in `kzg::shplonk::finalize_shplonk_pairs`.
  Projection: −15 to −20 % total verify CU; would fold large circuits
  back into a single tx.
- `alt_bn128_fr_batch_inverse` — would beat the software path because
  per-Fr-mul on BPF is ~3 k CU vs the projected `4 000 + n × 200` for
  a syscall. Additional saving: ~400 k CU on Fibonacci-shape circuits.

## Quick start

```bash
# Reference impls — host-side unit tests.
cargo test -p g1-msm-ref      # 11/11
cargo test -p fr-batch-inv-ref # 7/7

# Mollusk bench grid (sequential syscall vs proposed-SIMD model).
cargo build-sbf --manifest-path programs/g1-msm-bench/Cargo.toml --features bpf-entrypoint
cargo test -p g1-msm-bench-program --test cu_grid -- --nocapture
```

Tests: 1 / 1 (the Mollusk grid; reference crate tests run with `-p`).

## License

MIT OR Apache-2.0. See `LICENSE-MIT` and `LICENSE-APACHE`.
