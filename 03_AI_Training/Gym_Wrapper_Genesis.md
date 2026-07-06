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

## Arquitectura de referencia: RoboVerse / MetaSim (Handler + Env)

Fuente: [[RoboVerse_2025]] — patrón confirmado en paper y repo GitHub.

MetaSim separa el wrapper en dos capas independientes:

```
ScenarioCfg (config agnóstica)
       ↓
   Handler  ←— implementación específica por simulador
   (launch / get_states / set_states / step / render / close)
       ↓
     Env    ←— Gym API estándar (reset / step / render / close)
```

**Adaptación para Genesis + tela:**

- `GenesisHandler.launch()` → inicializa escena, robot y tela con [[PBD_Cloth_Config_Denim]]
- `GenesisHandler.get_states()` → posición de partículas PBD + joints del UR10e
- `GenesisHandler.set_states(action)` → aplica acción al gripper 2F-85
- `GenesisHandler.step()` → avanza `gs.options.PBDOptions` un timestep
- `Env.reset()` y `Env.step()` exponen Gym API sin cambios

**Ventaja clave:** si Genesis tiene limitaciones para cloth, se puede swapear el
Handler a Isaac Sim sin tocar la lógica de recompensa/observación.

## Decisión pendiente
Construir un wrapper Gymnasium propio sobre Genesis para manipulación de tela —
esto puede ser una contribución citable del proyecto. Usar el patrón Handler+Env
de MetaSim como blueprint de diseño.

## Relacionado
- [[Genesis_Simulator]]
- [[UR10e_2F85_Genesis]]
- [[RoboVerse_2025]] — fuente del patrón Handler+Env
