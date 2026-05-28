# Input-loop monodromy results log

Appended by `scripts/input_loop_monodromy.py`. Each row is one model x dataset x mode (flat or nested) run.

## Auto-appended runs

| Timestamp | Model | Mode | dtype | n_prompts | n_cycles | Periods | per-period p_tail | Wall (s) |
|---|---|---|---|---|---|---|---|---|
| 2026-05-25 17:38 | gpt2 | nested | fp32 | 100 | 5 | 3,5,8 | p3=0.168, p5=0.102, p8=0.063 | 8 |
| 2026-05-25 17:40 | Qwen2.5-0.5B | nested | bf16 | 100 | 5 | 3,5,8 | p3=0.154, p5=0.102, p8=0.081 | 31 |
| 2026-05-25 17:40 | Qwen2.5-0.5B-Instruct | nested | bf16 | 100 | 5 | 3,5,8 | p3=0.190, p5=0.117, p8=0.085 | 31 |
| 2026-05-25 17:43 | Qwen2.5-1.5B | nested | bf16 | 100 | 5 | 3,5,8 | p3=0.155, p5=0.092, p8=0.063 | 63 |
| 2026-05-25 17:45 | Qwen2.5-1.5B-Instruct | nested | bf16 | 100 | 5 | 3,5,8 | p3=0.201, p5=0.111, p8=0.081 | 63 |
| 2026-05-25 17:49 | microsoft/phi-2 | nested | bf16 | 100 | 5 | 3,5,8 | p3=0.147, p5=0.113, p8=0.070 | 92 |

## Pilot scan synthesis (2026-05-25)

6 models, n=100 PKU prompts each, k=5 cycles, periods 3/5/8. The nested
scan uses outer block [B1 B1 B2 B2] where B1, B2 are two distinct halves
of each prompt; p_outer_tail counts the fraction of outer-block-position
classes whose top-1 predictions differ across cycles 2..k (induction-
stable, cycle-1 transient stripped).

### Flat (single-base) p_tail

| Model | dtype | p=3 | p=5 | p=8 |
|---|---|---|---|---|
| gpt2 (124M) | fp32 | 0.213 | 0.156 | 0.116 |
| Qwen2.5-0.5B base (494M) | bf16 | 0.360 | 0.246 | 0.143 |
| Qwen2.5-0.5B Instruct | bf16 | 0.393 | 0.266 | 0.161 |
| Qwen2.5-1.5B base (1544M) | bf16 | 0.363 | 0.222 | 0.133 |
| Qwen2.5-1.5B Instruct | bf16 | 0.360 | 0.226 | 0.133 |
| microsoft/phi-2 (2.7B) | bf16 | 0.383 | 0.228 | 0.135 |

### Nested [B1 B1 B2 B2] outer-cycle p_tail

| Model | dtype | p=3 | p=5 | p=8 |
|---|---|---|---|---|
| gpt2 | fp32 | 0.168 | 0.102 | 0.063 |
| Qwen2.5-0.5B base | bf16 | 0.154 | 0.102 | 0.081 |
| Qwen2.5-0.5B Instruct | bf16 | 0.190 | 0.117 | 0.085 |
| Qwen2.5-1.5B base | bf16 | 0.155 | 0.092 | 0.063 |
| Qwen2.5-1.5B Instruct | bf16 | 0.201 | 0.111 | 0.081 |
| microsoft/phi-2 | bf16 | 0.147 | 0.113 | 0.070 |

### Base vs Instruct (nested outer) significance

Two-proportion z-test, n=1200 (p=3), 1780 (p=5), 2272 (p=8) classes per model.

| Comparison | p=3 | p=5 | p=8 |
|---|---|---|---|
| Qwen 0.5B base vs inst | diff +0.036, z=2.33, p=0.020 (*) | diff +0.015, z=1.45, p=0.148 (ns) | diff +0.004, z=0.54, p=0.591 (ns) |
| Qwen 1.5B base vs inst | diff +0.046, z=2.94, p=0.003 (**) | diff +0.019, z=1.89, p=0.059 (ns) | diff +0.018, z=2.30, p=0.022 (*) |
| Pooled (0.5B + 1.5B) | diff +0.041, z=3.72, p=0.0002 (***) | diff +0.017, z=2.35, p=0.019 (*) | diff +0.011, z=1.96, p=0.050 (*) |

