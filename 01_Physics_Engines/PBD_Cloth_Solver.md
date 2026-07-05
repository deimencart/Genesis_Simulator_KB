---
category: [simulation]
status: draft
created: 2026-07-05
updated: 2026-07-05
source: "[[Unphased_Wrinkles_2022]]"
tags: [genesis, pbd, cloth]
---

# PBD Cloth Solver (Genesis)

Solver recomendado en Genesis para tela: `gs.materials.PBD.Cloth()` +
`gs.options.PBDOptions`. Documentado en `PBDOptions`: "Position-Based Dynamics solver
for cloth, volumetric deformable objects, liquid, and particles."

## API mínima

```python (genesis_env)
cloth = scene.add_entity(
    material=gs.materials.PBD.Cloth(),
    morph=gs.morphs.Mesh(file='meshes/cloth.obj', scale=2.0, pos=(0, 0, 0.5)),
    surface=gs.surfaces.Default(vis_mode='visual'),
)
```

## Parámetros clave
- `stretch_compliance` — a menor valor, más rígido al estiramiento.
- `bending_compliance` — a menor valor, más rígido a la flexión (dobleces).
- `air_resistance`, `particle_size`.
- `max_stretch_solver_iterations`, `max_bending_solver_iterations` (en `PBDOptions`).

## Limitación importante
PBD no mapea directamente a Young's modulus / Poisson's ratio como FEM. La conversión
de `compliance` → unidades físicas reales requiere calibración empírica — ver
[[Unphased_Wrinkles_2022]] para valores de referencia de denim vs algodón.

## Configuración específica para mezclilla
Ver [[PBD_Cloth_Config_Denim]].

## Relacionado
- Hub: [[Genesis_Simulator]]
- Robot que manipula esta tela: [[UR10e_2F85_Genesis]]
