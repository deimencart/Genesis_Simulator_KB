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
