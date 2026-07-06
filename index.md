# Índice del Wiki

> Léeme antes de crear cualquier nota nueva. Actualízame en cada ingest.

## 01_Physics_Engines
| Nota | Resumen | Status | Actualizado |
|---|---|---|---|
| [[Genesis_Simulator]] | Hub: arquitectura de Genesis (4 capas), solvers disponibles | draft | 2026-07-05 |
| [[PBD_Cloth_Solver]] | Solver PBD para tela: parámetros, compliance, iteraciones | draft | 2026-07-05 |

## 02_Robotics
| Nota | Resumen | Status | Actualizado |
|---|---|---|---|
| [[UR10e_2F85_Genesis]] | Carga y control del UR10e + gripper 2F-85 en Genesis | validated | 2026-07-05 |

## 03_AI_Training
| Nota | Resumen | Status | Actualizado |
|---|---|---|---|
| [[Gym_Wrapper_Genesis]] | Estado de wrappers tipo Gymnasium para Genesis (genesis_lr); arquitectura Handler+Env de RoboVerse | draft | 2026-07-06 |

## 04_Integration_ROS
_(vacío — pendiente de primer ingest)_

## 05_Literature_Review
| Nota | Resumen | Status | Actualizado |
|---|---|---|---|
| [[Unphased_Wrinkles_2022]] | Estimación de parámetros de elasticidad de tela (denim vs algodón) por pérdida en frecuencia | draft | 2026-07-05 |
| [[Unfolding_Cloth_Review_2024]] | Survey 2024: taxonomía de tareas de cloth manipulation, simuladores (PBD vs FEM), gaps (fricción, construction technique) | draft | 2026-07-05 |
| [[SoftGym_CoRL2020]] | Benchmark RL estándar para deformables (NVIDIA FleX/PBD); 6 tareas de tela/agua; CEM > SAC > imagen | draft | 2026-07-05 |
| [[GarmentLab_NeurIPS2024]] | Benchmark NeurIPS 2024 (Isaac Sim, PBD+FEM); 20 tareas; RL <15% vs visión ~60%; sim2real con 3 métodos | draft | 2026-07-06 |
| [[Petrik_GarmentFolding_IROS2016]] | Modelo Euler-Bernoulli para trayectorias de doblado; parámetro ηᵦ=peso/rigidez; contrasta con PBD | draft | 2026-07-06 |
| [[RoboVerse_2025]] | Plataforma unificada multi-simulador; patrón Handler+Env (MetaSim); Tabla VI completada del PDF | draft | 2026-07-06 |
| [[RealToSim_DynamicFabric_2025]] | Compara DIFFCLOUD/DiffCP/PhysNet/PINN; parámetros quasi-estáticos no generalizan a dinámico; PBD sin resolver | draft | 2026-07-06 |

## 06_Code_Snippets
| Nota | Resumen | Status | Actualizado |
|---|---|---|---|
| [[PBD_Cloth_Config_Denim]] | Config de PBD.Cloth calibrada para mezclilla | draft | 2026-07-05 |
