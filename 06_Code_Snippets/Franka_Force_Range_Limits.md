---
category: [robotics, simulation]
status: validated
created: 2026-07-12
updated: 2026-07-21
source: "[[Franka_Force_Range_Experiment]]"
tags: [franka, force-range, force-limits, code]
---

# Franka — Force Range Limits (código real, v1 y v2)

Código real completo de `~/genesis_ws/phase_0_franka/`. Contexto, hallazgos y
cifras verificadas en [[Franka_Force_Range_Experiment]].

## v1 — `5_force_range_limits.py` (exploratorio, solo consola)

Pide 200 Nm a `joint5` (dof índice 4, límite real 12 Nm). Sin gráfica.

```python (genesis_env)
import argparse
import os

import numpy as np
from tqdm import tqdm

import genesis as gs
from genesis.recorders.plotters import IS_MATPLOTLIB_AVAILABLE, IS_PYQTGRAPH_AVAILABLE

gs.init(backend=gs.gpu)

scene = gs.Scene(show_viewer=True, viewer_options=gs.options.ViewerOptions(camera_pos=(0, -3.5, 2.5), camera_lookat=(0.0, 0.0, 0.5), camera_fov=30))
scene.add_entity(gs.morphs.Plane())
robot = scene.add_entity(gs.morphs.MJCF(file="xml/franka_emika_panda/panda.xml"))
scene.build()

print(robot.get_dofs_force_range())

# Intenta pedirle al joint5 (límite real: 12 Nm) una fuerza que excede su límite
excessive_force = np.array([200])  # muy por encima del límite de 12 Nm

robot.control_dofs_force(excessive_force, dofs_idx_local=[4])  # índice del joint5 (0-indexed: 4)

for i in range(100):
    robot.control_dofs_force(excessive_force, dofs_idx_local=[4])
    scene.step()
    print("Step", i, "- Fuerza real aplicada:", robot.get_dofs_force()[4])
```

## v2 — `5_1_graphs_force_limits.py` (con tracking + gráfica)

⚠️ Tal como está guardado, controla el DOF de índice **5** (que es `joint6`,
no `joint5`) con **12 Nm** (no 200) — el título del gráfico y el comentario
`# pedimos 200 Nm...` no coinciden con el código real. El hallazgo del pico
de fuerza por contacto se reproduce igual (ver
[[Franka_Force_Range_Experiment]], Hallazgo 1) porque no depende de cuál DOF
ni de cuánta fuerza se pida — pero la etiqueta "Joint5"/"200 Nm" del `.png`
guardado es incorrecta.

```python (genesis_env)
import numpy as np
import genesis as gs
import matplotlib.pyplot as plt

# ---- Inicialización de Genesis ----
gs.init(backend=gs.gpu)

# ---- Escena con viewer visible (para ver el brazo caer/moverse en vivo) ----
scene = gs.Scene(
    show_viewer=True,
    viewer_options=gs.options.ViewerOptions(
        camera_pos=(0, -3.5, 2.5),
        camera_lookat=(0.0, 0.0, 0.5),
        camera_fov=30
    )
)

scene.add_entity(gs.morphs.Plane())  # el suelo
robot = scene.add_entity(gs.morphs.MJCF(file="xml/franka_emika_panda/panda.xml"))
scene.build()  # a partir de aquí ya no se pueden agregar entidades nuevas

# Confirmamos (ya lo hicimos antes) que los límites de fuerza vienen del XML
print("Límites de fuerza por joint:", robot.get_dofs_force_range())

# ---- El experimento: pedir una fuerza excesiva al joint5 ----
excessive_force = np.array([12])  # pedimos 200 Nm, sabiendo que el límite real es 12 Nm

# ---- Listas vacías para ir guardando el historial, step por step ----
# Esto es necesario porque scene.step() no "recuerda" nada del pasado por sí solo;
# si queremos graficar la EVOLUCIÓN en el tiempo, tenemos que ir guardando
# nosotros mismos cada valor en cada iteración, en una lista de Python normal.
force_history = []
position_history = []

# ---- El loop principal: 100 pasos de simulación ----
for i in range(100):
    # 1. Le seguimos pidiendo la fuerza excesiva EN CADA STEP.
    #    Si solo la pidiéramos una vez fuera del loop, el comando se "olvidaría"
    #    después del primer step (esto ya lo comprobamos en un experimento anterior).
    robot.control_dofs_force(excessive_force, dofs_idx_local=[5])

    # 2. Avanzamos la física un paso. Aquí es donde Genesis calcula qué le pasa
    #    realmente al robot (gravedad, colisiones, límites de fuerza, etc.)
    scene.step()

    # 3. Leemos qué pasó DE VERDAD en este step (no lo que pedimos, sino el resultado real)
    current_force = robot.get_dofs_force()[5]      # fuerza real aplicada al joint5
    current_position = robot.get_dofs_position()[5]  # posición angular real del joint5 (radianes)

    # 4. Convertimos de tensor de GPU (torch) a número simple de Python con .item(),
    #    porque matplotlib no sabe graficar tensores de CUDA directamente.
    force_history.append(current_force.item())
    position_history.append(current_position.item())

    print(f"Step {i} - Fuerza: {current_force.item():.4f} - Posición: {current_position.item():.4f}")

# ---- Fin del loop: ya tenemos 100 valores guardados en cada lista ----
# Ahora sí podemos graficar, porque ya no dependemos de Genesis, solo de matplotlib

fig, axes = plt.subplots(2, 1, figsize=(10, 8), sharex=True)

# Gráfica de arriba: fuerza a lo largo del tiempo
axes[0].plot(force_history, label="Fuerza real aplicada")
axes[0].axhline(y=12, color='r', linestyle='--', label='Límite superior (12 Nm)')
axes[0].axhline(y=-12, color='r', linestyle='--', label='Límite inferior (-12 Nm)')
axes[0].set_ylabel("Fuerza (Nm)")
axes[0].legend()
axes[0].set_title("Joint5: fuerza pedida (200 Nm) vs. fuerza real, y su límite físico")

# Gráfica de abajo: posición angular a lo largo del tiempo
axes[1].plot(position_history, label="Posición angular real", color='orange')
axes[1].axhline(y=2.8973, color='g', linestyle='--', label='Límite posición superior')
axes[1].axhline(y=-2.8973, color='g', linestyle='--', label='Límite posición inferior')
axes[1].set_xlabel("Step de simulación")
axes[1].set_ylabel("Posición (rad)")
axes[1].legend()

plt.tight_layout()
plt.savefig("force_and_position_over_time.png")
plt.show()
```

