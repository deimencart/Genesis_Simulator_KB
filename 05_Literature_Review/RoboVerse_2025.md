---
category: [literature, training, simulation]
status: draft
created: 2026-07-06
updated: 2026-07-06
tags: [paper, benchmark, metasim, gym-wrapper, simulator-comparison, architecture]
---

# RoboVerse: Towards a Unified Platform, Dataset and Benchmark for Scalable and Generalizable Robot Learning (2025)

arXiv: https://arxiv.org/abs/2504.18904
Autores: Haoran Geng, Feishi Wang, Songlin Wei et al. (UC Berkeley, PKU, USC, UMich, UIUC, Stanford, CMU, UCLA, BIGAI; incluye Pieter Abbeel)
Venue: preprint arXiv, abril 2025

> **Nota de ingest:** el PDF supera el límite de lectura. La sección del Handler+Env
> pattern y la lista de simuladores se confirmaron en HTML/ar5iv y en el repo GitHub.
> **Tabla VI** (comparativa de simuladores) y los **experimentos de conservación de
> leyes físicas** estaban en el PDF pero no en la versión HTML accesible — el usuario
> debe completar esos datos directamente del PDF y actualizar esta nota.

## Por qué importa para el proyecto

RoboVerse/MetaSim define el patrón arquitectónico de referencia para wrappers
multi-simulador con Gym API. El patrón **Handler + Env** es directamente aplicable
al diseño de [[Gym_Wrapper_Genesis]]. Además, reporta que ningún simulador conserva
perfectamente las leyes físicas básicas — hallazgo crítico para calibrar
[[PBD_Cloth_Config_Denim]] y entender los límites de [[Genesis_Simulator]].

## Simuladores soportados por MetaSim

| Simulador | Estado de soporte |
|-----------|-----------------|
| **Genesis** | Activo |
| Isaac Sim | Activo |
| Isaac Gym | Activo |
| MuJoCo | Activo |
| SAPIEN 2/3 | Activo |
| PyBullet | Activo |
| Newton | Activo |
| MJX | Experimental |
| Blender | Experimental |

Genesis se describe como el único con "high-fidelity soft-body and liquid simulation"
entre los soportados activamente — relevante para tela deformable.

**Limitación crítica documentada:** "Integration of a unified format for non-rigid
objects is not yet fully supported, which we leave for future work." — no hay pipeline
unificado para deformables/cloth entre simuladores a fecha del paper.

## Tabla VI — Comparativa cuantitativa de simuladores

> **Pendiente de completar.** No accesible en la versión HTML. El usuario debe
> transcribir aquí las columnas y filas de la Tabla VI del PDF (Genesis vs. Isaac
> Sim/Gym/MuJoCo/PyBullet/SAPIEN — velocidad, física, soporte deformables, etc.).

| Simulator | Physics Engine | Rendering | Sensor Support | Dynamics | GPU | Open |
| --- | --- | --- | --- | --- | --- | --- |
| SAPIEN [125] | PhysX-5, Warp | Rasterization<br>RayTracing | RGBD; Force; Contact | Rigid; Soft; Fluid | ✓ | ✓ |
| PyBullet [16] | Bullet | Rasterization | RGBD; Force<br>IMU; Tactile | Rigid; Soft; Cloth | | ✓ |
| MuJoCo [114] | MuJoCo | Rasterization | RGBD; Force<br>IMU; Tactile | Rigid; Soft; Cloth | ✓ | ✓ |
| CoppeliaSim [101] | MuJoCo; Bullet<br>ODE; Newton; Vortex | Rasterization | RGBD; Force; Contact | Rigid; Soft; Cloth | | ✓ |
| Isaac Sim [88] | PhysX-5 | RayTracing | RGBD; Lidar; Force<br>Effort; IMU; Contact<br>Proximity | Rigid; Soft<br>Cloth; Fluid | ✓ | |
| Isaac Gym [75] | PhysX-5, Flex | Rasterization | RGBD; Force; Contact | Rigid; Soft; Cloth | ✓ | |
| Genesis [2] | Genesis | Rasterization<br>RayTracing | RGBD; Force; Tactile | Rigid; Soft | ✓ | ✓ |

Referencia: image_b3bf03.png

