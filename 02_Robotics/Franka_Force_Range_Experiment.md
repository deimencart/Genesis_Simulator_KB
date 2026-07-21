---
category: [robotics, simulation]
status: validated
created: 2026-07-12
updated: 2026-07-21
tags: [franka, force-range, force-limits, joint-limits, dof-control, dynamics]
---

# Franka — Experimento de Force Range (límites de fuerza por DOF)

Dos scripts reales en `~/genesis_ws/phase_0_franka/`: `5_force_range_limits.py`
(v1, exploratorio, solo consola) y `5_1_graphs_force_limits.py` (v2, agrega
tracking de posición + gráfica, produce `force_and_position_over_time.png`).
Cifras de esta nota verificadas re-ejecutando ambos scripts contra el motor
Genesis instalado (0.3.13) — no tomadas de memoria de conversación.

## Método

Ambos scripts: cargan el Franka, aplican `control_dofs_force` a **un solo
DOF** repetido en cada uno de 100 steps, leyendo en cada paso
`get_dofs_force()` (fuerza real) y `get_dofs_position()` (posición real).
v1 pide 200 Nm al DOF de `joint5` (índice 4). v2 pide 12 Nm al DOF de índice
5 — que en realidad es **`joint6`**, no `joint5` como dicen su título de
gráfico y comentarios (inconsistencia real entre código y comentarios,
detectada al re-ejecutar; ver más abajo). `joint5`, `joint6` y `joint7`
comparten el mismo `force_range` real: ±12 Nm.

## Hallazgo 1 — el pico grande no es el actuador, es fuerza de reacción de contacto

**v1** (`joint5`, se pide 200 Nm — muy por encima del límite): re-ejecutado y
verificado — `get_dofs_force()` queda topado en **exactamente 12.0 Nm en el
step 0** (el límite real, no los 200 pedidos). Luego, un pico negativo de
**-177.49 Nm en el step 37**, que coincide con que la posición angular de
`joint5` cruza su límite articular (2.8973 rad) en el **step 36** — prácticamente
el mismo instante.

**v2** (`joint6`, solo se pide 12 Nm — dentro del límite, no lo excede):
re-ejecutado con el código tal cual está guardado hoy — reproduce la forma de
`force_and_position_over_time.png`: pico de **-206.29 Nm en el step 51**,
coincidiendo con que la posición cruza 2.8973 rad en el **step 43**.

**Interpretación:** `get_dofs_force()` reporta la fuerza *real total* en el
DOF, no solo el comando clampeado por `force_range`. Al chocar contra el tope
mecánico de posición, el solver de restricciones genera una fuerza de
reacción de contacto grande y transitoria — el mismo fenómeno aparece en
ambas corridas (v1 pidiendo 200 Nm, v2 pidiendo solo 12 Nm), lo que confirma
que el pico depende de chocar el límite de *posición*, no de la magnitud de
fuerza pedida. `force_range` sí topa la componente de actuador (12.0 Nm exacto
en v1, step 0), pero no impide la fuerza de reacción del contacto.

Límite de 12 Nm confirmado contra dos fuentes internas: el MJCF de Genesis
(`panda.xml`, `forcerange="-12 12"` en los actuadores de `joint5`/`6`/`7`) y
`robot.get_dofs_force_range()` en vivo. No se pudo cruzar contra
`franka_ros/franka_description/robots/panda/joint_limits.yaml` — ese repo no
está disponible en este entorno; queda como validación externa pendiente si
hace falta.

**Nota de proceso (v1→v2):** el script v2 evolucionó agregando gráfica y
guardado de imagen, pero en algún punto se cambió el DOF objetivo (de
`joint5` a `joint6`) y la fuerza pedida (de 200 a 12 Nm) sin actualizar el
título del gráfico ni el comentario que dice *"pedimos 200 Nm"*. El hallazgo
central no cambia (ver arriba), pero la etiqueta del gráfico guardado es
incorrecta tal como está.

## Hallazgo 2 — sistema acoplado: un joint controlado no sostiene al resto del brazo

Extraídas las 9 posiciones articulares del re-run de v1 (donde solo `joint5`
recibe `control_dofs_force`, ningún otro DOF recibe ganancias ni control):

