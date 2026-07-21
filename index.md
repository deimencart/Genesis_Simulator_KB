# Índice del Wiki

> Léeme antes de crear cualquier nota nueva. Actualízame en cada ingest.

## 01_Physics_Engines
| Nota | Resumen | Status | Actualizado |
|---|---|---|---|
| [[Genesis_Simulator]] | Hub: Genesis World, solvers (PBD/FEM/IPC/MPM/SPH), Quadrants compiler | draft | 2026-07-06 |
| [[PBD_Cloth_Solver]] | Solver PBD para tela: parámetros, compliance, iteraciones | draft | 2026-07-05 |
| [[Genesis_Config_System]] | Todos los *Options y materiales del config system (fuente: repo oficial) | validated | 2026-07-06 |
| [[Genesis_Cloth_PBD_Examples]] y [[Genesis_Cloth_IPC_Examples]] | (→ dividida; ver PBD y IPC) | draft | 2026-07-06 |
| [[Genesis_Cloth_PBD_Examples]] | pbd_cloth.py + cloth_on_rigid.py + patrón add_camera/start_recording | validated | 2026-07-06 |
| [[Genesis_Cloth_IPC_Examples]] | ipc_robot_cloth_teleop.py (robot+cloth FEM, friction_mu=0.5) — hallazgo prioritario | validated | 2026-07-06 |
| [[Genesis_Installation]] | Instalación genesis-world: pip/fuente, backends CUDA/Metal/ROCm, extras | validated | 2026-07-06 |
| [[Genesis_Simulation_Interface]] | Carga de robots MJCF/URDF, DOF control, IK, n_envs paralelos, sensores, cámaras | validated | 2026-07-06 |

## 02_Robotics
| Nota | Resumen | Status | Actualizado |
|---|---|---|---|
| [[UR10e_2F85_Genesis]] | Carga y control del UR10e + gripper 2F-85 en Genesis | validated | 2026-07-05 |
| [[UR10e_Gripper_Cloth_Issues]] | Slippage gripper-tela: causas (friction 0.15 vs 0.5), parámetros a corregir; + hipótesis sostén insuficiente del brazo (pendiente verificar) | issue | 2026-07-12 |
| [[Franka_Workspace_Reachability]] | Mapeo de alcanzabilidad por sampling+IK (N=200, 113 alcanzables=56.5%): límite máximo real ~0.85-1.0m verificado; límite mínimo NO verificado (solo exclusión de diseño); orientación fija (limitación) | validated | 2026-07-09 |
| [[Franka_Force_Range_Experiment]] | force_range topa el actuador (12.0 Nm exacto) pero el pico grande (-177 a -206 Nm) es fuerza de reacción de contacto al chocar el límite de posición; DOF sin control cede bajo gravedad (joint2 +1.79 rad); set_dofs_force_range propio confirmado que se respeta sobre el default | validated | 2026-07-21 |
| [[Franka_Control_Baseline]] | Carga MJCF, índices de joints, ganancias kp/kv/force_range (3 variantes); posición y fuerza verificados, velocidad SIN verificar (solo comandada); waypoints+smoothstep pendiente | draft | 2026-07-21 |

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
| [[ClothParamOpt_PBD_JCDE2025]] | Optimización real-to-sim con PBD/Taichi (UR10); 4 parámetros k'_s, k'_b, c', μ; 4.8× mejor que DIFFCLOUD | draft | 2026-07-06 |
| [[FoldingTwoParams_FrontNR2022]] | Solo 2 parámetros de trayectoria Bézier (height ratio + tilt); contacto/friction > constitutiva para folding | draft | 2026-07-06 |
| [[ManiSkill_HAB_2024]] | Benchmark GPU-acelerado (4,109 SPS, 1024 envs); arquitectura SequentialTask; solo rígidos (no cloth) | draft | 2026-07-06 |
| [[Gymnasium_API_2024]] | Spec oficial de la interfaz Gym: reset/step/render, terminated vs truncated, VectorEnv, custom env template | validated | 2026-07-06 |
| [[GOG_Gripper_RA_L2024]] | Gripper custom bimanual-con-un-brazo (UR5); folding MIoU=0.917; baja relevancia directa al proyecto | draft | 2026-07-06 |

## 06_Code_Snippets
| Nota | Resumen | Status | Actualizado |
|---|---|---|---|
| [[PBD_Cloth_Config_Denim]] | Config de PBD.Cloth calibrada para mezclilla | draft | 2026-07-05 |
| [[Franka_Reachability_Sampling]] | Código: sampling+IK para mapeo de alcanzabilidad, visualización esferas verde/rojo | validated | 2026-07-09 |
| [[Franka_Force_Range_Limits]] | Código real v1/v2/v3: control_dofs_force excesivo a un DOF, tracking + gráfica fuerza/posición, override de force_range propio | validated | 2026-07-21 |