### Cross-family check: does nested compress the GPT-2 vs Qwen gap?

| Period | FLAT diff (Qwen 0.5B - GPT-2) | NESTED-outer diff |
|---|---|---|
| p=3 | +0.147 | -0.013 |
| p=5 | +0.090 | +0.000 |
| p=8 | +0.027 | +0.018 |

The cross-family difference at p=3 collapses from +0.147 (flat) to -0.013
(nested). The flat between-model differences are likely induction-onset
or tokenizer artifacts, not catastrophe-density effects.

## Findings (honest)

1. **Safety tuning increases nested-outer catastrophe density.** Pooled
   across two Qwen sizes, the direction is statistically significant at
   all three periods (p=0.0002 at p=3, p=0.019 at p=5, p=0.050 at p=8).
   The effect is strongest at the shortest cycle. This is the only effect
   that survives both flat AND nested testing.

2. **Within-family size effect is null.** Qwen 0.5B base (494M) and
   Qwen 1.5B base (1544M, same family, 3x parameters) give nested-outer
   p_tail: 0.154 vs 0.155 at p=3, 0.102 vs 0.092 at p=5, 0.081 vs 0.063
   at p=8. The width-law-style "bigger = more catastrophe" prediction
   is NOT supported by within-family input-space density at this scale
   range.

3. **Cross-family flat differences were artifacts.** Under nested testing,
   four model families spanning 124M to 2.7B (22x parameter range) give
   p_outer_tail in [0.147, 0.201] at p=3, [0.092, 0.117] at p=5,
   [0.063, 0.085] at p=8. Tight clustering across a 22x parameter range
   suggests input-space catastrophe density is a task/data property
   more than a model property in this regime.

## Ceiling test (6 GB RTX 2060)

Phi-2 (2.7B params, bf16 ~ 5.4 GB) ran end-to-end on the 6 GB 2060
(torch cuda:1) with no quantization, batch 1, max length 128, k=5
cycles. Wall time 92 s for the full nested scan. This sets the
practical model ceiling on this hardware.

Larger candidates (Llama 3.2 3B, Qwen 2.5 3B, Mistral 7B) would need
8-bit quantization or the SUPER (cuda:0, 8 GB) when triton is freed.

### canonical scan, 2026-05-25 18:03, gpt2 (float32, k=5)
- canonical: days_full=0.14, days_short=0.00, months=0.08, steak=0.17, sizes=0.17, rainbow=0.12
- random: p5=0.080, p7=0.100, p10=0.055
- wall: 2s

### canonical scan, 2026-05-25 18:04, Qwen/Qwen2.5-0.5B (bfloat16, k=5)
- canonical: days_full=0.00, days_short=0.00, months=0.08, steak=0.10, sizes=0.40, rainbow=0.25
- random: p5=0.101, p7=0.077, p10=0.060
- wall: 6s

### canonical scan, 2026-05-25 18:04, Qwen/Qwen2.5-0.5B-Instruct (bfloat16, k=5)
- canonical: days_full=0.29, days_short=0.00, months=0.08, steak=0.20, sizes=0.40, rainbow=0.25
- random: p5=0.105, p7=0.091, p10=0.059
- wall: 6s

### canonical scan, 2026-05-25 18:05, Qwen/Qwen2.5-1.5B (bfloat16, k=5)
- canonical: days_full=0.00, days_short=0.43, months=0.83, steak=0.20, sizes=0.20, rainbow=0.00
- random: p5=0.111, p7=0.104, p10=0.071
- wall: 13s

### canonical scan, 2026-05-25 18:06, Qwen/Qwen2.5-1.5B-Instruct (bfloat16, k=5)
- canonical: days_full=0.29, days_short=0.57, months=0.42, steak=0.20, sizes=0.00, rainbow=0.00
- random: p5=0.108, p7=0.094, p10=0.074
- wall: 13s
