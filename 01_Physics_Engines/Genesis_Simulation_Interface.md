---
category: [simulation]
status: validated
created: 2026-07-06
updated: 2026-07-06
tags: [genesis, robot-loading, dof-control, parallel-envs, sensors, camera, api]
---

# Genesis World — Simulation Interface: Robots, DOFs, Parallelismo y Sensores

Fuente: `examples/tutorials/` y `examples/sensors/` del repo `Genesis-Embodied-AI/genesis-world`.

## Carga de robots (MJCF / URDF)

De `examples/tutorials/control_your_robot.py`:

```python (genesis_env)
import genesis as gs

gs.init(backend=gs.gpu)
scene = gs.Scene(...)

# Carga desde MJCF (recomendado para robots con accionadores bien definidos)
franka = scene.add_entity(
    gs.morphs.MJCF(file="xml/franka_emika_panda/panda.xml"),
)
# Para UR10e:
ur10e = scene.add_entity(
    gs.morphs.MJCF(file="xml/universal_robots_ur10e/ur10e_2f85_rk4.xml"),
)

scene.build()
```

Otros morfologías de carga: `gs.morphs.URDF(file=...)`, `gs.morphs.Mesh(...)`,
`gs.morphs.Box(...)`, `gs.morphs.Sphere(...)`, `gs.morphs.Plane()`.

## Control de DOFs

```python (genesis_env)
# 1. Obtener índices de joints por nombre
joints_name = ("joint1", "joint2", "joint3", "joint4",
               "joint5", "joint6", "finger_joint1", "finger_joint2")
motors_dof_idx = [
    franka.get_joint(name).dofs_idx_local[0] for name in joints_name
]

# 2. Configurar ganancias de control
franka.set_dofs_kp([...], motors_dof_idx)          # ganancia proporcional
franka.set_dofs_kv([...], motors_dof_idx)          # amortiguación
franka.set_dofs_force_range([min,...], [max,...], motors_dof_idx)  # límites

# 3. Tres modos de control
franka.control_dofs_position(targets, motors_dof_idx)   # posición (rad)
franka.control_dofs_velocity(targets, motors_dof_idx)   # velocidad (rad/s)
franka.control_dofs_force(commands, motors_dof_idx)     # torque (N·m)

# 4. Lectura de estado
force_cmd = franka.get_dofs_control_force(motors_dof_idx)  # fuerza comandada
force_act = franka.get_dofs_force(motors_dof_idx)          # fuerza real
q_pos     = franka.get_dofs_position(motors_dof_idx)       # posición actual
```

**Gripper (2F-85 adaptado del ejemplo Franka):**
```python (genesis_env)
fingers_dof = [ur10e.get_joint("finger_joint1").dofs_idx_local[0],
               ur10e.get_joint("finger_joint2").dofs_idx_local[0]]
# 0.04 = abierto; 0.0 o −0.03 = cerrado (verificar con el MJCF del 2F-85)
ur10e.control_dofs_position([0.04, 0.04], fingers_dof)
```

## Kinematics inversa (IK)

```python (genesis_env)
# IK para el end-effector
q_ik = ur10e.inverse_kinematics(
    link=ur10e.get_link("tool0"),
    pos=target_pos,      # np.array shape (3,)
    quat=target_quat,    # np.array shape (4,)
)
ur10e.control_dofs_position(q_ik[motors_dof_idx], motors_dof_idx)
```

## Simulación paralela (n_envs)

De `examples/tutorials/parallel_simulation.py`:

```python (genesis_env)
B = 20  # número de entornos paralelos
scene.build(n_envs=B, env_spacing=(1.0, 1.0))

# Acción idéntica para todos los envs — tensor shape (B, n_dofs)
franka.control_dofs_position(
    torch.tile(torch.tensor([0, 0, -1.0, 0, 1.0, 0, 0.02, 0.02],
               device=gs.device), (B, 1)),
)

# Acción por envs específicos — tensor shape (len(envs_idx), n_dofs)
franka.control_dofs_position(
    position=torch.zeros(3, 9, device=gs.device),
    envs_idx=torch.tensor([1, 5, 7], device=gs.device),
)

scene.step()  # avanza TODOS los envs en paralelo
```

Para el [[Gym_Wrapper_Genesis]]: el target es `n_envs=1024` con física GPU (CUDA).

## Cámaras

```python (genesis_env)
# Cámara offscreen (independiente del viewer, válida para headless)
cam = scene.add_camera(
    res=(640, 480),         # resolución en píxeles
    pos=(3.5, 0.0, 2.5),
    lookat=(0, 0, 0.5),
    fov=30,
    GUI=True,               # False para entrenamiento headless
)
# Para ray tracing (alta calidad): añadir spp=512 y renderer=gs.renderers.RayTracer(...)

# Render de un frame:
rgb, depth, seg, normal = cam.render(rgb=True, depth=True, segmentation=True, normal=True)

# Pose dinámica:
cam.set_pose(pos=(...), lookat=(...))

# Grabación video:
cam.start_recording()
for i in range(N): scene.step(); cam.render()
cam.stop_recording(save_to_filename="video.mp4", fps=60)
```

## Sensores disponibles (examples/sensors/)

| Ejemplo | Sensor |
|---------|--------|
| `camera_as_sensor.py` | RGB/Depth/Segmentation offscreen |
| `contact_force_go2.py` | Fuerza de contacto por link |
| `imu_franka.py` | IMU (acelerómetro + giroscopio) |
| `joint_torque_franka.py` | Torque por joint |
| `lidar_teleop.py` | LiDAR (point cloud) |
| `surface_distance_shadowhand.py` | Distancia superficie-objeto |
| `tactile_franka.py` | Sensor táctil |
| `temperature_grid.py` | Temperatura (para fluidos) |

## Patrones de acceso a estados de la tela (PBD)

```python (genesis_env)
# Posiciones de partículas PBD → observación para RL
particles_pos = cloth.get_particles()        # shape: (n_particles, 3) o (n_envs, n_particles, 3)

# Fijar partícula específica (anclar esquina):
idx = cloth.find_closest_particle((x, y, z))
cloth.fix_particles(idx)
```

## Relacionado
- [[Genesis_Simulator]] — hub; 4 capas del motor
- [[Genesis_Config_System]] — todas las Options y materiales
- [[Genesis_Cloth_PBD_Examples]] — ejemplos PBD cloth + patrón grabación
- [[Genesis_Cloth_IPC_Examples]] — ejemplo robot+cloth IPC (hallazgo friction)
- [[Gym_Wrapper_Genesis]] — implementación del wrapper Gym sobre esta interfaz
- [[UR10e_2F85_Genesis]] — carga específica del UR10e + 2F-85
