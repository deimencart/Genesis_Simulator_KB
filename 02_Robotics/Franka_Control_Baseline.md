---
category: [robotics, simulation]
status: draft
created: 2026-07-21
updated: 2026-07-21
tags: [franka, control, kp, kv, force-range, baseline, position-control, velocity-control, force-control]
---

# Franka — Baseline de Carga y Control (Fase 0)

Patrón completo de carga y control del Franka en Genesis, reconstruido desde
los scripts reales de `~/genesis_ws/phase_0_franka/`. Consolida 4 de los 6
ítems de Fase 0 del [[ROADMAP]]; los otros 2 (waypoints+smoothstep,
documentación) se tratan aquí también, marcando honestamente lo que falta.

## 1. Carga del robot con MJCF

`1_load_franka.py` — patrón mínimo, verificado:

```python (genesis_env)
robot = scene.add_entity(
    gs.morphs.MJCF(file="xml/franka_emika_panda/panda.xml", pos=(0, 0, 0)),
)
scene.build()
```

## 2. Índices de joints por nombre

Patrón repetido en todos los scripts posteriores a `1_load_franka.py`
(`2_franka_mapping_joints.py`, `2_1`, `2_2`, `5_2`):

```python (genesis_env)
joints_name = ("joint1", "joint2", "joint3", "joint4", "joint5", "joint6", "joint7")
motors_dof_idx = [robot.get_joint(name).dofs_idx_local[0] for name in joints_name]
```

## 3. Ganancias: `set_dofs_kp`, `set_dofs_kv`, `set_dofs_force_range`

Tres variantes reales encontradas, cada una con un propósito distinto:

**a) Valores de referencia tipo datasheet Franka** (`2_franka_mapping_joints.py`):
```python (genesis_env)
robot.set_dofs_kp([4500.0, 4500.0, 3500.0, 3500.0, 2000.0, 2000.0, 2000.0], motors_dof_idx)
robot.set_dofs_kv([450.0, 450.0, 350.0, 350.0, 200.0, 200.0, 200.0], motors_dof_idx)
# Dedos: mucha menos rigidez, requieren fuerza distinta
robot.set_dofs_kp([500.0, 500.0], fingers_dof_idx)
robot.set_dofs_kv([20.0, 20.0], fingers_dof_idx)
```

**b) Tutorial oficial completo** (`2_2_robot_tuto_control_your_robot.py`, 9 DOFs
incl. dedos, con `force_range` explícito de seguridad):
```python (genesis_env)
franka.set_dofs_kp(kp=np.array([4500, 4500, 3500, 3500, 2000, 2000, 2000, 100, 100]), dofs_idx_local=motors_dof_idx)
franka.set_dofs_kv(kv=np.array([450, 450, 350, 350, 200, 200, 200, 10, 10]), dofs_idx_local=motors_dof_idx)
franka.set_dofs_force_range(
    lower=np.array([-87, -87, -87, -87, -12, -12, -12, -100, -100]),
    upper=np.array([87, 87, 87, 87, 12, 12, 12, 100, 100]),
    dofs_idx_local=motors_dof_idx,
)
```

**c) Override de `force_range` más restrictivo que el default del MJCF**,
verificado en vivo (`5_2_set_force_2joints.py`) — ver [[Franka_Force_Range_Experiment]]
Hallazgo 3 para el experimento completo.

**d) Contraste — desactivar el controlador nativo** (`2_1_moving_independent_joints.py`):
pone `kp=kv=0` y calcula un PID manual propio con `control_dofs_force`. Útil
como referencia de que Genesis solo implementa PD nativo (no `ki`).

## 4. Los 3 modos de control — posición y fuerza verificados, velocidad NO

| Modo | Método | Verificación real |
|---|---|---|
| Posición | `control_dofs_position` | Verificada con lectura de posición real (`3_inverse_kinematics.py` compara posición final vs. target; `2_franka_mapping_joints.py`, `2_2` también la usan) |
| Fuerza | `control_dofs_force` | Verificada extensamente con `get_dofs_force()` (PID manual en `2_1_moving_independent_joints.py`; `5_force_range_limits.py`, `5_1`, `5_2` con lectura + gráfica) |
| Velocidad | `control_dofs_velocity` | **Solo comandada, nunca verificada.** Aparece una única vez en todo `phase_0_franka/`, dentro de `2_2_robot_tuto_control_your_robot.py` (tutorial oficial copiado, no experimento propio): 1 DOF, steps 750-1000. El script lee `get_dofs_control_force`/`get_dofs_force` en cada step pero **nunca llama a `get_dofs_velocity()`** — no hay evidencia de que la velocidad comandada (1.0 rad/s) se haya alcanzado realmente. |

**Pendiente real:** falta un experimento propio de `control_dofs_velocity` que
lea `get_dofs_velocity()` y confirme el valor real alcanzado, igual que ya
existe para posición y fuerza.

## 5. Trayectoria con waypoints + interpolación — parcial, no cumple el criterio original

`3_inverse_kinematics.py` interpola, pero:
- Es **un solo segmento** (posición inicial → un único target de IK), no una
  secuencia de varios waypoints.
- La interpolación es **lineal** (`alpha = step / 119`), no smoothstep.

```python (genesis_env)
for step in range(120):
    alpha = step / 119.0
    interpolated_qpos = initial_qpos + (qpos - initial_qpos) * alpha
    robot.control_dofs_position(interpolated_qpos)
    scene.step()
```

`2_2_robot_tuto_control_your_robot.py` tiene varios waypoints (steps 0/250/500)
pero **sin interpolación** — son saltos directos de posición.

**Pendiente real:** ningún script combina múltiples waypoints con
interpolación smoothstep entre ellos. Este es el único sub-ítem de Fase 0 que
sigue sin resolver.

## 6. Esta nota

Cumple el ítem "documentar patrón completo" para carga, índices y ganancias
(puntos 1-3). El punto 4 (3 modos de control) queda **parcial**: posición y
fuerza verificados, velocidad solo comandada sin verificar. El punto 5
(waypoints+smoothstep) queda abierto.

## Relacionado
- [[Genesis_Simulation_Interface]] — API general de control
- [[Franka_Force_Range_Experiment]] — detalle de force_range y fuerza de reacción
- [[Franka_Workspace_Reachability]] — usa IK de forma similar a `3_inverse_kinematics.py`
- [[UR10e_Gripper_Cloth_Issues]] — hipótesis de sostén del brazo, depende de kp/kv bien configurados
