---
category: [simulation]
status: draft
created: 2026-07-05
updated: 2026-07-05
tags: [genesis, physics-engine, hub]
---

# Genesis Simulator

Motor de física multi-solver para robótica (Genesis-Embodied-AI). Arquitectura de 4 capas:
Simulation Interface → Physics → Render → Compiler (Quadrants, compila a CUDA/ROCm/Metal).

- Docs: https://genesis-world.readthedocs.io/en/latest/
- Repo: https://github.com/Genesis-Embodied-AI/genesis-world
- Bibtex oficial en la doc de arquitectura.

## Solvers disponibles
Ver nota atómica de cada uno:
- [[PBD_Cloth_Solver]] — recomendado para tela/cloth (nuestro caso de uso).
- FEM (`FEMOptions`) — deformables volumétricos, más preciso físicamente, más lento.
- MPM — granulares, plasticina, nieve.
- Rigid body — hasta 43M FPS en Franka arm (RTX 4090).

## Robots cargados en este proyecto
- [[UR10e_2F85_Genesis]]

## RL / entornos
- [[Gym_Wrapper_Genesis]]

## Este proyecto
Ver `05_Literature_Review/` para papers que respaldan la elección de parámetros de tela
y `06_Code_Snippets/` para configuraciones ya probadas.
