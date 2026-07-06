---
category: [simulation]
status: validated
created: 2026-07-06
updated: 2026-07-06
tags: [genesis, config, options, materials, api-reference]
---

# Genesis — Config System: Options y Materials

Fuente: `genesis/options/solvers.py` y `genesis/engine/materials/` (rama `main`).
Docs oficiales (429 en el momento del ingest): https://genesis-world.readthedocs.io/en/latest/user_guide/getting_started/config_system.html

## Jerarquía de configuración

```python (genesis_env)
gs.init(backend=gs.gpu)

scene = gs.Scene(
    sim_options   = gs.options.SimOptions(dt=4e-3, substeps=10),
    rigid_options = gs.options.RigidOptions(...),
    pbd_options   = gs.options.PBDOptions(...),
    fem_options   = gs.options.FEMOptions(...),
    coupler_options = gs.options.LegacyCouplerOptions(...),
)
```

Si `dt` o `gravity` no se especifican en una clase hija, heredan de `SimOptions`.

## gs.options — todos los solvers

| Clase | Descripción oficial (verbatim) |
|-------|-------------------------------|
| `SimOptions` | "Options configuring the top-level simulator." |
| `RigidOptions` | "Options configuring the RigidSolver." |
| `PBDOptions` | "Options configuring the PBDSolver." |
| `FEMOptions` | "Options configuring the FEMSolver." Damping vía Rayleigh model. |
| `MPMOptions` | "Options configuring the MPMSolver." Hybrid Lagrangian-Eulerian. |
| `SPHOptions` | "Options configuring the SPHSolver." (WCSPH o DFSPH) |
| `SFOptions` | "Options configuring the SFSolver." (Smoke/Fluid euleriano, grid-based) |
| `ToolOptions` | "Options configuring the ToolSolver." One-way coupling, sin dinámica interna. |
| `KinematicOptions` | "Options configuring the KinematicSolver." FK visual-only. |
| `LegacyCouplerOptions` | Flags de acoplamiento inter-solver (rigid↔pbd, rigid↔fem, fem↔sph…). |
| `SAPCouplerOptions` | SAP contact solver (Drake; arXiv 2110.10107). |
| `IPCCouplerOptions` | IPC contact con CCD, fricción, Newton/PCG, libuipc. Requerido para `FEM.Cloth`. |
| `ViewerOptions` | Config del viewer interactivo. |
| `VisOptions` | Config de rendering/visualización. |

## PBDOptions — parámetros completos

El solver PBD maneja cloth (stretch/bending), líquido (density/viscosity) y partículas.

| Parámetro | Tipo | Default | Nota |
|-----------|------|---------|------|
| `dt` | float\|None | None | Hereda SimOptions |
| `gravity` | tuple\|None | None | Hereda SimOptions |
| `max_stretch_solver_iterations` | int | **4** | Rigidez al estiramiento |
| `max_bending_solver_iterations` | int | **1** | Rigidez a la flexión |
| `max_volume_solver_iterations` | int | 1 | Deformables volumétricos |
| `max_density_solver_iterations` | int | 1 | Líquido |
| `max_viscosity_solver_iterations` | int | 1 | Líquido |
| `particle_size` | float | **0.01 m** | Diámetro de partícula; determina resolución |
| `hash_grid_cell_size` | float\|None | auto (≥1.25×particle_size) | Malla espacial |
| `hash_grid_res` | tuple\|None | auto (≤150³) | Si dominio > 150³ celdas, usa fallback |
| `lower_bound` | tuple | (-100, -100, 0) | Límite del dominio |
| `upper_bound` | tuple | (100, 100, 100) | Límite del dominio |

## FEMOptions — parámetros clave

| Parámetro | Tipo | Default |
|-----------|------|---------|
| `damping` | float | 0.0 |
| `damping_alpha` / `damping_beta` | float | 0.5 / 5e-4 (Rayleigh mass/stiffness) |
| `use_implicit_solver` | bool | False |
| `n_newton_iterations` | int | 1 |
| `n_pcg_iterations` | int | 500 |
| `enable_vertex_constraints` | bool | False |

## MPMOptions — parámetros clave

| Parámetro | Tipo | Default |
|-----------|------|---------|
| `grid_density` | float | 64 |
| `particle_size` | float\|None | None (auto) |
| `enable_CPIC` | bool | False |
| `lower_bound` / `upper_bound` | tuple | (-1,-1,0) / (1,1,1) |

## gs.materials — tabla completa por solver

