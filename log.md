# Log

## [2026-07-05] ingest | Investigación inicial: Genesis + tela mezclilla + UR10e + RL
- Notas creadas: [[Genesis_Simulator]], [[PBD_Cloth_Solver]], [[UR10e_2F85_Genesis]],
  [[Gym_Wrapper_Genesis]], [[Unphased_Wrinkles_2022]], [[PBD_Cloth_Config_Denim]]
- Notas actualizadas: ninguna (primer ingest)
- Contradicciones encontradas: ninguna
- Pendiente: ingestar el resto de los ~10 papers listados en la conversación de origen
  (ver `raw/genesis_research_2026-07-05.md`), especialmente GarmentLab y SoftGym.

## [2026-07-05] ingest | Unfolding the Literature: A Review of Robotic Cloth Manipulation
- Notas creadas: [[Unfolding_Cloth_Review_2024]]
- Notas actualizadas: index.md
- Contradicciones encontradas: ninguna
- Nota: confirma que Genesis no aparece en este survey por ser posterior a su publicación.

## [2026-07-05] ingest | SoftGym: Benchmarking Deep RL for Deformable Object Manipulation (CoRL 2020)
- Notas creadas: [[SoftGym_CoRL2020]]
- Notas actualizadas: index.md
- Contradicciones encontradas: ninguna
- Nota: SoftGym usa FleX/PBD (misma familia que Genesis); parámetros de tela no son portables directamente — requieren recalibración con [[Unphased_Wrinkles_2022]].

## [2026-07-06] ingest | GarmentLab: A Unified Simulation and Benchmark for Garment Manipulation (NeurIPS 2024)
- Notas creadas: [[GarmentLab_NeurIPS2024]]
- Notas actualizadas: index.md
- Contradicciones encontradas: ninguna, pero hallazgo crítico — RL puro obtiene <15% de éxito en tareas de prendas vs. ~60% de métodos vision-based; cuestiona enfoque RL puro en [[Gym_Wrapper_Genesis]]

## [2026-07-06] ingest | Physics-Based Model of a Rectangular Garment for Robotic Folding (Petrik, IROS 2016)
- Notas creadas: [[Petrik_GarmentFolding_IROS2016]]
- Notas actualizadas: index.md
- Contradicciones encontradas: ninguna; modelo Euler-Bernoulli (analítico, estático) contrasta con PBD de [[PBD_Cloth_Solver]] (partículas, dinámico) — son enfoques complementarios, no contradictorios
- Nota: PDF original ilegible (binario); contenido reconstruido desde papers de seguimiento (arXiv 1902.11021, 1904.01298) y búsqueda web

## [2026-07-06] ingest | RoboVerse: Towards a Unified Platform, Dataset and Benchmark for Scalable and Generalizable Robot Learning (arXiv 2504.18904)
- Notas creadas: [[RoboVerse_2025]]
- Notas actualizadas: [[Gym_Wrapper_Genesis]] (sección Handler+Env añadida), index.md, raw/genesis_research_2026-07-05.md (paper añadido a la lista)
- Contradicciones encontradas: ninguna
- Nota: Tabla VI (comparativa de simuladores) y experimentos de conservación física no accesibles en HTML — pendiente de completar desde el PDF directamente en [[RoboVerse_2025]]

## [2026-07-06] ingest | Can Real-to-Sim Approaches Capture Dynamic Fabric Behavior for Robotic Fabric Manipulation? (arXiv 2503.16310)
- Notas creadas: [[RealToSim_DynamicFabric_2025]]
- Notas actualizadas: index.md
- Contradicciones encontradas: ninguna; complementa [[Unphased_Wrinkles_2022]] — ese paper calibra quasi-estático, este advierte que esos parámetros no generalizan a dinámico
- Nota: brecha crítica identificada — ningún método del paper usa PBD como backend; traducción a parámetros Genesis queda sin resolver

## [2026-07-06] ingest | Real-to-Sim High-Resolution Cloth Modeling: Physical Parameter Optimization Using Particle-Based Simulation (JCDE 2025)
- Notas creadas: [[ClothParamOpt_PBD_JCDE2025]]
- Notas actualizadas: index.md
- Contradicciones encontradas: ninguna; resuelve la brecha PBD de [[RealToSim_DynamicFabric_2025]] — primer paper que optimiza k'_s/k'_b/c'/μ directamente sobre simulador particle-based; hereda la limitación de no validar en dinámico

## [2026-07-06] ingest | Effective Cloth Folding Trajectories in Simulation with Only Two Parameters (Frontiers in Neurorobotics, 2022)
- Notas creadas: [[FoldingTwoParams_FrontNR2022]]
- Notas actualizadas: index.md
- Contradicciones encontradas: ninguna; converge con [[ClothParamOpt_PBD_JCDE2025]] en que friction es el parámetro más crítico — evidencia independiente desde FEM/C-IPC y PBD/Taichi

## [2026-07-06] ingest | ManiSkill-HAB: A Benchmark for Low-Level Manipulation in Home Rearrangement Tasks (arXiv 2412.13211)
- Notas creadas: [[ManiSkill_HAB_2024]]
- Notas actualizadas: index.md
- Contradicciones encontradas: ninguna
- Nota: MS-HAB es referencia de arquitectura y throughput (4,109 SPS); no maneja deformables — para cloth seguir [[SoftGym_CoRL2020]] y [[ClothParamOpt_PBD_JCDE2025]]

## [2026-07-06] ingest | Gymnasium: A Standardized Interface for Reinforcement Learning Environments (arXiv 2407.17032)
- Notas creadas: [[Gymnasium_API_2024]]
- Notas actualizadas: index.md
- Contradicciones encontradas: ninguna
- Nota: marcada como status=validated por ser spec oficial; incluye template de GenesisClothEnv y explicación de terminated vs truncated para bootstrap correcto en RL

## [2026-07-06] ingest | G.O.G.: A Versatile Gripper-On-Gripper Design for Bimanual Cloth Manipulation with a Single Robotic Arm (IEEE RA-L, 2024)
- Notas creadas: [[GOG_Gripper_RA_L2024]]
- Notas actualizadas: index.md
- Contradicciones encontradas: ninguna
- Nota: relevancia baja para el proyecto — hardware custom (no 2F-85), UR5 (no UR10e), folding (no unfolding), sin simulación ni Genesis; documentado por completitud del literature review
