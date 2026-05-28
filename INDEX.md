# data/ INDEX

Last refreshed 2026-05-27.

Top-level pointer file for the `data/` submodule. Each section maps a
directory to the script that produced it, the paper section that
consumes it, and a one-line headline. Full per-file inventory with
status flags lives in
`../Butterfly_Knife/core/10 - Data Inventory.md` (sections A-AF).

## Quick map: paper section -> data directory

| Paper section | Data directory | Headline number |
|---------------|----------------|-----------------|
| MAIN/Bergman alpha_k panel | `alpha_k_uniform_fp32/` | 11-model fp32 alpha_k profiles, cross-checked vs OLD scanner |
| MAIN/Bergman geometry | `bergman_phase1/`, `bergman_results_t1t3_v2/`, `bergman_results_zephyr/` | GPT-2 + StableLM Zephyr T1/T2/T3/C1/C2 |
| MAIN/Bergman MNIST | `mnist_bergman/` | 4 architectures (CNN/MLP/ViT/Large-CNN), Case P |
| MAIN/Fisher | `fisher_safety_scan/`, `fisher_safety_scan_v2/`, `re_safety_scan/` | 5 pairs base vs instruct, vuln metric |
| MAIN/Width-law | `alpha_k_shard/` | 24 models 0.5B-141B, L0 alpha_k ~ N^0.74 pooled |
| MAIN/Hazard | `phase_b/` | kitchen_light refusal-flip rate per model |
| 5.2-5.7 | `evidence/` | 166 crossings, EOS flips, FGSM |
| 5.8 factuality | `factuality/` | 48 germs, fold-dominated atlas |
| 5.9 hazard atlas | `navier_stokes_results/hazard_atlas/`, `turbulent_results/` | 76+150 hazard points, GPT-2 + StableLM |
| 5.10 control graph | `control_graph/` | c-bar, 100% opposing |
| 5.11 P2 superlinearity | `p2_results/` | 30 points, GPT-2, mean 1.46x |
| 5.12 arithmetic / helix | `all_tests/`, `arithmetic_*/`, `helix_*/`, `fourier_results/`, `tight_validation/`, `statistical_validation/` | Tests T1-T5 + Fourier |
| 5.13 logic | `logic_results/` | Fold/cusp truth table + braid |
| 5.14 density | `density_results/` | 17 layers, alpha_k vs sigma_min, rho = 0.76 |
| Appendix discrete monodromy | `sweep_winding/` | 133 hits |
| Appendix steering / GCG | `*_steering.json`, `targeted_gcg_*.json`, `circuit_*.json` | one-off probes |

## Geometric + safety scanning suite (the new work, 2026-05)

| Directory | What it contains | Script | Paper section |
|-----------|------------------|--------|---------------|
| `alpha_k_uniform_fp32/` | Per-layer alpha_k via in-process scanner (fp32), 11 LMs | `src/cat_scanner/uniform/alpha_k_panel.py`, `scripts/alpha_k_uniform.py` | MAIN/Bergman |
| `alpha_k_shard/` | Per-model L0/L_last alpha_k via HTTP-Range partial fetch, 24 models 0.5B-141B | `scripts/alpha_k_shard_scan.py`, `_panel.py`, `_aggregate.py` | MAIN/Width-law |
| `alpha_k_shard/scaling.png` | 4-panel width-law figure | `scripts/plot_shard_scaling.py` | MAIN/Width-law figure |
| `fisher_safety_scan/`, `_v2/`, `_pgd/` | Tr(F) base vs instruct + PGD variant | `scripts/fisher_safety_scan.py`, `scripts/re_safety_scan.py` | MAIN/Fisher |
| `phase_b/` | kitchen_light refusal-flip rate per OpenRouter model, n=100 each (10 BACKED, 9 ERR) | `scripts/phase_b_hazard.py` + `scripts/blackbox_attacks/harness.py` | MAIN/Hazard |
| `cross_reference/` | Lambda_obs (Phase B) joined to alpha_k (Phase A) by HF repo, n=4 matched | `scripts/hazard_cross_reference.py` | MAIN/Hazard |
| `mnist_bergman/` | 4-arch MNIST Bergman + kl_inflation | `scripts/mnist_bergman.py`, `scripts/kl_inflation_diagnostic.py` | MAIN/Bergman |
| `bergman_phase1/`, `bergman_results_t1t3_v2/`, `bergman_results_zephyr/` | T1/T2/T3 on GPT-2 + StableLM Zephyr | `cli.py --mode bergman` | MAIN/Bergman |
| `widthlaw_spectral/` | Pilot spectral width-law on Qwen 0.5B | `scripts/widthlaw_spectral.py` | SUPERSEDED by `alpha_k_shard/` |

