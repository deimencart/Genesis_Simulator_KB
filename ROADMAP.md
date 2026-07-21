# ROADMAP — Genesis World + UR10e Cloth Manipulation

Documento de planificación vivo. No sigue el frontmatter de SCHEMA.md.
Actualizar la sección **Estado actual** al completar cada tarea o fase.

**Rationale de orden:** Franka primero (Fase 0–1) para validar el pipeline
completo de control + visualización sobre un robot sin bugs conocidos, antes
de atacar el slippage del UR10e+2F-85 como problema aislado (Fase 3). Las
fases 2 (física de tela sin robot) y 3 (debug del gripper) pueden solaparse
una vez Fase 1 completada.

---

## Fase 0 — Carga y control base del robot (Franka)

**Objetivo:** cargar Franka en Genesis World, moverlo con control de
posición/velocidad/fuerza, ejecutar una trayectoria de waypoints con
interpolación suave, y documentar el patrón completo como baseline.

**Nota destino:** `02_Robotics/Franka_Control_Baseline.md` (pendiente de crear)

- [x] Cargar Franka con `gs.morphs.MJCF` (archivo XML oficial del repo Genesis World) —
  `1_load_franka.py`, documentado en [[Franka_Control_Baseline]].
- [x] Obtener índices de joints por nombre con `get_joint().dofs_idx_local` —
  patrón repetido en todos los scripts desde `2_franka_mapping_joints.py`.
- [x] Configurar ganancias: `set_dofs_kp`, `set_dofs_kv`, `set_dofs_force_range` —
  3 variantes reales verificadas (`2_franka_mapping_joints.py`,
  `2_2_robot_tuto_control_your_robot.py`, `5_2_set_force_2joints.py`); ver
  Hallazgo 3 en [[Franka_Force_Range_Experiment]] para la confirmación de que
  `set_dofs_force_range` propio se respeta sobre el default del MJCF.
- [ ] Probar los 3 modos: `control_dofs_position` / `_velocity` / `_force` —
  **parcial**: posición y fuerza verificados con lectura real
  (`get_dofs_position`/`get_dofs_force`) en varios scripts propios; velocidad
  solo comandada una vez, dentro del tutorial oficial copiado
  (`2_2_robot_tuto_control_your_robot.py`, steps 750-1000, 1 DOF), **sin
  verificar con `get_dofs_velocity()`** — falta experimento propio. Ver
  [[Franka_Control_Baseline]] sección 4.
- [ ] Trayectoria con waypoints + interpolación smoothstep entre ellos —
  **parcial**: `3_inverse_kinematics.py` interpola pero solo un segmento
  (inicio→un target IK) y con interpolación **lineal**, no smoothstep;
  `2_2_robot_tuto_control_your_robot.py` tiene varios waypoints pero sin
  interpolación (saltos directos). Falta combinar ambas cosas — ver detalle
  en [[Franka_Control_Baseline]] sección 5.
- [x] Documentar patrón completo en nota nueva, enlazando a [[Genesis_Simulation_Interface]] —
  ver [[Franka_Control_Baseline]] (cubre ítems 1-4; ítem de waypoints queda
  documentado como pendiente ahí mismo).
- [x] Mapeo de alcanzabilidad del workspace (IK sampling, esferas verde/rojo) —
  ver [[Franka_Workspace_Reachability]] y [[Franka_Reachability_Sampling]].
  Extensión no prevista en el plan original de esta fase, agregada porque
  informa directamente la Fase 3 (posible causa del slippage del UR10e).
- [x] Force range del actuador vs. fuerza de reacción por límite articular —
  ver [[Franka_Force_Range_Experiment]] y [[Franka_Force_Range_Limits]].
  Extensión no prevista en el plan original, agregada porque originó la
  hipótesis de "sostén insuficiente del brazo" documentada (como pendiente
  de verificar) en [[UR10e_Gripper_Cloth_Issues]] para Fase 3.

---

## Fase 1 — Visualización con cámara

**Objetivo:** agregar cámara al escenario de Franka, grabar un video completo,
confirmar que el pipeline visual funciona antes de complicar con tela.

**Nota destino:** `02_Robotics/Franka_Control_Baseline.md` (sección añadida)
o nota separada de cámara si crece más de ~50 líneas.

- [ ] Configurar `scene.add_camera` (resolución, posición, lookat, fov)
- [ ] Grabar video con `start_recording` / `stop_recording(save_to_filename=...)`
- [ ] Añadir cámara orbital o seguimiento de end-effector con `set_pose`
- [ ] Verificar headless (sin viewer GUI) para entrenamiento futuro
- [ ] Documentar en la nota destino, enlazando a [[Genesis_Cloth_PBD_Examples]]
  (ya contiene el patrón completo de grabación)

---

## Fase 2 — Calibración de telas sin robot

**Objetivo:** probar 3 materiales con 3 tests cada uno. Sin robot — solo
física de tela para validar los parámetros antes de introducir el gripper.

**Nota destino:** `06_Code_Snippets/Cloth_Material_Tests.md` (pendiente de crear)

**Materiales a probar:**

| Material | Referencia de parámetros |
|----------|--------------------------|
| Algodón | [[Unphased_Wrinkles_2022]], [[ClothParamOpt_PBD_JCDE2025]] |
| Seda | [[Unphased_Wrinkles_2022]] |
| Mezclilla (denim) | [[PBD_Cloth_Config_Denim]], [[ClothParamOpt_PBD_JCDE2025]] |

**Tests por material:**

- [ ] **Test estiramiento:** fijar dos bordes opuestos, aplicar desplazamiento
  creciente en el eje perpendicular; medir elongación vs fuerza resultante.
  Comparar con valores E/compliance de la literatura.