| DOF | posición step 0 | posición step 99 | Δ |
|---|---|---|---|
| `joint2` | -0.007 | 1.788 | **+1.794 rad** |
| `joint4` | -0.017 | -0.252 | -0.234 rad |
| `joint1` | -0.001 | -0.076 | -0.075 rad |
| `joint6` | -0.002 | 0.147 | +0.149 rad |

`joint2` se desplaza **+1.79 rad (~103°)** en 100 steps sin recibir ningún
comando — cae bajo gravedad porque no tiene `kp`/`kv` ni fuerza activa que lo
sostenga. No es un bug: en una cadena cinemática serial, un DOF sin control
activo queda "libre" y cede al peso del resto del brazo y a la gravedad.

Contraste con el ejemplo oficial `examples/sensors/imu_franka.py`: llama a
`set_dofs_kp` / `set_dofs_kv` / `set_dofs_force_range` para los **9 DOFs
completos**, sin restringir `dofs_idx_local`, antes de cualquier control —
confirmado leyendo el archivo real. Ninguno de los dos scripts de este
experimento hace eso; ambos controlan un único DOF y dejan el resto sin
configurar.

## Hallazgo 3 — `set_dofs_force_range` propio se respeta por encima del default del MJCF

Script adicional encontrado en `~/genesis_ws/phase_0_franka/`, no incluido en
la redacción original de esta nota: `5_2_set_force_2joints.py` (12-jul).

Fija un límite propio en `joint5` (±5 Nm), **más restrictivo** que el default
del MJCF (±12 Nm), y pide una fuerza de 200 Nm — muy por encima de ambos
límites. Mientras tanto, sostiene el resto del brazo en una postura neutral
con `set_dofs_kp`/`set_dofs_kv` (aplicando directamente el Hallazgo 2 de
arriba) para que no ceda por gravedad durante la prueba.

```python (genesis_env)
robot.set_dofs_force_range(lower=np.array([-5]), upper=np.array([5]), dofs_idx_local=[joint5_dof_idx])
```

Confirma que `set_dofs_force_range` invocado explícitamente sobrescribe el
default leído del MJCF — el simulador topa contra el límite propio (5 Nm),
no contra el default (12) ni contra lo pedido (200). Cierra el ítem 3 de
Fase 0 del [[ROADMAP]] (`set_dofs_kp`/`set_dofs_kv`/`set_dofs_force_range`):
faltaba verificar que `set_dofs_force_range` tuviera efecto real y no solo
se leyera; este script lo confirma.

Código completo en [[Franka_Force_Range_Limits]] (v3).

## Código real (ambos scripts, patrón central)

```python (genesis_env)
# v1 — 5_force_range_limits.py: joint5 (dof idx 4), 200 Nm, solo consola
excessive_force = np.array([200])
for i in range(100):
    robot.control_dofs_force(excessive_force, dofs_idx_local=[4])
    scene.step()
    print("Step", i, "- Fuerza real aplicada:", robot.get_dofs_force()[4])
```

```python (genesis_env)
# v2 — 5_1_graphs_force_limits.py: dof idx 5 (en realidad joint6), 12 Nm,
# con tracking + gráfica (título/comentarios del script dicen "joint5"/"200Nm"
# — no coincide con el código real, ver Hallazgo 1)
excessive_force = np.array([12])
force_history, position_history = [], []
for i in range(100):
    robot.control_dofs_force(excessive_force, dofs_idx_local=[5])
    scene.step()
    force_history.append(robot.get_dofs_force()[5].item())
    position_history.append(robot.get_dofs_position()[5].item())
```

Código completo de ambos scripts, verbatim: [[Franka_Force_Range_Limits]] en
`06_Code_Snippets/`.

## Documentación exhaustiva (fuera del vault)

Desglose línea por línea y proceso de depuración completo, en
`~/genesis_ws/tutor_notes/` (capa complementaria, sin límite de tamaño).

## Relacionado

- [[Genesis_Config_System]] — `force_range` y demás parámetros de `RigidOptions`.
- [[Genesis_Simulation_Interface]] — patrón base de `control_dofs_force`.
- [[Franka_Workspace_Reachability]] — experimento hermano de esta misma fase.
- [[UR10e_Gripper_Cloth_Issues]] — hipótesis adicional de sostén insuficiente
  del brazo añadida a partir del Hallazgo 2 de esta nota.
- [[Franka_Force_Range_Limits]] — código fuente real completo.
- [[Franka_Control_Baseline]] — consolida el ítem de ganancias (kp/kv/force_range) de Fase 0.
