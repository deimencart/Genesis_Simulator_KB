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

## [2026-07-06] ingest | Genesis Config System (genesis/options/solvers.py + genesis/engine/materials/)
- Notas creadas: [[Genesis_Config_System]]
- Notas actualizadas: [[Genesis_Simulator]] (enlaces a las 3 notas nuevas), index.md
- Contradicciones encontradas: resuelve C1 del lint — Genesis sí soporta `PBD.Cloth` y `FEM.Cloth`; RoboVerse los clasifica bajo "Soft" en su taxonomía, no es contradicción de soporte real

## [2026-07-06] ingest | Genesis GitHub issues #161 y #484 (cloth simulation examples)
- Notas creadas: [[Genesis_Cloth_PBD_Examples]] y [[Genesis_Cloth_IPC_Examples]]
- Notas actualizadas: index.md
- Contradicciones encontradas: ninguna; #484 cerrado sin respuesta oficial — requisitos de malla deducidos del solver, no documentación explícita

## [2026-07-06] ingest | Genesis README (instalación, backends, requisitos)
- Notas creadas: [[Genesis_Installation]]
- Notas actualizadas: index.md
- Contradicciones encontradas: ninguna; clarifica que FEM.Cloth requiere pyuipc (Linux/Windows NVIDIA), PBD.Cloth no requiere extras

## [2026-07-06] ingest | Genesis Cloth Examples — 3 ejemplos oficiales (pbd_cloth.py, cloth_on_rigid.py, ipc_robot_cloth_teleop.py)
- Notas creadas: ninguna nueva (actualización completa de [[Genesis_Cloth_PBD_Examples]] y [[Genesis_Cloth_IPC_Examples]])
- Notas actualizadas: [[Genesis_Cloth_PBD_Examples]] y [[Genesis_Cloth_IPC_Examples]] (reemplazado: código real de 3 ejemplos oficiales + patrón cámara/grabación), index.md
- Contradicciones encontradas: ninguna; hallazgo crítico — ipc_robot_cloth_teleop usa FEM.Cloth con friction_mu=0.5 vs default PBD.Cloth de 0.15 → explica slippage

## [2026-07-06] ingest | Genesis Simulation Interface (examples/tutorials/: control_your_robot, parallel_simulation, visualization; examples/sensors/)
- Notas creadas: [[Genesis_Simulation_Interface]]
- Notas actualizadas: [[Genesis_Simulator]] (renombrado a Genesis World, Quadrants=fork Taichi, IPC solver añadido, enlace a Simulation Interface), index.md
- Contradicciones encontradas: ninguna

## [2026-07-06] investigación | UR10e gripper slippage (issue #199 + ipc_robot_cloth_teleop + Genesis_Config_System)
- Notas creadas: [[UR10e_Gripper_Cloth_Issues]]
- Notas actualizadas: index.md
- Contradicciones encontradas: ninguna; confirma friction como causa principal (converge con [[ClothParamOpt_PBD_JCDE2025]] y [[FoldingTwoParams_FrontNR2022]])

## [2026-07-06] lint-fix | Resolución contradicción C1 — Genesis World sí soporta cloth
- Notas creadas: ninguna
- Notas actualizadas: [[RoboVerse_2025]] (nota aclaratoria añadida en Tabla VI: "Rigid; Soft" es simplificación de taxonomía RoboVerse, no limitación de Genesis)
- Contradicciones encontradas: ninguna (C1 resuelto con evidencia de 3 ejemplos oficiales + código fuente)

## [2026-07-06] refactor | División de Genesis_Cloth_Examples en dos notas atómicas
- Notas creadas: [[Genesis_Cloth_PBD_Examples]], [[Genesis_Cloth_IPC_Examples]]
- Notas actualizadas: [[Genesis_Cloth_PBD_Examples]] y [[Genesis_Cloth_IPC_Examples]] (stub redirect), [[Genesis_Simulator]], [[Genesis_Simulation_Interface]], [[Genesis_Config_System]], [[RoboVerse_2025]], [[UR10e_Gripper_Cloth_Issues]], index.md
- Contradicciones encontradas: ninguna; todos los wikilinks de [[Genesis_Cloth_PBD_Examples]] y [[Genesis_Cloth_IPC_Examples]] actualizados a la nota correcta según contenido

## [2026-07-06] planning | Creación de ROADMAP.md (6 fases: Franka baseline → cámara → cloth calibration → UR10e debug → manipulación avanzada → Gymnasium)
- Notas creadas: ninguna (ROADMAP.md no es una nota atómica — no va en index.md)
- Notas actualizadas: CLAUDE.md (referencia a ROADMAP.md añadida como ítem 3 en la lista de lectura inicial)
- Contradicciones encontradas: ninguna

## [2026-07-06] investigación | coup_friction por link en Genesis World (Rigid material API)
- Notas creadas: ninguna (contenido añadido a [[UR10e_Gripper_Cloth_Issues]])
- Notas actualizadas: [[UR10e_Gripper_Cloth_Issues]] (sección nueva: coup_friction es entity-level; set_friction() es rigid-rigid solo; 3 opciones documentadas; Opción A recomendada)
- Contradicciones encontradas: ninguna; coup_friction default=0.1 (confirmado en código fuente), no existe API pública per-link para coup_friction