> **Resolución de C1 (lint 2026-07-06):** La clasificación "Rigid; Soft" para Genesis
> en esta tabla es una **simplificación de la taxonomía de RoboVerse**, no una limitación
> real del motor. El código fuente de Genesis World (`genesis/engine/materials/PBD/cloth.py`
> y `genesis/engine/materials/FEM/cloth.py`) confirma soporte explícito para cloth, con
> tres ejemplos oficiales: `examples/tutorials/pbd_cloth.py`,
> `examples/coupling/cloth_on_rigid.py` y `examples/IPC_Solver/ipc_robot_cloth_teleop.py`.
> Ver [[Genesis_Cloth_PBD_Examples]], [[Genesis_Cloth_IPC_Examples]] y [[Genesis_Config_System]] para evidencia directa.
## Experimentos de conservación de leyes físicas

> **Pendiente de completar.** El hallazgo de que ningún simulador conserva
> perfectamente momento/energía debe completarse del PDF: qué simuladores se probaron,
> qué métricas, y los valores numéricos de error por simulador.

**Implicación ya documentable:** estos experimentos justifican no confiar ciegamente
en la física de Genesis para denim — es necesaria la calibración empírica descrita
en [[PBD_Cloth_Config_Denim]] y los valores de referencia de [[Unphased_Wrinkles_2022]].

## Arquitectura MetaSim: patrón Handler + Env

El diseño de MetaSim separa la lógica de simulación en dos capas:

### Capa 1 — Handler (por simulador)

Cada simulador implementa un `Handler` con esta interfaz:

```python
class Handler:  # interfaz abstracta, una implementación por simulador
    def launch(self): ...
    def get_states(self) -> States: ...
    def set_states(self, states=None, action=None): ...
    def step(self): ...
    def render(self): ...
    def close(self): ...
    def get_extra(self): ...
```

### Capa 2 — Env (Gym API, agnóstico al simulador)

```python
class Env:
    def __init__(self, handler: Handler):
        self.handler = handler
        handler.launch()

    def reset(self):
        self.handler.set_states()
        states = self.handler.get_states()
        return get_observation(states), self.handler.get_extra()

    def step(self, action):
        self.handler.set_states(action=action)
        self.handler.step()
        states = self.handler.get_states()
        return (
            get_observation(states),
            get_reward(states),
            get_success(states),
            get_termination(states),
            get_time_out(states),
            self.handler.get_extra(),
        )

    def render(self): return self.handler.render()
    def close(self):  self.handler.close()
```

### Capa 0 — Configuración universal (MetaConfig)

`ScenarioCfg`: dataclass anidada que abstrae agentes, objetos, tareas, sensores y
parámetros físicos de forma agnóstica al simulador. Permite cambiar de backend
con una sola línea (`simulator="genesis"` → `simulator="mujoco"`).

**Principio de diseño:** "Each simulator has its own handler instance implementing
this interface...spanning the whole lifecycle of simulating a task."

## Aplicación directa a Gym_Wrapper_Genesis

Ver sección añadida en [[Gym_Wrapper_Genesis]] — el patrón Handler+Env es el blueprint
para nuestro wrapper propio sobre Genesis para manipulación de denim.

## Conclusiones para el proyecto

1. **Arquitectura:** implementar [[Gym_Wrapper_Genesis]] con separación Handler/Env
   siguiendo MetaSim — facilita futuro cambio de backend si Genesis tiene limitaciones.
2. **Deformables:** MetaSim aún no unifica cloth entre simuladores; Genesis debe usarse
   en modo single-simulator para tela (no hay ventaja de abstracción aquí todavía).
3. **Física no perfecta:** la falta de conservación en simuladores refuerza la necesidad
   de calibración empírica sobre valores teóricos — ver [[PBD_Cloth_Config_Denim]].

## Relacionado
- [[Gym_Wrapper_Genesis]] — patrón Handler+Env como blueprint de diseño
- [[Genesis_Simulator]] — uno de los backends soportados; único con soft-body activo
- [[PBD_Cloth_Config_Denim]] — calibración necesaria dado que física no es perfecta
- [[SoftGym_CoRL2020]] — benchmark previo de referencia para diseño de envs de tela
- [[GarmentLab_NeurIPS2024]] — dataset de GarmentLab incluido en RoboVerse
