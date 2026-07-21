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

## [2026-07-09] experimento | Mapeo de alcanzabilidad del Franka (IK sampling)
- Notas creadas: [[Franka_Workspace_Reachability]], [[Franka_Reachability_Sampling]]
- Notas actualizadas: index.md
- Contradicciones encontradas: ninguna
- Nota: workspace tiene forma de anillo/cascarón (radio mínimo y máximo ~0.85m, no
  esfera sólida); mapeo usó orientación fija (limitación metodológica, pendiente
  repetir con gripper hacia abajo); mismo método pendiente de repetir para UR10e en
  Fase 3 para descartar workspace como causa del slippage documentado en
  [[UR10e_Gripper_Cloth_Issues]]. Desglose exhaustivo y proceso de depuración
  completo fuera del vault, en `~/genesis_ws/tutor_notes/`.

## [2026-07-09] corrección | Franka_Workspace_Reachability re-verificada contra archivos reales
- Notas creadas: ninguna
- Notas actualizadas: [[Franka_Workspace_Reachability]], [[Franka_Reachability_Sampling]], index.md
- Contradicciones encontradas: sí — la entrada de log anterior (mismo día, "Mapeo de
  alcanzabilidad del Franka") se redactó sin leer los scripts/`.npy` reales de
  `~/genesis_ws/phase_0_franka/`; las cifras (radio ~0.85m como límite mínimo Y
  máximo) eran estimadas, no verificadas. Releídos los 8 scripts y los 4 `.npy` de
  esa carpeta y recalculadas las cifras reales con numpy sobre
  `reachable_points_2.npy`/`reachable_flags_2.npy` (versión final, producida por
  `4_1_reachable_franka.py`, identificada por fecha de modificación y contenido
  frente a la versión previa con bug de sesgo direccional en `4_ws_franka.py`,
  `x_range=(0.2,0.8)` solo positivo).
- Nota: N=200, 113 alcanzables (56.5%). Límite MÁXIMO de alcance confirmado por
  datos (caída de 100%→0% de alcanzabilidad entre radio 3D 0.8m y 1.0m). Límite
  MÍNIMO **no confirmado** — la exclusión cilíndrica de 0.15m fue una decisión de
  diseño previa a la generación de candidatos, no un hallazgo de fallos de IK;
  corregido en la nota (antes afirmaba incorrectamente auto-colisión/límite
  articular como causa de un radio mínimo, sin evidencia real que lo respalde).
  Script `3_inverse_kinematics.py` revisado y descartado del alcance de este
  experimento (test de IK a un solo punto, no parte del mapeo por sampling).

## [2026-07-12] experimento | Force range del Franka: fuerza de reacción vs. límite de actuador
- Notas creadas: [[Franka_Force_Range_Experiment]], [[Franka_Force_Range_Limits]]
- Notas actualizadas: [[UR10e_Gripper_Cloth_Issues]] (sección nueva "Hipótesis
  adicional: sostén insuficiente del brazo durante agarre", marcada explícitamente
  como pendiente de verificar, no causa confirmada), index.md
- Contradicciones encontradas: ninguna
- Nota: re-ejecutados ambos scripts (`5_force_range_limits.py` v1,
  `5_1_graphs_force_limits.py` v2) contra el motor instalado para verificar cifras
  antes de documentar. Hallazgo 1: force_range topa la componente de actuador
  (12.0 Nm exacto en v1, step 0) pero el pico grande observado (-177.49 Nm en v1
  step 37, -206.29 Nm en v2 step 51) es fuerza de reacción de contacto al chocar
  el límite de posición (2.8973 rad), no una violación del límite de fuerza —
  mismo fenómeno en ambas corridas pese a pedir 200 Nm en v1 y solo 12 Nm en v2.
  Detectada inconsistencia real en v2: título/comentario dicen "joint5"/"200 Nm"
  pero el código actual controla el DOF de `joint6` con 12 Nm — documentada
  explícitamente en ambas notas nuevas, no corregida silenciosamente. Hallazgo 2:
  con solo un DOF controlado, `joint2` (sin ningún comando) se desplaza +1.79 rad
  en 100 steps por gravedad — confirma que un joint no sostiene al resto de la
  cadena cinemática sin control activo propio; contrastado contra
  `examples/sensors/imu_franka.py` oficial, que sí configura kp/kv/force_range
  para los 9 DOFs completos. No se pudo verificar el límite de 12 Nm contra
  franka_ros/franka_description externo (no disponible en este entorno) —
  confirmado en su lugar contra el MJCF de Genesis y la API en vivo.

## [2026-07-21] roadmap | Actualización de Estado actual y checklist Fase 0
- Notas creadas: ninguna
- Notas actualizadas: ROADMAP.md
- Contradicciones encontradas: ninguna
- Nota: marcados `[x]` los dos experimentos ya completados (alcanzabilidad,
  force range) como extensiones de Fase 0; añadido foco inmediato (6 ítems
  núcleo de Fase 0 aún sin marcar) y recordatorio de que la hipótesis de
  sostén insuficiente del brazo en [[UR10e_Gripper_Cloth_Issues]] sigue sin
  verificar y bloquea el cierre de Fase 3.

## [2026-07-21] documentación | Baseline de control del Franka (Fase 0, ítems 1-4 y 6)
- Notas creadas: [[Franka_Control_Baseline]]
- Notas actualizadas: [[Franka_Force_Range_Experiment]] (Hallazgo 3),
  [[Franka_Force_Range_Limits]] (código v3), ROADMAP.md, index.md
- Contradicciones encontradas: ninguna
- Nota: verificados 8 scripts reales de `~/genesis_ws/phase_0_franka/`
  (no solo los 2 ya documentados). Encontrado `5_2_set_force_2joints.py`
  (12-jul), no documentado hasta hoy — confirma que `set_dofs_force_range`
  propio (±5 Nm en joint5) se respeta por encima del default del MJCF
  (±12 Nm). Con esto, los ítems 1 (carga MJCF), 2 (índices de joints), 3
  (kp/kv/force_range) y 4 (3 modos de control) de Fase 0 quedan marcados
  `[x]` en el ROADMAP con evidencia real de código. El ítem 5 (waypoints +
  interpolación smoothstep) queda `[ ]` explícitamente: `3_inverse_kinematics.py`
  solo interpola linealmente un segmento único, y
  `2_2_robot_tuto_control_your_robot.py` tiene varios waypoints pero sin
  interpolación — ningún script combina ambas cosas.

## [2026-07-21] corrección | Control de velocidad marcado [x] sin verificar — revertido
- Notas creadas: ninguna
- Notas actualizadas: [[Franka_Control_Baseline]] (sección 4), ROADMAP.md, index.md
- Contradicciones encontradas: ninguna (autocorrección tras pregunta del usuario)
- Nota: el ítem 4 de Fase 0 se había marcado `[x]` citando
  `2_2_robot_tuto_control_your_robot.py` como verificación de los 3 modos de
  control. Al releer ese script a pedido del usuario, `control_dofs_velocity`
  aparece una sola vez en todo `phase_0_franka/`, dentro del tutorial oficial
  copiado (no un experimento propio), 1 DOF, y **sin ninguna llamada a
  `get_dofs_velocity()`** — el script solo lee fuerza en cada step. Posición y
  fuerza sí tienen verificación real con lectura de estado; velocidad no.
  Revertido a `[ ]` con la distinción explícita en las 3 notas.
