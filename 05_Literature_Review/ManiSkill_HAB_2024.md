---
category: [literature, training, simulation]
status: draft
created: 2026-07-06
updated: 2026-07-06
tags: [paper, benchmark, gym-wrapper, gpu-accelerated, maniskill, rigid-only, architecture]
---

# ManiSkill-HAB: A Benchmark for Low-Level Manipulation in Home Rearrangement Tasks (2024)

arXiv: https://arxiv.org/abs/2412.13211
Autores: Arth Shukla, Stone Tao, Hao Su (Hillbot Inc. / UC San Diego)
Venue: preprint arXiv, diciembre 2024

## Por qué importa para el proyecto

Referencia de arquitectura para wrappers gym GPU-accelerated. Logra **4,109 SPS con
1,024 entornos paralelos y rendering activo** — el nivel de throughput que debería
aspirar [[Gym_Wrapper_Genesis]]. Complementa [[RoboVerse_2025]]: MetaSim da portabilidad
multi-simulador; MS-HAB da el patrón de rendimiento máximo en un solo simulador.

**Importante:** MS-HAB maneja **solo objetos rígidos** — sin cloth, sin deformables.
Es referencia de arquitectura general, no de física de tela.

## Arquitectura del entorno Gym sobre ManiSkill3

```
ManiSkill3 (GPU physics + vectorized step)
        ↓
SequentialTask (clase base de MS-HAB)
 ├── step() vectorizado sobre N=1024 envs en GPU
 ├── observaciones sliceadas por subtask
 └── success/failure por índice de episodio
        ↓
{SubtaskName}SubtaskTrain
 ├── dense reward por end-effector
 ├── spawn randomization
 └── event logging (éxito/fallo filtrable)
        ↓
Gym API estándar: reset() / step() / observation_space / action_space
```

**Observation space:**
- 2 imágenes depth 128×128 (cámara cabeza + brazo)
- Pose de objetos (ground-truth del simulador)
- Estado del gripper (binario), TCP pose, propriocepción
- Espacios estáticos con masking para ejecución multi-subtask en paralelo

**Action space:** continuo normalizado [-1, 1]; control PD delta-posición en joints
del brazo + velocidades de base móvil.

## Rendimiento: 4,109 SPS con rendering

| Configuración | SPS |
|--------------|-----|
| 1,024 envs paralelos + 2 imágenes depth 128×128 + física 100 Hz | **4,109 ± 26** |
| Habitat 2.0 (referencia CPU) | ~1,400 |
| RoboCasa (sin rendering) | 31.9 |

**Cómo lo logran:**
1. GPU batching: physics step y rendering ejecutados en batch sobre todos los envs.
2. Frecuencia desacoplada: física a 100 Hz, control a 20 Hz (5 pasos de física por acción).
3. Observaciones estáticas: espacios de obs fijos (sin restructuración dinámica) → stable policy training.
4. ManiSkill3 como backend: hereda GPU parallelism nativo; MS-HAB añade tareas complejas encima.

**Referencia para Genesis:** Genesis apunta a velocidades similares (10–80× más rápido que Isaac Gym). El patrón de 1,024 envs paralelos + obs estáticas + física desacoplada del control es el target de diseño para [[Gym_Wrapper_Genesis]].

## Objetos deformables: NO soportados

MS-HAB usa exclusivamente YCB (objetos rígidos) + articulaciones rígidas (frigorífico, cajones). Los autores justifican explícitamente: "such complicated features often slow down simulation speed" — deformables, cloth y audio sacrificarían el throughput de 4,000 SPS.

**Conclusión para el proyecto:** MS-HAB es referencia de **arquitectura y velocidad**, no de física de tela. Para cloth deformable seguir [[SoftGym_CoRL2020]] (referencia de tareas) y [[ClothParamOpt_PBD_JCDE2025]] (referencia de parámetros PBD).

## Cinco lecciones de diseño para [[Gym_Wrapper_Genesis]]

| Lección | Descripción |
|---------|-------------|
| **1. Obs estáticas con masking** | Espacios de obs fijos; usar máscaras para multi-subtask, no reestructurar dinámicamente |
| **2. Dense reward hooks** | RL de bajo nivel requiere rewards densos por embodiment; el wrapper debe exponerlos como hooks |
| **3. Event logging** | Exponer API de logging de eventos del simulador para filtrar trayectorias de IL |
| **4. Física desacoplada del control** | Física a alta frecuencia (100 Hz), acción a baja (20 Hz); configurable en wrapper |
| **5. Políticas por objeto** | Una sola política universal rinde 10-15% peor; soportar composición de subentornos |

## Comparación con [[RoboVerse_2025]] (MetaSim)

| Aspecto | ManiSkill-HAB | RoboVerse / MetaSim |
|---------|--------------|---------------------|
| Foco | Throughput máximo (4,109 SPS) | Portabilidad multi-simulador |
| Simuladores | 1 (ManiSkill3/SAPIEN) | 9+ (Genesis, Isaac, MuJoCo…) |
| Cloth/deformables | No | Genesis lo soporta (no unificado) |
| Arquitectura wrapper | SequentialTask + GPU batching | Handler + Env (agnóstico) |
| Aplicación al proyecto | Patrón de throughput y obs design | Patrón de abstracción Handler |

**Para [[Gym_Wrapper_Genesis]]:** combinar la **abstracción Handler** de MetaSim con las
**técnicas de throughput** de MS-HAB (obs estáticas, física desacoplada, event logging).

## Algoritmos evaluados y resultados

| Subtask | Algoritmo | Mejor resultado |
|---------|-----------|----------------|
| Pick (por objeto) | SAC | 81.75% (TidyHouse) |
| Pick (todos objetos) | SAC | 71.63% (TidyHouse) |
| Open/Close | PPO | Variable por configuración |
| Tarea completa (TidyHouse) | Encadenamiento | ~16% |

**Nota relevante:** políticas por objeto superan en 10-15% a políticas universales —
análogo al hallazgo de [[GarmentLab_NeurIPS2024]] donde métodos específicos (UGM)
superan métodos genéricos (RL puro).

## Limitaciones reconocidas

- Solo objetos rígidos (YCB dataset).
- Un solo robot (Fetch); no generaliza a UR10e ni grippers distintos.
- Sin transferencia real demostrada ("we do not claim transfer to real robots").
- Tareas de largo horizonte con tasa de completado baja (~6-25%).
- IL (behavior cloning) significativamente peor que RL.

## Relacionado
- [[Gym_Wrapper_Genesis]] — patrón de throughput + obs design de referencia
- [[RoboVerse_2025]] — complementario: Handler+Env agnóstico vs GPU throughput
- [[SoftGym_CoRL2020]] — referencia de tareas de tela (MS-HAB no cubre deformables)
- [[GarmentLab_NeurIPS2024]] — convergencia: políticas específicas > universales
