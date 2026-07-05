---
category: [literature]
status: draft
created: 2026-07-05
updated: 2026-07-05
tags: [paper, survey, cloth-manipulation, taxonomy, sim2real, benchmarks, rl]
---

# Unfolding the Literature: A Review of Robotic Cloth Manipulation (2024)

arXiv: https://arxiv.org/abs/2407.01361
Autores: Longhini, Wang, Garcia-Camacho, Blanco-Mulero, Moletta, Welle, Alenyà, Yin, Erickson, Held, Borràs, Kragic
Afiliaciones: KTH Royal Institute of Technology · CMU · IRI-CSIC-UPC · U. Copenhagen

## Por qué importa para el proyecto

Survey exhaustivo (2024) que mapea todo el campo de manipulación robótica de tela:
taxonomía de tareas, simuladores, representaciones de estado, benchmarks y gaps abiertos.
Base teórica para justificar el uso de [[Genesis_Simulator]] + [[PBD_Cloth_Solver]] y
el diseño del entorno [[Gym_Wrapper_Genesis]] con [[UR10e_2F85_Genesis]].

## Taxonomía de tareas

Dos ejes:

**Por contexto de aplicación:**
- Doméstico: plegado, alisado, planchado, colgado.
- Healthcare: vestimiento asistido, mantas, vendaje.
- Industria moda: clasificación, reciclaje, manufactura.

**Por tipo físico:**
- *Quasi-estático*: clasificación, alisado, plegado — dependen de forma/material.
- *Dinámico*: lanzamiento (*flinging*), movimientos con aceleración — requieren
  propiedades mecánicas (elasticidad, rigidez).

**Nuestro caso:** desdoblar (*unfolding*) mezclilla es quasi-estático con posibles
componentes dinámicos (shake inicial). El denim es un caso difícil por alta rigidez —
ver [[Unphased_Wrinkles_2022]] para parámetros mecánicos reales.

## Simuladores comparados en la literatura

| Simulador | Método físico | Observaciones |
|-----------|--------------|---------------|
| MuJoCo | Mass-spring-damper | Rápido; popular en benchmarks |
| Bullet | FEM + deformables | General purpose |
| SOFA | FEM, dominio continuo | Físicamente preciso, lento |
| FLEX (NVIDIA) | PBD | Rápido y estable; base de SoftGym |
| Isaac Sim | FEM (Omniverse) | Orientado a industria |
| SoftGym | PBD (sobre FLEX) | Benchmark RL estándar para deformables |

**Genesis no aparece** (publicado a finales de 2024, posterior al survey). PBD domina
en RL por velocidad de simulación, lo que respalda la elección de [[PBD_Cloth_Solver]]
en Genesis para nuestro proyecto.

## Representaciones de estado de la tela

- **RGB / Depth**: estándar. Depth más robusto a variaciones de iluminación/color.
- **Point clouds**: generalización superior a distintas formas y propiedades mecánicas
  (según refs. 84-85 del paper).
- **Keypoints**: compacto, pero expresividad limitada.
- **GNN sobre mesh**: alineado con la física subyacente; generaliza a distintas geometrías.

## Métodos de manipulación

| Enfoque | Ventaja | Limitación |
|---------|---------|-----------|
| Model-based (analítico) | Transferible entre tareas | Sim2real gap |
| Data-driven (GNN, PointNet++) | Flexible | Datos extensos |
| RL | Adaptable | Ineficiente en muestras |
| Imitation Learning | Directo | Requiere demostraciones |
| Heurístico | Interpretable | Asunciones rígidas |

## Gaps críticos identificados

1. **Sim2real gap:** Simuladores priorizan velocidad sobre realismo físico. Ignoran:
   humedad, temperatura, desgaste, fricción anisotrópica, simulación a nivel de yarn.
2. **Construction technique** (tafetán/sarga/punto) casi nunca abordada — directamente
   relevante para denim (trama en sarga 3×1).
3. **Fricción** subexplorada: solo 7 de ~50 métodos la consideran.
4. **Adaptación dinámica** a propiedades: la mayoría generaliza pero no adapta en tiempo real.
5. **Política multi-tarea** unificada: prácticamente inexistente.

## Benchmarks y datasets clave

- **SoftGym** (CoRL 2020): benchmark RL estándar para deformables — nota pendiente de ingest.
- **Garcia-Camacho et al.**: protocolos físicos + métricas cualitativas (varios niveles de complejidad).
- **YCB Cloth Extension**: extensión del set YCB para textiles, presentada en IROS'22, ICRA'23.
- **Framework de caracterización** (ref. 23 del paper): mide propiedades físicas/mecánicas
  para reproducibilidad y comparabilidad entre publicaciones.

## Métricas de evaluación

- Alisado/plegado: *flatness*, detección de arrugas, configuración final vs. objetivo.
- Vestimiento: tasa de éxito de colocación, fuerzas de seguridad.
- Generalización: success rate con variación de material / forma / peso / stiffness.

## Propiedad más subexplorada (tabla resumen del paper)

| Propiedad | # métodos que la abordan |
|-----------|--------------------------|
| Shape, Color | >15 |
| Size, Weight, Elasticity, Stiffness | 12-15 |
| Material | ~10 |
| Friction | 7 |
| Construction Technique | 2 |

## Relacionado
- [[PBD_Cloth_Solver]] — método dominante en RL según este survey; adoptado en Genesis
- [[Genesis_Simulator]] — no mencionado (post-2024), pero alineado con PBD del survey
- [[Unphased_Wrinkles_2022]] — parámetros mecánicos reales de denim; calibra el solver
- [[Gym_Wrapper_Genesis]] — diseño del entorno Gymnasium para este proyecto
- [[UR10e_2F85_Genesis]] — robot y gripper del proyecto
