---
category: [robotics, simulation]
status: validated
created: 2026-07-05
updated: 2026-07-05
tags: [ur10e, robotiq-2f85, genesis]
---

# UR10e + Gripper 2F-85 en Genesis

Ruta de carga ya validada en este proyecto:
`xml/universal_robots_ur10e/ur10e_2f85_rk4.xml`, `pos=(0,0,0)`.

## Control
- `dof_idx_local` para obtener índices de joints.
- Tres comandos de control: `control_dofs_position`, `control_dofs_velocity`,
  `control_dofs_force`.
- `scene.step()` obligatorio en cada iteración del loop.
- Interpolación suave de trayectorias: función `smoothstep`.

## Uso con tela
Este robot es el manipulador en las tareas de folding/manipulación sobre
[[PBD_Cloth_Solver]]. Pendiente: robustecer el agarre (slippage detectado en pruebas
previas — fricción del gripper sobre la tela es más determinante que el modelo
constitutivo de la tela para el éxito del folding).

## Relacionado
- [[Genesis_Simulator]]
- [[PBD_Cloth_Config_Denim]]
