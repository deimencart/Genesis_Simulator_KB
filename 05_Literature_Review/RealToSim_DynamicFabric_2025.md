---
category: [literature, simulation]
status: draft
created: 2026-07-06
updated: 2026-07-06
tags: [paper, real-to-sim, cloth-parameters, dynamic, pbd-calibration, fem, mpm, pinn]
---

# Can Real-to-Sim Approaches Capture Dynamic Fabric Behavior for Robotic Fabric Manipulation? (2025)

arXiv: https://arxiv.org/abs/2503.16310
Autores: Yingdong Ru, Lipeng Zhuang, Zhuo He, Florent P. Audonnet, Gerardo Aragon-Caramasa
Afiliación: School of Computing Science, University of Glasgow
Venue: preprint arXiv, 2025

## Por qué importa para el proyecto

Evaluación rigurosa de cuatro métodos de estimación de parámetros de tela
(real-to-sim). La conclusión clave es que **los parámetros calibrados en escenarios
quasi-estáticos no generalizan a movimientos dinámicos**. Esto impacta directamente
la validez de [[PBD_Cloth_Config_Denim]]: una calibración basada en poses estáticas
puede no predecir bien shaking o fling con el [[UR10e_2F85_Genesis]].

Además, **ningún método usa PBD como backend** — todos usan FEM (DiffSim) o MPM
(DiffTaichi). Hay una brecha sin resolver entre los parámetros que estos métodos
estiman (C₁₁, C₁₂, C₂₂, C₃₃ ó E, ν) y los parámetros de Genesis
(`stretch_compliance`, `bending_compliance`).

## Métodos comparados

| Método | Backend simulador | Parámetros estimados | Estrategia |
|--------|------------------|----------------------|-----------|
| **DIFFCLOUD** | DiffSim (FEM) | C₁₁, C₁₂, C₂₂, C₃₃ (anisotrópico) | Optim. diferenciable, Chamfer loss |
| **DiffCP** | DiffTaichi (MPM) | E, ν (isotrópico) | Optim. diferenciable, Chamfer loss |
| **PhysNet** | DiffSim / DiffTaichi | Ídem al backend | Red Siamese + optim. Bayesiana |
| **PINN** | DiffSim / DiffTaichi | C₁₁–C₃₃ ó E, ν | MLP que satisface PDE elastodinámica |

**DiffSim** = FEM anisotrópico, malla de 81 vértices, 60 timesteps.
**DiffTaichi** = MPM (Material Point Method), mismo nº de vértices; maneja grandes
deformaciones pero usa modelo isotrópico — inadecuado para telas reales.

## Escenarios: calibración → evaluación en dinámicos ocultos

**Calibración (vistos):**
- *Lifting*: gripper levanta esquina izquierda desde mesa — quasi-estático.
- *Wind blowing*: ventilador empuja tela suspendida — semi-dinámico.
- *Stretching*: extensión tensil predefinida — usado solo por PINN, quasi-estático.

**Evaluación (ocultos, no vistos durante calibración):**
- *Folding*: esquina doblada hacia el lado opuesto — quasi-estático.
- *Fling*: dos grippers aceleran rápidamente — **altamente dinámico**.
- *Shaking*: un gripper oscila verticalmente — **dinámico**.

Robot: Rethink Robotics Baxter (bimanual). Sensores: 2× ZED2i RGB-D a 15 Hz + SAM2.

## Resultados numéricos (Chamfer Distance promedio, 5 telas)

| Método | Folding ↓ | Fling ↓ | Shaking ↓ |
|--------|-----------|---------|-----------|
| **DIFFCLOUD** | 0.115 | **0.669** | **0.215** |
| DiffCP | 0.116 | **0.561** | 1.375 |
| PhysNet-DiffSim | 0.029 | 0.805 | 0.194 |
| **PINN-DiffSim** | **0.015** | 1.023 | 0.360 |
| DiffTaichi (todos) | 0.291–1.513 | — | — |

Nota: DiffTaichi rinde mal en folding (CD hasta 1.513 vs. ~0.015 de DiffSim)
por el modelo isotrópico — confirma que E, ν son insuficientes para telas reales.

## Conclusión principal: parámetros quasi-estáticos ≠ dinámicos

**PINN gana en folding** (quasi-estático, CD = 0.015) pero es el peor en fling
(CD = 1.023) — calibración en stretching no generaliza a dinámica.

**DIFFCLOUD gana en dinámicos** (shaking CD = 0.215, fling CD = 0.669) — los
parámetros anisotrópicos FEM capturan mejor el comportamiento bajo aceleración.

**No existe un método universal**: la elección depende de si la tarea es
quasi-estática o dinámica. Para un sistema que hace ambas (desdoblar → sacudir),
se necesitaría calibración separada o un método más robusto.

## Comparación con [[Unphased_Wrinkles_2022]]

| Aspecto | Unphased_Wrinkles_2022 | Este paper |
|---------|----------------------|-----------|
| Backend | Shell FEM, pérdida en frecuencia | DiffSim (FEM) / DiffTaichi (MPM) |
| Parámetros | E_u, E_v, μ, b (4 parámetros) | C₁₁, C₁₂, C₂₂, C₃₃ ó E, ν |
| Escenario calibración | Drapeado quasi-estático | Lifting / wind / stretching |
| Validación dinámica | No | Sí (fling, shaking) |
| Resultado para denim | E alto, alta rigidez flexión | No reporta denim explícito |

**Conclusión combinada:** Unphased_Wrinkles_2022 da los mejores parámetros de
referencia para denim en quasi-estático; este paper advierte que esos parámetros
pueden no predecir bien movimientos dinámicos.

## Implicaciones para PBD_Cloth_Config_Denim y el proyecto

1. **Brecha PBD sin resolver:** todos los métodos comparados usan FEM o MPM como
   backend — ninguno estima `stretch_compliance` / `bending_compliance` de PBD
   directamente. La traducción de C₁₁–C₃₃ a parámetros Genesis requiere calibración
   adicional (ver [[PBD_Cloth_Config_Denim]]).

2. **Validar en dinámico obligatorio:** si la tarea de tesis incluye sacudir o
   lanzar la tela antes de doblar, la calibración de [[PBD_Cloth_Config_Denim]]
   debe verificarse en fling/shaking, no solo en poses estáticas.

3. **DIFFCLOUD como referencia de ground truth:** para estimar los 4 parámetros
   anisotrópicos (C₁₁, C₁₂, C₂₂, C₃₃) de la mezclilla con mayor robustez dinámica.

4. **DiffTaichi (MPM) descartable para denim:** el modelo isotrópico (E, ν) produce
   CD hasta 10× mayor que FEM anisotrópico en telas rígidas — no usar para calibración.

## Relacionado
- [[PBD_Cloth_Config_Denim]] — calibración actual; necesita validación dinámica según este paper
- [[PBD_Cloth_Solver]] — ningún método del paper usa PBD; brecha sin resolver
- [[Unphased_Wrinkles_2022]] — parámetros quasi-estáticos de denim; complementario a este paper
- [[Petrik_GarmentFolding_IROS2016]] — también usa modelo quasi-estático; misma advertencia aplica
- [[GarmentLab_NeurIPS2024]] — incluye fling y shaking como tareas; mismo contexto dinámico