## Aggregate files to cite directly

| File | Contents |
|------|----------|
| `alpha_k_shard/aggregate.json` | Width-law master table: 24 rows with n_params, L0 alpha_k, L_last alpha_k, zero counts, per-layer detail, bytes_downloaded |
| `alpha_k_shard/shard_panel_journal.json` | Scan status journal (ok / oom_giveup / no_access / cuda_dead / error) |
| `alpha_k_uniform_fp32/aggregate.json` | All-model whitebox alpha_k aggregate |
| `alpha_k_uniform_fp32/normalized.json` | Per-layer alpha_k normalized by per-model max |
| `phase_b/summary.json` | 19-row refusal-flip summary across the OpenRouter sweep (10 BACKED + 9 ERR) |
| `cross_reference/hazard_vs_alpha_k.json` | Joined Lambda_obs vs alpha_k_L0 panel (n=4 currently; pending Llama-70B shard scans) |
| `cross_reference/hazard_vs_alpha_k.png` | 4-panel cross-reference figure |
| `hazard_validation/lemma_alpha_k_epsilon_scaling.json` | Direct lemma test: 144 (model,layer) slope fits; predicted 1.0 observed median 0.92 |
| `hazard_validation/lemma_alpha_k_epsilon_scaling.png` | 4-panel figure: E[count] vs eps, slope histogram, per-arch medians |
| `random_init_baseline/comparison.json` | random-init vs trained alpha_k at matched shape (8 models): training amplifies 2-1000x; SmolLM2 inverted |
| `random_init_baseline/cdf_comparison.png` | 4-panel control: shows slope=1 is generic, magnitude is training-imposed |
| `random_init_baseline/catastrophe_type_test.png` | codim-1 fold confirmation: sigma_(k) slopes 0.94, 2.43, 3.39, 4.90 vs predicted k=1,2,3,4 |
| `random_init_baseline/catastrophe_type_test.json` | per-rank slope fits + raw samples |
| `bergman_correspondence/comparison.json` | α_k vs Tr(J^TJ) vs log|det J| at 8 models random vs trained; ρ=-0.81 cross-model |
| `bergman_correspondence/bergman_alpha_k.png` | 4-panel Bergman correspondence plot |
| `fisher_safety_scan/summary.json` | 5-pair vulnerability summary |
| `bergman_phase1/bergman_summary.json` | GPT-2 5000-step T1/T2/T3/C1/C2 + return-time |

## Top-level reference data

| File | Contents |
|------|----------|
| `pku_prompts.csv` | PKU-SafeRLHF source prompts used by Phase B + helper harness |
| `truthfulqa_100.json` | 100 TruthfulQA prompts used in 5.8 |
| `chat/` | Chat templates and per-model formatting |

## See also

- `../Butterfly_Knife/core/10 - Data Inventory.md` -- full inventory
  with status flags (BACKED / SUPERSEDED / DEMO / UNUSED / TODO).
- `../Butterfly_Knife/frontier/blackbox_findings.md` -- kitchen_light
  campaign notes that motivated Phase B.
- `../scripts/model_lists/tiers.json` -- model lists by parameter band
  used by the shard scanner.
- `../scripts/model_lists/openrouter_map.json` -- hf_repo to OpenRouter
  ID mapping used by Phase B.
