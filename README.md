# Hackaton world model

**Yentl Collin · Hack the World · JEPA world modeling and planning**

Hackathon work on an action-conditioned world model that predicts multiple
future latent states in one forward pass, then uses self-speculative rollouts
inside model-predictive control (MPC). Future trajectories are drafted in
parallel, checked by the same predictor in one-step mode, and corrected from
the first rejected step.

## What I implemented

- A causal multi-horizon action-conditioned predictor, with masks that expose
  only the action prefix needed for each future state.
- Direct prediction of several future latent states in a single forward pass.
- Latent trajectory drafting, one-step verification, prefix acceptance and
  regeneration after a mismatch.
- Integration into the MPPI planning loop and a dedicated speculative config.

## Read the code

| File | Role |
|---|---|
| [architectures.py](eb_jepa/architectures.py) | `CausalMultiHorizonPredictor` and its causal attention mask. |
| [jepa.py](eb_jepa/jepa.py) | Direct multi-horizon inference and `_self_speculative_unroll`. |
| [planning.py](eb_jepa/planning.py) | MPC/MPPI integration and rollout-mode selection. |
| [main.py](examples/ac_video_jepa/main.py) | Training entry point. |
| [Speculative MPPI config](examples/ac_video_jepa/cfgs/planning_speculative_mppi.yaml) | Planning horizon, verification metric and acceptance threshold. |
| [Tests](tests/test_jepa_output_formats.py) | Multi-horizon masks, prediction shapes, losses and speculative rollout. |

[Technical explanation — PDF](rapport/main.pdf) · [LaTeX source](rapport/main.tex)

## Run

The original project targets **Python 3.12**. From the repository root:

```bash
uv sync
uv run pytest tests/test_jepa_output_formats.py -k 'direct_multi_horizon or self_speculative'
```

Start from [the training config](examples/ac_video_jepa/cfgs/train.yaml).
For speculative planning, select
`examples/ac_video_jepa/cfgs/planning_speculative_mppi.yaml` through the
training/evaluation configuration's `eval.plan_cfg_path` field. A trained
checkpoint is needed for meaningful planning; no model weights are bundled.

## Speed measurement

The implementation provides the draft/verify mechanism. Saved measurements
establishing a particular wall-clock speedup are not included in this copy.
The [LeWM branch](https://github.com/YentlCollin/Hackaton-world-model/tree/lewm)
contains a timed latent-rollout benchmark. It measures autoregressive versus
self-speculative prediction, not complete MPC episode time. The gain depends
on horizon, acceptance threshold, batch size, hardware and model quality.

## Origin and attribution

This project builds on **EB-JEPA**, the Meta FAIR community implementation,
and the collaborative [Gklajer/eb_jepa](https://github.com/Gklajer/eb_jepa)
repository. The upstream license and author metadata are retained; the base
library is not claimed as my own work.

The copied MPC snapshot is commit
[`3c674747`](https://github.com/Gklajer/eb_jepa/commit/3c674747d9687809ac5f4163ab4b7c94a0fe3ad2).
My relevant contributions are recorded in
[the multi-horizon predictor commit](https://github.com/Gklajer/eb_jepa/commit/7ab3c44bfa5174e0dc5d3922394f35c9be822edc)
and [the self-speculative MPC commit](https://github.com/Gklajer/eb_jepa/commit/3c674747d9687809ac5f4163ab4b7c94a0fe3ad2).

Code license: [Apache 2.0](LICENSE.md).
