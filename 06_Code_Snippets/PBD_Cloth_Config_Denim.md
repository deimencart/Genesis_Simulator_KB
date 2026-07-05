---
category: [simulation, robotics]
status: draft
created: 2026-07-05
updated: 2026-07-05
source: "[[Unphased_Wrinkles_2022]]"
tags: [genesis, pbd, cloth, denim, code]
---

# Config PBD.Cloth — punto de partida para mezclilla

Base ya validada para tela ligera en este proyecto (pendiente subir rigidez para denim,
según [[Unphased_Wrinkles_2022]]):

```python (genesis_env)
gs.materials.PBD.Cloth(
    stretch_compliance=1e-7,
    bending_compliance=1e-4,   # denim: bajar este valor (más rígido a flexión)
    air_resistance=0.01,
    particle_size=0.02,
)
gs.options.PBDOptions(
    max_stretch_solver_iterations=10,
    max_bending_solver_iterations=5,
)
```

## Pendiente de calibración
- [ ] Reducir `bending_compliance` para reflejar mayor rigidez de denim vs. tela ligera.
- [ ] Ajustar fricción cloth-gripper (ver [[UR10e_2F85_Genesis]] — más determinante que
  el modelo constitutivo para el éxito del folding).
- [ ] Documentar tabla de conversión compliance → E/ν una vez calibrado contra
  [[Unphased_Wrinkles_2022]].

## Relacionado
- [[PBD_Cloth_Solver]]
