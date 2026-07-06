---
category: [literature, training, simulation]
status: draft
created: 2026-07-06
updated: 2026-07-06
tags: [paper, benchmark, cloth-manipulation, sim2real, rl, vision-based, isaac-sim, physx, pbd, fem]
---

# GarmentLab: A Unified Simulation and Benchmark for Garment Manipulation (NeurIPS 2024)

arXiv: https://arxiv.org/abs/2411.01200
Página del proyecto: https://garmentlab.github.io/
Autores: Haoran Lu, Ruihai Wu, Yitong Li et al. (PKU, THU)
Venue: NeurIPS 2024

## Por qué importa para el proyecto

Benchmark de referencia (NeurIPS 2024) más completo que [[SoftGym_CoRL2020]] en tareas
de manipulación de prendas. Incluye Fold y Unfold — exactamente nuestro caso con denim.
**Hallazgo crítico:** los métodos RL (PPO) obtienen <15% de éxito en tareas de prendas;
los métodos basados en visión alcanzan ~60-67%. Esto cuestiona directamente el enfoque
puro de RL en [[Gym_Wrapper_Genesis]] y abre la discusión de híbridos.

## Simulador y física

- **Motor:** NVIDIA Omniverse **Isaac Sim** con backend **PhysX5** (no Genesis)
- **PBD:** prendas grandes (tops, pantalones) — misma familia que [[PBD_Cloth_Solver]]
- **FEM:** objetos elásticos pequeños (guantes, calcetines, juguetes)
- **PhysX articulation:** control del robot (fuerza, P-D, dinámica inversa)
- **Flow models:** simulación de viento/secadora

Parámetros físicos configurables por material: tamaño de partícula, stiffness,
módulo elástico (FEM), fricción, tensión superficial. No se publican valores
específicos para algodón/denim — ver [[Unphased_Wrinkles_2022]] para eso.

## Tareas (20 en total, 5 categorías + 4 long-horizon)

Las más relevantes para el proyecto:

| Tarea | Categoría | Métrica de éxito |
|-------|-----------|-----------------|
| **Fold** | Garment-Garment | IoU partícula-a-partícula vs. demo humana |
| **Unfold** | Garment-Garment | Coverage area (vértices dentro de rango del estado plano) |
| Hang | Garment-Garment | Estabilidad ≥5s + similitud de pose |
| Place | Garment-Garment | Colocado sin caer |
| Wash / Dry | Garment-Fluid | Estado final de limpieza/secado |
| Dress Person in T-shirt | Garment-Avatar | Colocación sobre maniquí articulado |

Tareas long-horizon: Organizing Clothes, Washing Clothes, Make Up Tables, Dress Up.

## Robots disponibles en el benchmark

| Robot | Gripper | Uso |
|-------|---------|-----|
| Franka Panda (7-DOF) | Parallel gripper | Tareas principales |
| **UR5** | Suction cup | Garment retrieval |
| **Shadow Dexterous Hand** | 16-DOF | Tareas de precisión (montado sobre UR10) |
| RidgebackFranka | Parallel gripper | Tareas móviles |

El Shadow Hand se monta sobre un UR10 — plataforma compartida con nuestro
[[UR10e_2F85_Genesis]] (brazo compatible, distinto end-effector).

## Resultados de algoritmos (simulación)

| Método | Tipo | Fold (top/pantalón) | Unfold | Hang |
|--------|------|---------------------|--------|------|
| **UGM** (PointNet++) | Visión-3D | ~61% / ~62% | ~58% / ~61% | ~62% / ~57% |
| **DIFT** (Stable Diffusion) | Visión-2D | ~33% / ~37% | ~19% / ~23% | ~31% / ~28% |
| **Affordance** (PointNet++) | Visión-3D | ~53% / ~52% | ~32% / ~37% | ~64% / ~60% |
| **RL-State** (PPO + estado) | RL | ~15% / ~13% | ~7% / ~9% | ~13% / ~20% |
| **RL-Vision** (PPO + pointcloud) | RL | ~7% / ~8% | ~5% / ~6% | ~8% / ~5% |

**Conclusión clave:** RL puro falla en tareas de prendas. Los autores explican:
"RL genera trayectorias anómalas que enredan la ropa con el brazo robótico."
UGM (correspondencia visual 3D) es el método más robusto en prendas grandes.

## Resultados sim-to-real (15 ensayos reales)

| Método | Fold (T-shirt) | Hang (sombrero) |
|--------|----------------|-----------------|
| UGM (completo) | 10/15 (67%) | 10/15 (67%) |
| DIFT | 8/15 (53%) | 14/15 (93%) |
| Affordance | 6/15 (40%) | 9/15 (60%) |
| UGM sin alineación point cloud | 2/15 | 5/15 |

Los tres componentes del pipeline sim2real son necesarios; sin alineación de point
cloud el rendimiento colapsa.

## Pipeline sim-to-real (los 3 métodos de alineación)

1. **Keypoint Embedding Alignment** — aprendizaje contrastivo (InfoNCE) entre sim y real.
2. **Noisy Observation Augmentation** — ruido en point clouds durante entrenamiento.
3. **Point Cloud Alignment** — transformación afín optimizada con Chamfer distance.

Adicionalmente: integración con **MoveIt** (planificación de movimiento) y sistema de
**teleoperación** (Leap Motion) para captura de demostraciones humanas.

## Comparación directa con SoftGym

| Característica | [[SoftGym_CoRL2020]] | GarmentLab |
|----------------|----------------------|------------|
| Simulador | FleX (PBD) | Isaac Sim (PBD + FEM) |
| Categorías prendas | ~2 | 11 + accesorios |
| Multi-robot | No | Sí (4 plataformas) |
| Sim2real | No | Sí (3 métodos) |
| RL funciona bien | Sí (CEM) | No (<15%) |
| Benchmark real | No | Sí |

## Implicaciones para el proyecto

1. **RL puro es insuficiente para prendas.** Considerar visión-basada (correspondencia
   3D / affordance) como componente principal, con RL solo para refinamiento.
2. **Sim2real requiere los 3 métodos de alineación.** Diseñar [[Gym_Wrapper_Genesis]]
   con ruido en observaciones y alineación de point cloud desde el inicio.
3. **MoveIt sobre trayectorias heurísticas.** Mejora la calidad de los datos de
   demostración y reduce el sim2real gap.
4. **FleX y PhysX usan PBD** — Genesis con [[PBD_Cloth_Solver]] es coherente con
   ambos benchmarks.

## Limitaciones reconocidas

- RL no resuelve tareas de prendas con los métodos actuales.
- Métodos de visión sensibles al estado inicial del objeto.
- Tareas long-horizon sin solución convincente.
- No específico a mezclilla (sin parámetros para denim).

## Relacionado
- [[SoftGym_CoRL2020]] — benchmark previo; comparación directa en el paper
- [[Gym_Wrapper_Genesis]] — implicación crítica: RL puro insuficiente, diseñar para visión
- [[PBD_Cloth_Solver]] — GarmentLab usa PBD para prendas grandes (misma familia)
- [[UR10e_2F85_Genesis]] — UR10 con Shadow Hand presente en el benchmark
- [[Unfolding_Cloth_Review_2024]] — survey que contextualiza GarmentLab
- [[Unphased_Wrinkles_2022]] — parámetros físicos de denim que GarmentLab no provee
