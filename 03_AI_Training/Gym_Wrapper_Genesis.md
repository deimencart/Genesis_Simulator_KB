---
category: [training, simulation]
status: issue
created: 2026-07-05
updated: 2026-07-05
tags: [rl, gymnasium, genesis]
---

# Wrapper tipo Gymnasium para Genesis

No existe (a 2026-07-05) un wrapper oficial tipo SoftGym/ManiSkill para manipulación de
deformables en Genesis. Esto es un hueco de investigación identificado.

## Lo más cercano
- `genesis_lr` (LeggedGym-Ex): https://github.com/lupinjia/genesis_lr — porta el patrón
  de `legged_gym`/IsaacGymEnvs a Genesis, pero está orientado a **locomoción**, no
  manipulación. Habría que adaptar `reset()`/`step()`/`action_space` para tela.

## Referencia de API estándar
Gymnasium: `reset()`, `step()`, `render()`, `observation_space`, `action_space`.
Paper: arXiv 2407.17032.

## Benchmarks comparables (no en Genesis, pero sirven de referencia de diseño)
- SoftGym (CoRL 2020) — Fold Cloth, Straighten Rope, variante SoftGym-Robot (Sawyer/Franka).
- GarmentLab (NeurIPS 2024) — nota: RL le va mal en garments complejos vs. métodos
  vision-based; dato relevante para justificar el enfoque del proyecto.
- ManiSkill-HAB — diseño de benchmark GPU-accelerated.

## Decisión pendiente
Construir un wrapper Gymnasium propio sobre Genesis para manipulación de tela —
esto puede ser una contribución citable del proyecto.

## Relacionado
- [[Genesis_Simulator]]
- [[UR10e_2F85_Genesis]]
