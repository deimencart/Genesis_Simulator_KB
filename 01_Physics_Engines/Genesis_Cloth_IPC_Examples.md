---
category: [simulation]
status: validated
created: 2026-07-06
updated: 2026-07-06
tags: [genesis, fem, ipc, cloth, robot, teleop, official, hallazgo-prioritario]
---

# Genesis World — Cloth IPC: Ejemplo Robot + Teleop (hallazgo prioritario)

Fuente: `examples/IPC_Solver/ipc_robot_cloth_teleop.py`
del repo `Genesis-Embodied-AI/genesis-world`.

**Este es el ejemplo oficial más cercano a nuestro caso de uso:**
robot articulado manipulando tela sobre superficie con obstáculos.

## Código del ejemplo

```python (genesis_env)
import genesis as gs

gs.init(backend=gs.cpu)  # IPC requiere CPU; no hay soporte GPU actualmente

scene = gs.Scene(
    sim_options=gs.options.SimOptions(dt=0.02),
    coupler_options=gs.options.IPCCouplerOptions(
        constraint_strength_translation=100.0,
        constraint_strength_rotation=100.0,
        n_linesearch_iterations=8,
        linesearch_report_energy=False,
        newton_tolerance=1e-1,
        newton_translation_tolerance=1,
        newton_semi_implicit_enable=False,
        linear_system_tolerance=1e-3,
        contact_enable=True,
        enable_rigid_rigid_contact=True,
        contact_d_hat=0.001,       # umbral de distancia de contacto (1 mm)
        contact_resistance=1e7,    # rigidez de contacto
    ),
    viewer_options=gs.options.ViewerOptions(
        camera_pos=(2.0, -1.0, 1.5),
        camera_lookat=(0.5, 0.0, 0.2),
        camera_fov=40,
    ),
    show_viewer=True,
)

# Tela 1 — FEM.Cloth, más blanda (bending_stiffness=10.0)
cloth_1 = scene.add_entity(
    material=gs.materials.FEM.Cloth(
        E=6e4,               # Young's modulus (Pa) — denim real ~0.35–0.45 GPa
        nu=0.49,             # casi incompresible
        rho=200,             # kg/m³ (densidad volumétrica para shell)
        thickness=0.001,     # 1 mm
        bending_stiffness=10.0,
        friction_mu=0.5,     # ← fricción alta para evitar slippage con gripper
    ),
    morph=gs.morphs.Mesh(
        file="meshes/grid20x20.obj",
        scale=0.5,
        pos=(0.5, 0.0, 0.1),
        euler=(90, 0, 0),
    ),
)

# Tela 2 — FEM.Cloth, más rígida (bending_stiffness=40.0)
cloth_2 = scene.add_entity(
    material=gs.materials.FEM.Cloth(
        E=6e4, nu=0.49, rho=200, thickness=0.001,
        bending_stiffness=40.0,  # 4× más rígida
        friction_mu=0.5,
    ),
    morph=gs.morphs.Mesh(
        file="meshes/grid20x20.obj",
        scale=0.3,
        pos=(0.5, 0.0, 0.14),
        euler=(90, 0, 0),
    ),
)

# Robot Franka en (0, 0, 0.005) + 16 cubos rígidos en grid 4×4 bajo las telas
# Gripper: 0.04 rad = abierto, −0.03 rad = cerrado
# Control: IK para el brazo, control_dofs_position para dedos
```

## Hallazgos críticos para el proyecto

| Aspecto | Este ejemplo (oficial) | Nuestro proyecto (plan actual) |
|---------|------------------------|-------------------------------|
| Material tela | `FEM.Cloth` | `PBD.Cloth` |
| Fricción tela | `friction_mu=0.5` | `static_friction=0.15` (default!) |
| Acoplamiento | `IPCCouplerOptions` (IPC) | `LegacyCouplerOptions.rigid_pbd` |
| Backend | CPU (obligatorio para IPC) | GPU (objetivo para RL a escala) |
| Robot | Franka Panda | UR10e + 2F-85 |
| Paralelismo | No (IPC no vectorizado) | Target 1,024 envs |

**La brecha de fricción (0.5 vs 0.15 default) es la causa más probable del slippage.**
Ver [[UR10e_Gripper_Cloth_Issues]] para el análisis completo y la corrección.

## Por qué IPC y no PBD para manipulación con robot

FEM.Cloth + IPC da contacto **intersection-free**: los dedos del gripper no pueden
atravesar la tela aunque la velocidad sea alta. PBD.Cloth con coupling rígido puede
tener interpenetración si `dt` es grande o la malla poco densa.

**Trade-off para nuestro proyecto:**
- IPC: más correcto físicamente, pero CPU-only → incompatible con RL a escala (1,024 envs GPU)
- PBD: más rápido y GPU-acelerado, pero requiere calibrar `static_friction` y `dt` con cuidado

**Estrategia recomendada:** validar primero la tarea con FEM.Cloth+IPC en CPU
(confirmar que el agarre es físicamente posible), luego migrar a PBD.Cloth+GPU
con `static_friction=0.5` y `coup_friction=0.8` en el material del gripper.

## Requisitos del ejemplo

- `pip install pyuipc` (Linux/Windows x86 + NVIDIA GPU — solo para compilar IPC)
- `gs.init(backend=gs.cpu)` — IPC no soporta GPU en la versión actual
- `meshes/grid20x20.obj` — malla plana 20×20 quads (resolución media)

## Relacionado
- [[Genesis_Cloth_PBD_Examples]] — ejemplos PBD.Cloth + patrón de grabación
- [[Genesis_Config_System]] — IPCCouplerOptions y FEM.Cloth completos
- [[UR10e_Gripper_Cloth_Issues]] — slippage; cómo adaptar este ejemplo al UR10e+2F-85
- [[UR10e_2F85_Genesis]] — robot del proyecto; adaptar IK y gripper del ejemplo Franka
- [[Unphased_Wrinkles_2022]] — E_denim real para calibrar E=6e4 vs E_denim real ≈350 GPa
