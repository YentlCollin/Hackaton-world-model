# Hackaton world model — LeWM experiments

**Yentl Collin and collaborators · Hack the World**

This branch preserves the LeWorldModel (LeWM) action-conditioned JEPA
experiment: multi-token future prediction, self-speculative latent rollout,
training, representation validation and a timed speed benchmark.

For the version integrated into the MPC planner, use
[the main branch](https://github.com/YentlCollin/Hackaton-world-model/tree/main).

## Read the code

- [eb_jepa/lewm.py](eb_jepa/lewm.py): encoder, action-conditioned MTP heads,
  drafting, latent verification, prefix acceptance and correction.
- [Training](examples/lewm/main_lewm.py) and [configurations](examples/lewm/cfgs/).
- [Validation](examples/lewm/val_lewm.py): position probes and rollout error.
- [Timed benchmark](examples/lewm/bench_speculative.py): AR versus speculative
  latent rollout on the same checkpoint and inputs.
- [Tests](tests/test_lewm_speculative.py): prefix acceptance and rollout behavior.

## Install and run

The original project targets Python 3.12:

```bash
uv sync
uv run pytest tests/test_lewm_speculative.py

uv run python -m examples.lewm.main_lewm \
  --fname examples/lewm/cfgs/lewm_two_rooms.yaml \
  --folder runs/lewm
```

After training an MTP checkpoint:

```bash
uv run python -m examples.lewm.bench_speculative \
  --ckpt runs/lewm/latest.pth.tar \
  --out runs/lewm/speculative_speedup.pdf --repeats 30 --threshold 0.1
```

## Interpreting the benchmark

The timing script warms up the predictor, synchronizes CUDA and averages
repeated measurements at several rollout horizons. Speedup is measured as
`AR time / speculative time`. No fixed speedup is asserted by this repository;
saved benchmark results and checkpoints are not included.

The benchmark times latent rollouts, not a complete MPC episode. It does not
measure planning success or rollout quality; use the validation scripts as well.
The optional acceleration animation uses an idealized full-acceptance schedule,
which is an illustration rather than measured wall-clock evidence.

## Origin

Copied from [Gklajer/eb_jepa](https://github.com/Gklajer/eb_jepa), snapshot
[`90aad8db`](https://github.com/Gklajer/eb_jepa/commit/90aad8db324b0dd5ab233bcc2f49879a3b11e5ef),
built on EB-JEPA by Meta FAIR. Upstream authorship and [Apache 2.0](LICENSE.md)
are retained. Only this experiment, its package dependencies and tests are
selected; cluster-specific launchers are omitted.
