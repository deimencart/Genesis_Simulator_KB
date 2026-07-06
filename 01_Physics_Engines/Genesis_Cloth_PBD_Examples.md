---
category: [simulation]
status: validated
created: 2026-07-06
updated: 2026-07-06
tags: [genesis, pbd, cloth, examples, camera, recording, official]
---

# Genesis World — Cloth PBD: Ejemplos Oficiales y Patrón de Grabación

Fuente: `examples/tutorials/pbd_cloth.py`, `examples/coupling/cloth_on_rigid.py`,
`examples/tutorials/visualization.py` del repo `Genesis-Embodied-AI/genesis-world`.

## Patrón de cámara y grabación (reutilizable)

De `examples/tutorials/visualization.py`:

```python (genesis_env)
import math
import genesis as gs

# Cámara offscreen — independiente del viewer GUI, válida en headless
cam = scene.add_camera(
    res=(640, 480),
    pos=(3.5, 0.0, 2.5),
    lookat=(0, 0, 0.5),
    fov=30,
    GUI=True,   # False para entrenamiento headless/server
)

# Render de un frame (cualquier combinación de canales):
rgb, depth, segmentation, normal = cam.render(
    rgb=True, depth=True, segmentation=True, normal=True
)

# Grabación de video completo:
cam.start_recording()
for i in range(horizon):
    scene.step()
    cam.set_pose(                                   # cámara orbital
        pos=(3.0 * math.sin(i / 60), 3.0 * math.cos(i / 60), 2.5),
        lookat=(0, 0, 0.5),
    )
    cam.render()
cam.stop_recording(save_to_filename="video.mp4", fps=60)
```

**Viewer interactivo** (no graba — solo para desarrollo):
```python (genesis_env)
viewer_options=gs.options.ViewerOptions(
    res=(1280, 720),
    camera_pos=(x, y, z),
    camera_lookat=(x, y, z),
    camera_fov=30,
)
```

> `ViewerOptions` = viewer GUI interactivo (no headless).
> `scene.add_camera()` = cámara offscreen independiente, la única opción para RL headless.
> Ambas coexisten: viewer para desarrollo, `add_camera` para evaluación y grabación.

---

## Ejemplo 1 — PBD Cloth Tutorial

`examples/tutorials/pbd_cloth.py` — Tela suspendida, esquinas ancladas.

```python (genesis_env)
import genesis as gs

gs.init()

scene = gs.Scene(
    sim_options=gs.options.SimOptions(dt=4e-3, substeps=10),
    viewer_options=gs.options.ViewerOptions(
        camera_fov=30,
        res=(1280, 720),
    ),
    show_viewer=True,
)

cloth_1 = scene.add_entity(
    material=gs.materials.PBD.Cloth(),
    morph=gs.morphs.Mesh(
        file="meshes/cloth.obj",
        scale=2.0,
        pos=(0, 0, 0.5),
        euler=(0.0, 0, 0.0),
    ),
    surface=gs.surfaces.Default(
        color=(0.2, 0.4, 0.8, 1.0),
        vis_mode="visual",    # render superficie; alternativa: "particle"
    ),
)

cloth_2 = scene.add_entity(
    material=gs.materials.PBD.Cloth(),
    morph=gs.morphs.Mesh(
        file="meshes/cloth.obj",
        scale=2.0,
        pos=(0, 0, 1.0),
        euler=(0.0, 0, 0.0),
    ),
    surface=gs.surfaces.Default(
        color=(0.8, 0.4, 0.2, 1.0),
        vis_mode="particle",  # debug: esfera por partícula PBD
    ),
)

scene.build()

# Anclar esquinas por coordenada 3D (solo post-build)
cloth_1.fix_particles(cloth_1.find_closest_particle((-1, -1, 1.0)))
cloth_1.fix_particles(cloth_1.find_closest_particle(( 1,  1, 1.0)))
cloth_1.fix_particles(cloth_1.find_closest_particle((-1,  1, 1.0)))
cloth_1.fix_particles(cloth_1.find_closest_particle(( 1, -1, 1.0)))
cloth_2.fix_particles(cloth_2.find_closest_particle((-1, -1, 1.0)))

for _ in range(1000):
    scene.step()
```

`dt=4e-3 s` + `substeps=10` → frecuencia efectiva 2,500 Hz.

---

## Ejemplo 2 — Cloth sobre cuerpo rígido (Coupler)

`examples/coupling/cloth_on_rigid.py` — PBD.Cloth cayendo sobre esfera fija.

```python (genesis_env)
import genesis as gs

gs.init(backend=gs.gpu, precision="32", logging_level="info")

scene = gs.Scene(
    sim_options=gs.options.SimOptions(dt=2e-3, substeps=10),
    pbd_options=gs.options.PBDOptions(particle_size=1e-2),
    viewer_options=gs.options.ViewerOptions(
        camera_pos=(1.5, 0.0, 1.0),
        camera_lookat=(0.0, 0.0, 0.0),
        camera_fov=40,
    ),
    vis_options=gs.options.VisOptions(rendered_envs_idx=[0]),
    show_viewer=False,
)

# Rígido con acoplamiento PBD habilitado, fricción nula
frictionless_rigid = gs.materials.Rigid(needs_coup=True, coup_friction=0.0)

plane  = scene.add_entity(morph=gs.morphs.Plane(), material=frictionless_rigid)
sphere = scene.add_entity(
    morph=gs.morphs.Sphere(radius=0.2, pos=(0, 0, 0), fixed=True),
    material=frictionless_rigid,
)
cloth = scene.add_entity(
    material=gs.materials.PBD.Cloth(),
    morph=gs.morphs.Mesh(
        file="meshes/cloth.obj",
        scale=1.0,
        pos=(0.0, 0.0, 0.3),
        euler=(180.0, 0.0, 0.0),  # boca abajo
    ),
    surface=gs.surfaces.Default(color=(0.2, 0.4, 0.8, 1.0)),
)

scene.build(n_envs=0)

for _ in range(500):
    scene.step()
```

**Puntos clave:**
- `Rigid(needs_coup=True)` habilita el acoplamiento con el solver PBD.
- `coup_friction=0.0` = sin fricción rígido↔tela; default es `0.1`.
- Para el gripper del [[UR10e_2F85_Genesis]]: usar `coup_friction=0.8–1.0`.
- `VisOptions(rendered_envs_idx=[0])` renderiza solo el primer env en modo multi-env.

## Relacionado
- [[PBD_Cloth_Solver]] — parámetros del material PBD.Cloth (static_friction, compliance…)
- [[Genesis_Config_System]] — PBDOptions, ViewerOptions, Rigid material completo
- [[Genesis_Cloth_IPC_Examples]] — ejemplo robot+cloth con FEM.Cloth e IPC (hallazgo friction)
- [[UR10e_Gripper_Cloth_Issues]] — slippage gripper; coup_friction y static_friction a calibrar
- [[PBD_Cloth_Config_Denim]] — config calibrada para mezclilla
