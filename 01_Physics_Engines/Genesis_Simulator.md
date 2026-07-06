---
category: [simulation]
status: draft
created: 2026-07-05
updated: 2026-07-06
tags: [genesis, genesis-world, physics-engine, hub]
---

# Genesis World

**Genesis World** (`genesis-world`) — motor de física multi-solver para robótica (Genesis-Embodied-AI).
Arquitectura de 4 capas: Simulation Interface → Physics → Render → Compiler.

- Docs: https://genesis-world.readthedocs.io/en/latest/
- Repo: https://github.com/Genesis-Embodied-AI/genesis-world

**Compiler — Quadrants:** compilador JIT que soporta CUDA, ROCm (HIP), Apple Metal, Vulkan, x86 y ARM64.
Fork de Taichi desde junio 2025 (re-implementación del backend de compilación).

## Solvers disponibles
Ver nota atómica de cada uno:
- [[PBD_Cloth_Solver]] — recomendado para tela/cloth (nuestro caso de uso).
- **IPC** (`IPCCouplerOptions` vía libuipc) — contacto intersection-free para FEM.Cloth; requiere CPU + `pip install pyuipc`. Ver [[Genesis_Cloth_IPC_Examples]] (ejemplo `ipc_robot_cloth_teleop`).
- FEM (`FEMOptions`) — deformables volumétricos; `FEM.Cloth` usa thin-shell con IPC.
- MPM (`MPMOptions`) — granulares, plasticina, nieve.
- SPH (`SPHOptions`) — fluido smoothed-particle (WCSPH / DFSPH).
- SF (`SFOptions`) — smoke/fluid euleriano (grid-based).
- Rigid body (`RigidOptions`) — hasta 43M FPS en Franka arm (RTX 4090).

Ver [[Genesis_Config_System]] para la lista completa de Options y materiales.

## Interfaces de simulación
[[Genesis_Simulation_Interface]] — carga de robots (MJCF/URDF), control de DOFs, IK,
simulación paralela (n_envs), cámaras, sensores, grabación de video.

## Instalación y backends
[[Genesis_Installation]] — Python 3.10–3.13, PyTorch, backends CUDA/ROCm/Metal/CPU.

## Ejemplos de cloth
[[Genesis_Cloth_PBD_Examples]] — PBD cloth (pbd_cloth.py, cloth_on_rigid.py) + patrón grabación.
[[Genesis_Cloth_IPC_Examples]] — robot+cloth con IPC (ipc_robot_cloth_teleop.py, hallazgo prioritario).

## Robots cargados en este proyecto
- [[UR10e_2F85_Genesis]]

## RL / entornos
- [[Gym_Wrapper_Genesis]]

## Este proyecto
Ver `05_Literature_Review/` para papers que respaldan la elección de parámetros de tela
y `06_Code_Snippets/` para configuraciones ya probadas.