| Solver | Material | Parámetros físicos clave |
|--------|----------|--------------------------|
| **PBD** | `PBD.Cloth` | stretch/bending compliance, friction, rho (kg/m²) |
| **PBD** | `PBD.Liquid` | rho (kg/m³), density_relaxation |
| **PBD** | `PBD.Elastic` | sólido elástico PBD |
| **PBD** | `PBD.Particle` | partícula básica |
| **FEM** | `FEM.Elastic` | E, nu, rho; modelos: linear / stable_neohookean / linear_corotated |
| **FEM** | `FEM.Cloth` | E, nu, rho, thickness, bending_stiffness; requiere IPC+GPU |
| **FEM** | `FEM.Muscle` | E, nu + n_groups, activación direccional |
| **MPM** | `MPM.Elastic` | E, nu, rho; modelos: corotation / neohooken |
| **MPM** | `MPM.ElastoPlastic` | + yield (von Mises o clamped strain) |
| **MPM** | `MPM.Liquid` | fluido MPM |
| **MPM** | `MPM.Sand` | granular seco |
| **MPM** | `MPM.Snow` | granular con plasticidad |
| **MPM** | `MPM.Muscle` | muscular MPM |
| Rígido | `Rigid` | cuerpo rígido articulado |
| Tool | `Tool` | rígido unidireccional |
| Kinematic | `Kinematic` | visual-only |
| SF | `SF` | smoke/fluid euleriano (grid) |
| SPH | `SPH` | fluido SPH (WCSPH o DFSPH) |

## gs.materials.PBD.Cloth — parámetros completos

| Parámetro | Tipo | Default | Descripción oficial (verbatim) |
|-----------|------|---------|-------------------------------|
| `rho` | float | **4.0 kg/m²** | "density of the cloth. **kg/m², not kg/m³**, as cloth is a 2D material" |
| `static_friction` | float | **0.15** | "Static friction coefficient." |
| `kinetic_friction` | float | **0.15** | "Kinetic friction coefficient." |
| `stretch_compliance` | float | **1e-7 m/N** | "The stretch compliance (m/N)." |
| `bending_compliance` | float | **1e-5 rad/N** | "The bending compliance (rad/N)." |
| `stretch_relaxation` | float | **0.3** | "The stretch relaxation. Smaller value weakens the stretch constraint." |
| `bending_relaxation` | float | **0.1** | "The bending relaxation. Smaller value weakens the bending constraint." |
| `air_resistance` | float | **1e-3** | "The air resistance. Damping force due to air drag." |

> **Atención para [[PBD_Cloth_Config_Denim]]:** `static_friction` y `kinetic_friction`
> son parámetros del material (no de PBDOptions). Sus defaults (0.15) probablemente
> son bajos para denim — calibrar junto a los compliance según [[ClothParamOpt_PBD_JCDE2025]].
> `stretch_relaxation` y `bending_relaxation` son parámetros adicionales no documentados
> en la nota de config existente.

## gs.materials.FEM.Cloth — parámetros

| Parámetro | Tipo | Default |
|-----------|------|---------|
| `E` | float | 1e4 Pa |
| `nu` | float | 0.49 |
| `rho` | float | 200.0 kg/m³ |
| `thickness` | float | 0.001 m |
| `bending_stiffness` | float\|None | None |
| `model` | str | "stable_neohookean" |
| `friction_mu` | float | 0.1 |

Restricción: requiere `IPCCouplerOptions` + backend GPU. Solo acepta mallas de superficie.
Ventaja: usa parámetros físicos reales (E, nu) directamente compatibles con [[Unphased_Wrinkles_2022]].

## PBD.Cloth vs FEM.Cloth

| Aspecto | PBD.Cloth | FEM.Cloth |
|---------|-----------|-----------|
| Parámetros | compliance (adimensional) | E, nu (físicos, medibles) |
| Velocidad | Alta (iterativa, GPU) | Lenta (Newton/IPC) |
| Req. adicionales | Ninguno | `pip install pyuipc` + GPU |
| Calibración de ref. | [[ClothParamOpt_PBD_JCDE2025]] | [[Unphased_Wrinkles_2022]] directo |
| **Elección proyecto** | ✅ Plan actual | Alternativa si PBD insuficiente |

> **Resuelve contradicción C1 del lint:** RoboVerse Tabla VI muestra Genesis con
> "Rigid; Soft" sin "Cloth" explícito. El código fuente confirma soporte tanto para
> `PBD.Cloth` como `FEM.Cloth` — RoboVerse los clasifica bajo "Soft". Ver [[RoboVerse_2025]].

## Relacionado
- [[Genesis_Simulator]] — hub; arquitectura de 4 capas
- [[PBD_Cloth_Solver]] — uso del solver PBD para tela en el proyecto
- [[PBD_Cloth_Config_Denim]] — parámetros calibrados (actualizar con friction + relaxation)
- [[Genesis_Cloth_PBD_Examples]] — código PBD cloth.obj + grabación
- [[Genesis_Cloth_IPC_Examples]] — código IPC robot+cloth (friction_mu=0.5)
- [[Genesis_Installation]] — backends y dependencias opcionales (pyuipc para FEM.Cloth)
