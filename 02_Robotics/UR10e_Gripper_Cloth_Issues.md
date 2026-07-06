---
category: [robotics, simulation]
status: issue
created: 2026-07-06
updated: 2026-07-06
tags: [ur10e, gripper, 2f-85, cloth, slippage, friction, contact, pbd]
---

# UR10e + 2F-85: Problema de Slippage en Agarre de Tela

## Problema documentado

**Issue #199 Genesis-World** (cerrado 27-Sep-2025): "Robot gripper cannot grasp cloth (slipping problem)"
- URL: https://github.com/Genesis-Embodied-AI/genesis-world/issues/199
- El reportante prueba con reducción de substeps y control de posición/fuerza sin éxito
- No hay respuesta oficial de los maintainers visible en el issue

El mismo comportamiento es el "slippage detectado en pruebas previas" de [[UR10e_2F85_Genesis]].

## Análisis de causas

### Causa 1 — Fricción insuficiente (más probable)

`gs.materials.PBD.Cloth()` tiene `static_friction=0.15` y `kinetic_friction=0.15` como defaults.
El ejemplo oficial de robot + tela (`ipc_robot_cloth_teleop.py`) usa `FEM.Cloth` con `friction_mu=0.5`.

| Material | Parámetro | Valor | Fuente |
|----------|-----------|-------|--------|
| `PBD.Cloth` default | `static_friction` | 0.15 | código fuente Genesis |
| `PBD.Cloth` default | `kinetic_friction` | 0.15 | código fuente Genesis |
| `FEM.Cloth` (ejemplo oficial robot+cloth) | `friction_mu` | **0.5** | `ipc_robot_cloth_teleop.py` |
| Denim real | μ estimado | ~0.4–0.6 | refs. de materiales textiles |

**La fricción default de PBD.Cloth (0.15) es ~3× menor que la usada en el ejemplo oficial.**
Esto explica directamente el slippage: el gripper no genera suficiente fuerza de fricción tangencial.

### Causa 2 — Acoplamiento rigid-PBD sin parámetros de fricción del rígido

En `cloth_on_rigid.py` (ejemplo oficial) el rígido se define con `coup_friction=0.0` explícito.
Para el gripper, el parámetro correcto sería un valor alto:

```python (genesis_env)
# INCORRECTO para agarre:
gripper_mat = gs.materials.Rigid(needs_coup=True, coup_friction=0.0)

# CORRECTO para agarre de tela:
gripper_mat = gs.materials.Rigid(needs_coup=True, coup_friction=1.0)  # o más alto
```

`coup_friction` en `Rigid` controla la fricción del rígido con los sólidos PBD acoplados.
Sin este parámetro o con valor 0, el gripper desliza sin resistencia sobre la tela.

### Causa 3 — Número de contactos insuficiente (secundaria)

`RigidOptions.max_collision_pairs=150` (default). Con cloth de alta resolución
(~2500 partículas), solo una fracción interactúa con el gripper. Aumentar puede ayudar
si las causas 1 y 2 ya están resueltas.

### Causa 4 — dt muy grande para PBD cloth + gripper dinámico

A velocidades de gripper rápidas, `dt` grande → partículas atraviesan los dedos del gripper
(tunneling). Los ejemplos oficiales usan `dt=2e-3–4e-3 s` con `substeps=10`.

## Parámetros a ajustar

| Parámetro | Ubicación | Valor actual | Valor sugerido |
|-----------|-----------|-------------|----------------|
| `static_friction` | `PBD.Cloth(...)` | 0.15 | **0.4–0.6** (calibrar con [[ClothParamOpt_PBD_JCDE2025]]) |
| `kinetic_friction` | `PBD.Cloth(...)` | 0.15 | **0.4** (ligeramente menor que estática) |
| `coup_friction` | `Rigid(needs_coup=True, ...)` | no especificado | **0.8–1.0** |
| `dt` | `SimOptions` | verificar | 2e-3–4e-3 con substeps=10 |

## Ejemplo corregido para gripper

```python (genesis_env)
# Tela con fricción realista para denim
cloth = scene.add_entity(
    material=gs.materials.PBD.Cloth(
        static_friction=0.5,   # punto de partida; calibrar
        kinetic_friction=0.4,
        stretch_compliance=1e-7,
        bending_compliance=1e-4,
    ),
    morph=gs.morphs.Mesh(file="meshes/denim_50cm.obj", scale=1.0),
)

# Gripper con acoplamiento y fricción habilitados
gripper_material = gs.materials.Rigid(
    needs_coup=True,
    coup_friction=1.0,
)
# Aplicar este material a las entidades de los dedos del 2F-85
# (requiere separar el MJCF o sobreescribir material por link)
```