- [ ] **Test caída libre:** tela suspendida de una esquina, observar pliegues y
  arrugas durante caída. Comparar aspecto visual con fotos reales de cada material.
- [ ] **Test caída en bucket/contenedor:** tela cayendo dentro de un cubo con
  paredes rígidas. Verificar que no hay interpenetración (tunneling) y que el
  acumulamiento es físicamente plausible.
- [ ] Documentar resultados en nota destino
- [ ] Actualizar [[PBD_Cloth_Config_Denim]] con valores finales calibrados

---

## Fase 3 — Debug estructural UR10e + gripper

**Objetivo:** resolver el slippage del UR10e+2F-85 identificado en
[[UR10e_Gripper_Cloth_Issues]], aislando si el problema es de fricción,
de estructura del MJCF, o de comprensión del 2F-85.

**Nota destino:** [[UR10e_Gripper_Cloth_Issues]] (actualizar a `status: validated`
cuando se resuelva)

- [ ] Imprimir árbol de links/joints del MJCF `ur10e_2f85_rk4.xml` (jerarquía,
  nombres exactos de los dedos) para confirmar qué links usar en `get_link()`
- [ ] Aplicar fix de fricción de [[UR10e_Gripper_Cloth_Issues]]:
  `gs.materials.Rigid(needs_coup=True, coup_friction=0.8)` + `PBD.Cloth(static_friction=0.5)`
- [ ] Registrar si el slippage mejora cualitativamente; ajustar en grid
  `coup_friction ∈ {0.5, 0.8, 1.0, 1.5}` y `static_friction ∈ {0.4, 0.5, 0.6}`
- [ ] Si persiste: comparar comportamiento de agarre Franka (Fase 0) vs UR10e+2F-85
  para aislar si el problema es específico del mecanismo del 2F-85
- [ ] Si persiste: explorar Opción B de [[UR10e_Gripper_Cloth_Issues]]:
  separar brazo y gripper como entidades distintas con materiales diferentes
- [ ] Actualizar [[UR10e_Gripper_Cloth_Issues]] con resolución y cambiar
  `status: issue` → `status: validated`

---

## Fase 4 — Integración avanzada robot + tela (UR10e)

**Objetivo:** con el gripper ya funcionando correctamente (Fase 3 completada),
probar tareas de manipulación complejas y construir la base de datos de
observaciones para futura política de RL.

**Nota destino:** `02_Robotics/UR10e_Advanced_Manipulation.md` (pendiente de crear)

- [ ] Estirar tela sostenida por el gripper (levantarla por una esquina)
- [ ] Transferir tela de un bucket/área a otro
- [ ] Colocar tela sobre una mesa, observar comportamiento físico y pliegues
- [ ] Detección de esquinas y centro de masa de la tela
  (observación input para futura política RL)
- [ ] Documentar en `02_Robotics/UR10e_Advanced_Manipulation.md`

---

## Fase 5 — Entorno Gymnasium

**Objetivo:** envolver todo lo anterior en un wrapper tipo Gym production-ready
siguiendo [[Gymnasium_API_2024]] y el patrón Handler+Env de [[RoboVerse_2025]].

**Nota destino:** [[Gym_Wrapper_Genesis]] (actualizar a `status: validated`)
+ `06_Code_Snippets/GenesisClothEnv_Template.md` (pendiente de crear)

- [ ] Extraer template `GenesisClothEnv` de [[Gymnasium_API_2024]] a nota nueva
  `06_Code_Snippets/GenesisClothEnv_Template.md`
- [ ] Definir `observation_space`: point cloud de partículas PBD de la tela
  + pose del robot (`spaces.Dict` con `Box` para partículas y joints)
- [ ] Definir `action_space`: delta de pose del gripper normalizado `[-1, 1]⁶`
- [ ] Implementar `reset()` y `step()` siguiendo [[GenesisClothEnv_Template]];
  `terminated` = cobertura ≥ umbral; `truncated` = dejar a `TimeLimit` wrapper
- [ ] Validar con `SyncVectorEnv` (recomendado en [[Gymnasium_API_2024]] para
  paralelismo GPU nativo de Genesis — sin overhead de subprocesos)
- [ ] Target de rendimiento: ~4,000 SPS con 1,024 envs paralelos
  (referencia: [[ManiSkill_HAB_2024]])

---

## Estado actual

**Fase activa:** Fase 0 — Carga y control base del robot (Franka)
**Última actualización:** 2026-07-21
**Progreso global:** 0 / 6 fases completadas (Fase 0 al 4/6 de sus ítems núcleo)

**Foco inmediato (siguiente sesión):** dos ítems abiertos en Fase 0 —
1. **Control de velocidad sin verificar** (ver [[Franka_Control_Baseline]]
   sección 4): escribir un experimento propio con `control_dofs_velocity`
   que lea `get_dofs_velocity()` y confirme el valor real alcanzado —
   posición y fuerza ya tienen esa verificación, velocidad no.
2. **Waypoints + interpolación smoothstep** (sección 5): combinar múltiples
   waypoints (patrón de `2_2_robot_tuto_control_your_robot.py`) con una
   función smoothstep real entre ellos (hoy solo hay interpolación lineal
   de un solo segmento en `3_inverse_kinematics.py`).

Al cerrar ambos, Fase 0 queda completa y se puede pasar a Fase 1 (cámara).

**Pendiente de verificar (bloquea Fase 3):** la hipótesis de "sostén
insuficiente del brazo" en [[UR10e_Gripper_Cloth_Issues]] — replicar el
experimento de [[Franka_Force_Range_Experiment]] sobre el UR10e para confirmar
si el brazo cede durante el pico de fuerza de agarre.
