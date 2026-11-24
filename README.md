# WorldDynamics

**Reliable Action-Conditioned World Models for Long-Horizon Physical Prediction**

WorldDynamics is an auditable research codebase for predicting physical state and video under interventions. Its central question is whether a model can distinguish futures caused by alternative actions from the **same** initial physical state, remain stable over long rollouts, and support reliable closed-loop decisions. The historical Python package is `iwm`; the original sparse-intervention prototype and all documented negative results remain part of the project history.

## Research scope and implemented baselines

- **Physical replay:** MuJoCo cube scenes with exact state snapshot/restore, seven paired force branches, position, velocity, contact, calibrated RGB camera, artifact hashes, and scene/physics-disjoint evaluation families.
- **State dynamics:** action-blind, action-conditioned, paired-difference and shuffled-pair controls, plus an RSSM with a causal prior/posterior boundary. A future-observation leakage issue in the earlier RSSM was fixed and covered by regression tests.
- **Long-horizon evaluation:** 64-step teacher-forced and free-running state prediction, same-state counterfactual error, physics/geometry OOD tests, horizon curves, and an approximate floor-contact proxy.
- **Video prediction:** a compact from-scratch latent flow-matching model with reconstruction, teacher-forced prediction, autoregressive generation, action-blind controls, and shared-noise counterfactual evaluation. It uses a deterministic frame codec; it is not a Video VAE or a large Video DiT.
- **World/action models:** lightweight visual world and inverse-action heads, trained independently and jointly. These are exploratory WAM baselines, not a validated WAM contribution.
- **Planning:** MuJoCo closed-loop random, physics-based greedy Oracle, learned MPC, and learned CEM pilots. Model planners select actions using current observations and model predictions; the privileged Oracle is reported separately.

## V2 empirical evidence and limitations

The V2 dataset contains **72 scenes, 504 same-state action branches, and 64-step trajectories**. Eight state-model arms were trained with three seeds each. The independently sealed confirmation family contains eight scenes.

The predeclared primary statistic (blind minus conditioned final-step counterfactual-delta MSE) was **−5.405 m²**, with a scene-bootstrap 95% interval of **[−6.862, −3.926] m²**. This is a **negative result**: the action-conditioned baseline did not outperform the action-blind baseline on the primary test. The eight-scene confirmation is a pilot, not evidence of broad physical-world generalization.

In the held-out planning pilot, the greedy physics Oracle succeeded on 5/8 scenes per seed, while conditioned-model MPC achieved 1/8, 0/8, and 0/8 across the three seeds. The task has unresolved feasibility limitations; no learned-control success claim is supported. The single-seed video and joint world/action experiments are negative or inconclusive, and pixel PSNR alone does not establish action fidelity.

**Acceptance status:** L1 research engineering is locally demonstrated for state-model data, training, checkpoint loading, independent evaluation, and prediction. L2 is partial: multi-seed and OOD state experiments exist, but action causality and planning failed; video/WAM remain single-seed. L3 research science is **not passed**. No novel algorithm, SOTA, real-robot, official V-JEPA 2-AC, or large-GPU scaling result is claimed.

See the [V2 baseline and confirmation report](docs/V2_BASELINE_REPORT.md), [long-horizon report](docs/V2_LONG_HORIZON_REPORT.md), [video report](docs/V2_VIDEO_REPORT.md), [WAM report](docs/V2_WAM_REPORT.md), and [planning report](docs/V2_PLANNING_REPORT.md). Historical [V1.1 audit](docs/V11_AUDIT_REPORT.md) and [V1.1 baseline results](docs/V11_BASELINE_RESULTS.md) remain available.

## LeWorldModel (LeWM) G0 integration audit

This repository also preserves a separate, evidence-gated integration/preflight track for the upstream [LeWorldModel project](https://github.com/lucas-maes/le-wm), pinned to commit `8edfeb336732b5f3ce7b8b210d0ba370a09e2cac`. No upstream model source is copied into this repository.

As of the **2026-10-15 G0 audit**, a fixture passed the `stable-worldmodel` Loader contract and a bounded remote Lance probe read two scalar rows and one 224×224 RGB image from a pinned public candidate dataset revision. This does **not** establish that the full official dataset loads through the LeWM Loader or is equivalent to the published HDF5 data. Full-dataset loading, GPU training, checkpoint production, and independent model evaluation remain unverified; the recorded status is `BOUNDED_LANCE_READ_VERIFIED_GPU_BLOCKED`.

Key materials:
- [G0 baseline audit](docs/V2_BASELINE_AUDIT.md)
- [G0 research contract](docs/V2_RESEARCH_CONTRACT.md)
- [G0 implementation plan](docs/V2_IMPLEMENTATION_PLAN.md)
- [LeWM runbook](docs/G0_LEWM_RUNBOOK.md)
- [G0 preflight report](reports/G0_PRECHECK_2026-10-15.md)
- [Pinned source/data record](configs/lewm/g0_source_lock.json)
- [OpenSpec change](openspec/changes/world-model-v2-g0/proposal.md)

## Original prototype and research history

The early sparse-intervention prototype includes an auditable threshold baseline, abstention for underdetermined actions, graph-theoretic error certificates, and paired-replay contracts. These are retained as historical baselines and analysis tools, not presented as a novel world-model algorithm. The simplified RGB experiments did not establish that paired-branch supervision beats independent supervision. See [research audit](docs/research_audit.md), [theory compendium](docs/THEORY_COMPENDIUM.md), and [research proposal](docs/RESEARCH_PROPOSAL.md).

## Install and reproduce

The package requires Python 3.10 or newer. The documented V2 runs used Python 3.12 with CPU-only PyTorch and MuJoCo; CUDA/GPU training was not run for those results.

```bash
python -m pip install -e ".[test,research,visualization]"
python -m pytest -q
ruff check src/iwm/data.py src/iwm/evaluation.py src/iwm/models.py src/iwm/visual_models.py
```

Use **new output directories** for every experiment:

```bash
python scripts/generate_v2_data.py --output artifacts/my-v2-data --train-scenes 32 --eval-scenes 8 --horizon 64
python scripts/validate_data.py artifacts/my-v2-data
python scripts/run_v2_baselines.py --dataset artifacts/my-v2-data --output artifacts/my-v2-baselines --seeds 17 29 43 --epochs 10
python scripts/evaluate_v2_matrix.py --dataset artifacts/my-v2-data --runs artifacts/my-v2-baselines --output results/my-v2-exploratory --families iid_test physics_ood geometry_ood
```

Run the `confirmatory` family **once only after** model, metric, and analysis choices are frozen. Follow [the preregistered protocol](docs/CONFIRMATORY_PROTOCOL.md). Generated datasets, checkpoints, raw per-scene results, GIFs, and local environments are intentionally excluded from Git; regenerate them locally using the documented scripts and manifests.