## v3 — `5_2_set_force_2joints.py` (override de force_range propio, más restrictivo que el MJCF)

Fija `force_range=[-5, 5]` en `joint5` (default MJCF: ±12 Nm) y sostiene el
resto del brazo con `kp`/`kv` para que no ceda por gravedad. Ver Hallazgo 3
en [[Franka_Force_Range_Experiment]].

```python (genesis_env)
gs.init(backend=gs.gpu)
scene = gs.Scene(show_viewer=True, viewer_options=gs.options.ViewerOptions(
    camera_pos=(0, -3.5, 2.5), camera_lookat=(0.0, 0.0, 0.5), camera_fov=30))
scene.add_entity(gs.morphs.Plane())
robot = scene.add_entity(gs.morphs.MJCF(file="xml/franka_emika_panda/panda.xml"))
scene.build()

joints_name = ("joint1", "joint2", "joint3", "joint4", "joint5", "joint6", "joint7")
motors_dof_idx = [robot.get_joint(name).dofs_idx_local[0] for name in joints_name]
joint5_dof_idx = robot.get_joint("joint5").dofs_idx_local[0]

print("Rango default (MJCF):", robot.get_dofs_force_range())

joint5_lower = np.array([-5])
joint5_upper = np.array([5])
robot.set_dofs_force_range(lower=joint5_lower, upper=joint5_upper, dofs_idx_local=[joint5_dof_idx])
print("Rango tras set_dofs_force_range (joint5 -> ±5 Nm):", robot.get_dofs_force_range())

kp_vals = [4500.0, 4500.0, 3500.0, 3500.0, 2000.0, 2000.0, 2000.0]
kv_vals = [450.0, 450.0, 350.0, 350.0, 200.0, 200.0, 200.0]
robot.set_dofs_kp(kp_vals, motors_dof_idx)
robot.set_dofs_kv(kv_vals, motors_dof_idx)

hold_names = [name for name in joints_name if name != "joint5"]
hold_dof_idx = [robot.get_joint(name).dofs_idx_local[0] for name in hold_names]
neutral_pos = np.array([0.0, -0.3, 0.0, -2.2, 0.0, 2.0], dtype=np.float32)
robot.control_dofs_position(neutral_pos, hold_dof_idx)

excessive_force = np.array([200])
for i in range(100):
    robot.control_dofs_force(excessive_force, dofs_idx_local=[joint5_dof_idx])
    scene.step()
    print("Step", i, "- Fuerza real aplicada:", robot.get_dofs_force()[joint5_dof_idx])
```

## Relacionado

- [[Franka_Force_Range_Experiment]] — hallazgos, cifras reales verificadas y
  contexto completo del experimento.
- [[Genesis_Config_System]] — `force_range` y demás parámetros de `RigidOptions`.
- [[Franka_Control_Baseline]] — patrón consolidado de kp/kv/force_range en Fase 0.
