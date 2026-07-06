---
category: [literature, training]
status: draft
created: 2026-07-05
updated: 2026-07-05
tags: [paper, benchmark, rl, cloth, deformable, gymnasium, flex, pbd]
---

# SoftGym: Benchmarking Deep RL for Deformable Object Manipulation (CoRL 2020)

arXiv: https://arxiv.org/abs/2011.07215
Autores: Xingyu Lin, Yufei Wang, Jake Olkin, David Held (CMU)
Venue: Conference on Robot Learning (CoRL 2020) — PMLR vol. 155, pp. 432-448

## Por qué importa para el proyecto

Benchmark estándar de RL para manipulación de objetos deformables, citado como referencia
en [[Unfolding_Cloth_Review_2024]]. Define las tareas de tela más usadas en la literatura
y evalúa qué algoritmos funcionan mejor. Informa el diseño de [[Gym_Wrapper_Genesis]] y
la elección de algoritmo para nuestro entorno con [[UR10e_2F85_Genesis]].

## Framework

- Construido sobre **NVIDIA FleX 1.2.0** (motor PBD, igual familia que [[PBD_Cloth_Solver]])
- API estándar compatible con **OpenAI Gym** (antecedente del actual Gymnasium)
- Soporte para modo *headless* en clústeres
- Requiere Ubuntu 16.04 + CUDA 9.2 (plataforma antigua — limitación de reproducibilidad)

## Tareas incluidas (6 en el paper principal)

| Tarea | Descripción | Tipo físico |
|-------|-------------|-------------|
| DropCloth | Soltar tela desde altura para que caiga extendida | Dinámico |
| SpreadCloth | Extender tela desde estado arrugado | Quasi-estático |
| FoldCloth | Doblar tela a la mitad | Quasi-estático |
| StraightenRope | Enderezar cuerda desde config. aleatoria | Quasi-estático |
| PourWater | Verter agua de un vaso a otro | Dinámico |
| TransportWater | Mover vaso de agua a posición objetivo | Quasi-estático |

**Nuestro caso:** SpreadCloth y FoldCloth son los más cercanos a desdoblar mezclilla
con el [[UR10e_2F85_Genesis]]. DropCloth es relevante si se incluye shake inicial.

## Observaciones disponibles

- **Dynamics Oracle**: estado completo de partículas (acceso al simulador directamente)
- **Reduced State Oracle**: estado parcial predefinido
- **RGB images**: observación visual pura (sin acceso al estado interno)

## Algoritmos evaluados y resultados

| Algoritmo | Tipo | Observación |
|-----------|------|-------------|
| CEM | Model-free, basado en población | Dynamics Oracle |
| SAC | Model-free, actor-critic | Estado / imagen |
| CURL-SAC | SAC + representación contrastiva | Imagen |
| DrQ | SAC + data augmentation | Imagen |
| PlaNet | Model-based, recurrente | Imagen |

**Resultado clave:** CEM con dynamics oracle supera al resto en la mayoría de tareas.
Los métodos con acceso a estado (oracle) superan a métodos basados en imagen incluso
con técnicas modernas de data augmentation.

**Implicación para el proyecto:** Usar state oracle durante entrenamiento inicial
(acceso directo a posición de partículas en Genesis) antes de migrar a observaciones
de imagen/depth.

## Limitaciones del framework

- Físicamente basado en FleX/PBD — sin yarn-level ni propiedades anisotrópicas.
- Parámetros de tela no publicados en detalle (difícil calibrar a denim real).
- No incluye robots con muchos grados de libertad (usa pick-and-place simplificado).
- Plataforma obsoleta (Ubuntu 16.04 / CUDA 9.2) — difícil de reproducir hoy.
- Sim2real validado solo con Sawyer + gripper Weiss (no UR10e).

## Relevancia para Genesis

Genesis adopta PBD (misma familia que FleX), por lo que la física de tela en SoftGym
es el antecedente más cercano disponible. Sin embargo, los parámetros de FleX no
son directamente portables a Genesis — requieren recalibración usando [[Unphased_Wrinkles_2022]].

## Relacionado
- [[PBD_Cloth_Solver]] — mismo método físico (PBD) que SoftGym/FleX
- [[Gym_Wrapper_Genesis]] — nuestro equivalente de SoftGym para Genesis
- [[Unfolding_Cloth_Review_2024]] — cita SoftGym como benchmark estándar del campo
- [[Unphased_Wrinkles_2022]] — calibración de parámetros PBD para denim real
- [[Genesis_Simulator]] — plataforma objetivo de nuestro proyecto