## Diferencias con ejemplo oficial (IPC robot cloth)

El ejemplo `ipc_robot_cloth_teleop.py` evita estos problemas usando:
- `FEM.Cloth` en lugar de `PBD.Cloth` → contacto intersection-free (IPC)
- `IPCCouplerOptions` con `contact_resistance=1e7` → fuerza de contacto muy alta
- `friction_mu=0.5` directamente en el material de la tela

**Limitación del IPC para nuestro caso:** requiere CPU backend + `pip install pyuipc`,
incompatible con el objetivo de paralelización GPU para RL.

## Alternativa IPC como validación

Implementar primero con FEM.Cloth+IPC en CPU para confirmar que la tarea es resoluble,
y luego migrar a PBD.Cloth+GPU ajustando fricción. Esto evita perseguir bugs de PBD
en una tarea que podría ser irresoluble con PBD a la resolución de malla actual.

## Asignar coup_friction solo a los dedos del gripper (investigación)

La pregunta es: ¿cómo evitar que todo el brazo UR10e tenga `coup_friction` alto,
ya que `gs.materials.Rigid(needs_coup=True, coup_friction=...)` aplica al nivel de entidad?

**Hallazgos del código fuente de Genesis World:**

`gs.materials.Rigid` tiene DOS parámetros de fricción distintos:
- `friction` (default None → 1.0): fricción para contacto **rigid↔rigid** (brazo vs suelo, etc.)
- `coup_friction` (default **0.1**): fricción para contacto **rigid↔PBD** (gripper vs tela)

`RigidLink.set_friction(value)` existe y aplica a todos los geoms del link:
```python (genesis_env)
# Ajustar fricción rigid-rigid por link (post-build):
ur10e.get_link("left_inner_finger_pad").set_friction(1.5)
ur10e.get_link("right_inner_finger_pad").set_friction(1.5)
```
Pero `set_friction()` afecta la fricción **rigid-rigid** (`friction`), **no** `coup_friction`.
No existe API pública equivalente para `coup_friction` por link.

**Opciones disponibles:**

| Opción | Descripción | Complejidad |
|--------|-------------|------------|
| **A — Opción pragmática** | Aplicar `coup_friction=0.8` al robot completo vía `gs.materials.Rigid(needs_coup=True, coup_friction=0.8)`. Los links del brazo generalmente no tocan la tela — la fricción alta es inerte para ellos. | Mínima |
| **B — Entidades separadas** | Cargar brazo UR10e y gripper 2F-85 como entidades separadas, cada una con su propio `Rigid` material. Conectar con constraint cinemático. | Alta; requiere separar el MJCF |
| **C — MJCF geom friction** | El parser de Genesis extrae `mj_geom.friction[0]` por geom del MJCF. Especificar valores altos en los geoms de los dedos en el XML. Puede afectar `friction` (no confirmado para `coup_friction`). | Media; editar el MJCF |

**Recomendación para el proyecto:** empezar con **Opción A**. Si el brazo interfiere
con la tela (poco probable dado que el cloth está en el espacio de trabajo distal del
gripper), evaluar la Opción B.

```python (genesis_env)
# Opción A — implementación inmediata:
ur10e = scene.add_entity(
    gs.morphs.MJCF(file="xml/universal_robots_ur10e/ur10e_2f85_rk4.xml"),
    material=gs.materials.Rigid(needs_coup=True, coup_friction=0.8),
)
```

## Relacionado
- [[UR10e_2F85_Genesis]] — configuración del robot; slippage como issue abierto
- [[Genesis_Config_System]] — parámetros completos de PBD.Cloth y Rigid
- [[PBD_Cloth_Config_Denim]] — añadir static_friction=0.5 como prioridad
- [[Genesis_Cloth_IPC_Examples]] — ejemplo ipc_robot_cloth_teleop: friction_mu=0.5 oficial
- [[ClothParamOpt_PBD_JCDE2025]] — friction es el parámetro más crítico (confirma)
- [[FoldingTwoParams_FrontNR2022]] — idem desde FEM/C-IPC (confirma independientemente)
