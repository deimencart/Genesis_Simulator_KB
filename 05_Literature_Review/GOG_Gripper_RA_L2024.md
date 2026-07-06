---
category: [literature, robotics]
status: draft
created: 2026-07-06
updated: 2026-07-06
tags: [paper, gripper, hardware, bimanual, cloth-folding, low-relevance-to-project]
---

# G.O.G.: A Versatile Gripper-On-Gripper Design for Bimanual Cloth Manipulation with a Single Robotic Arm (IEEE RA-L, 2024)

arXiv: https://arxiv.org/abs/2401.10702
Autores: Dongmyoung Lee, Wei Chen, Xiaoshuai Chen, Nicolas Rojas (REDS Lab, Imperial College London)
Venue: IEEE Robotics and Automation Letters (RA-L), aceptado enero 2024

## Relevancia para el proyecto: BAJA

El paper propone un end-effector físico custom (no disponible comercialmente) probado
sobre UR5 (no UR10e), orientado a **folding** (no unfolding) de tela genérica (no denim),
sin simulación, sin Genesis, sin RL. La intersección con nuestro proyecto es tangencial.

**Matriz de compatibilidad:**

| Dimensión | G.O.G. | Nuestro proyecto |
|-----------|--------|-----------------|
| Brazo | UR5 | UR10e |
| Gripper | Custom G.O.G. (hardware) | Robotiq 2F-85 (comercial) |
| Tarea | Folding + spreading | **Unfolding** de mezclilla |
| Dominio | Hardware físico real | Simulación (Genesis) |
| RL/control | Heurístico (corner detection) | RL / policy learning |
| Sim-to-real | No | Sí (objetivo del proyecto) |

**Conclusión:** No es referencia de arquitectura, ni de física de tela, ni de diseño
de env Gym. Se documenta por completitud del literature review.

## Qué es G.O.G.

Diseño jerárquico de end-effector que simula bimanualidad con un solo brazo:

- **Width Control Gripper (WCG):** regula la separación entre dedos hasta **500 mm**
  (motor NEMA 17 + transmisión por tendón). Permite agarrar dos esquinas de la tela
  simultáneamente a distintas distancias.
- **Variable Friction Grippers (VFG):** cada dedo tiene dos modos de agarre:
  - *Firm*: almohadillas de silicona — para sostener con firmeza.
  - *Sliding*: rodillo TPU — para deslizar la tela sobre la superficie.
  - Conmutación pasiva (resorte de goma), sin señal de control explícita.

Hardware 100% custom: impresión 3D + actuadores comerciales. No es un wrapper ni
adaptador para el 2F-85 ni para ningún gripper estándar.

## Tareas evaluadas

Folding en 1 y 2 pliegues, spreading, hanging, dragging — todas sobre tela plana
rectangular en superficie plana. **No evalúa unfolding** desde estado arrugado.

## Resultados principales

| Tarea | Métrica | Resultado |
|-------|---------|-----------|
| Folding 1 pliegue | MIoU | 0.917 |
| Folding 2 pliegues | MIoU | 0.868 |
| Drag placement | Offset posicional | 1–3 mm promedio |
| Lift (payload) | Fuerza máxima | >30 N |

## Limitaciones del diseño

- Span máximo 500 mm — insuficiente para sábanas grandes (>1.4 m).
- Falla con materiales de muy baja fricción (el VFG no engancha).
- Evaluado solo en UR5; montaje en otros brazos requiere re-ingeniería.
- Degradación acumulada en pliegues secuenciales (MIoU baja de 0.917 → 0.868).
- Sin datos de tiempo de ciclo ni velocidad de operación.

## Única conexión débil con el proyecto

El concepto de **agarre en dos puntos simultáneos** (dos esquinas de la tela al
mismo tiempo) sí es relevante para unfolding de denim: agarrar ambas esquinas
superiores y desplegar. Sin embargo, el 2F-85 con una sola apertura no puede
implementar esto directamente — requeriría o bien un segundo brazo, o cambiar
el diseño del gripper, o hacer dos agarres secuenciales.

Este hallazgo apunta más hacia el diseño de la **política de manipulación** (qué
puntos de agarre elegir) que hacia el diseño del gripper.

## Relacionado
- [[UR10e_2F85_Genesis]] — nuestro robot/gripper; G.O.G. no es compatible directo
- [[Unfolding_Cloth_Review_2024]] — contextualiza G.O.G. dentro del survey de cloth manipulation
- [[GarmentLab_NeurIPS2024]] — también evalúa folding con distintos end-effectors (Franka, Shadow Hand)
