---
category: [robotics, simulation]
status: validated
created: 2026-07-09
updated: 2026-07-09
source: "[[Franka_Workspace_Reachability]]"
tags: [franka, inverse-kinematics, workspace, reachability, sampling, code]
---

# Franka — Sampling de Alcanzabilidad (cálculo + visualización)

Código real (versión final, ya corregida) de los dos scripts en
`~/genesis_ws/phase_0_franka/`. Contexto, hallazgos y cifras reales verificadas
en [[Franka_Workspace_Reachability]].

## 1. Cálculo — `4_1_reachable_franka.py`

Genera candidatos con exclusión cilíndrica + sobre-muestreo, clasifica con IK,
guarda `reachable_points_2.npy` / `reachable_flags_2.npy`.

```python (genesis_env)
import numpy as np
import genesis as gs

gs.init(backend=gs.cpu)

scene = gs.Scene(show_viewer=False)
scene.add_entity(gs.morphs.Plane())
robot = scene.add_entity(gs.morphs.MJCF(file="xml/franka_emika_panda/panda.xml"))
scene.build()

end_effector_link = robot.get_link("hand")

# Generamos de más para compensar los que se van a filtrar
N_target = 200
N_oversample = 500

min_radius = 0.15  # zona de exclusión cilíndrica alrededor de la base

x_range = (-0.8, 0.8)
y_range = (-0.8, 0.8)
z_range = (0.0, 1.0)

x_points = np.random.uniform(x_range[0], x_range[1], N_oversample)
y_points = np.random.uniform(y_range[0], y_range[1], N_oversample)
z_points = np.random.uniform(z_range[0], z_range[1], N_oversample)
raw_points = np.stack([x_points, y_points, z_points], axis=1)

# Distancia radial en el plano XY (ignorando Z)
radial_dist = np.sqrt(raw_points[:, 0]**2 + raw_points[:, 1]**2)

# Filtramos: nos quedamos solo con puntos FUERA del cilindro
mask = radial_dist > min_radius
filtered_points = raw_points[mask]

# Recortamos al tamaño que queríamos originalmente
candidate_points = filtered_points[:N_target]
print(f"Puntos generados: {N_oversample}, después de filtrar cilindro: {len(filtered_points)}, usados: {len(candidate_points)}")

fixed_quat = np.array(end_effector_link.get_quat())

results = []
for point in candidate_points:
    qpos, error = robot.inverse_kinematics(
        link=end_effector_link,
        pos=point,
        quat=fixed_quat,
        return_error=True,
    )
    pos_error = error[:3]
    error_magnitude = np.linalg.norm(pos_error)
    is_reachable = error_magnitude < 1e-2
    results.append((point, is_reachable))

points_array = np.array([r[0] for r in results])
reachable_array = np.array([r[1] for r in results])
np.save("reachable_points_2.npy", points_array)
np.save("reachable_flags_2.npy", reachable_array)
```

**Nota:** este script no fija una semilla de numpy (`np.random.seed`), así que
correrlo de nuevo genera candidatos distintos — las cifras reportadas en
[[Franka_Workspace_Reachability]] corresponden específicamente a los `.npy`
guardados en el run documentado, no son reproducibles byte a byte sin fijar
semilla.

## 2. Visualización — `4_2_plotting_space.py`

Carga los `.npy` de la versión corregida y dibuja una esfera verde/roja por
punto, en una escena sin física.

```python (genesis_env)
import numpy as np
import genesis as gs

gs.init(backend=gs.cpu)

points_array = np.load("reachable_points_2.npy")
reachable_array = np.load("reachable_flags_2.npy")

scene = gs.Scene(show_viewer=True)
scene.add_entity(gs.morphs.Plane())
robot = scene.add_entity(gs.morphs.MJCF(file="xml/franka_emika_panda/panda.xml"))
end_effector_link = robot.get_link("hand")

# Crear una esfera por cada punto, ANTES de scene.build()
for point, is_reachable in zip(points_array, reachable_array):
    if is_reachable:
        color = (0, 1, 0, 1)   # verde, tupla (r,g,b,alpha) en rango 0-1
    else:
        color = (1, 0, 0, 1)   # rojo

    scene.add_entity(
        morph=gs.morphs.Sphere(
            pos=tuple(point),
            radius=0.015,
        ),
        surface=gs.surfaces.Default(color=color),
    )

scene.build()

# No hay física que nos interese aquí (las esferas son solo marcadores
# visuales estáticos): scene.visualizer.update() en vez de scene.step().
for i in range(1000):
    scene.visualizer.update()
```

**Nota:** este script no fija `camera_pos`/`camera_lookat` explícitos (usa el
default de `gs.options.ViewerOptions`) ni `fixed=True`/`collision=False` en
las esferas — funciona porque la escena nunca llama a `scene.step()` (no hay
física que mueva las esferas), pero es una diferencia respecto a otros
scripts de este proyecto que sí fijan cámara explícita. Ver
[[Franka_Workspace_Reachability]], Hallazgo 5, sobre la limitación de una
sola vista de cámara para inspeccionar el workspace completo.

## Relacionado

- [[Franka_Workspace_Reachability]] — hallazgos, cifras reales verificadas y
  contexto completo del experimento.
- [[Genesis_Simulation_Interface]] — patrón base de `inverse_kinematics`.
