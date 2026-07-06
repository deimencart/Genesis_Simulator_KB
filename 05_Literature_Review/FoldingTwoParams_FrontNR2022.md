---
category: [literature, simulation]
status: draft
created: 2026-07-06
updated: 2026-07-06
tags: [paper, cloth-folding, trajectory, c-ipc, fem, friction, contact, sim-only]
---

# Effective Cloth Folding Trajectories in Simulation with Only Two Parameters (Frontiers in Neurorobotics, 2022)

DOI: https://doi.org/10.3389/fnbot.2022.989702
PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC9493483/
Autores: Victor-Louis De Gusseme, Francis Wyffels (IDLab-AIRO, Ghent University – imec)
Journal: Frontiers in Neurorobotics, Vol. 16:989702, septiembre 2022

## Por qué importa para el proyecto

Identifica el **espacio mínimo de parámetros de trayectoria** para folding robótico —
solo dos (height ratio y tilt angle). El argumento central es que el **modelo
constitutivo importa menos que el manejo de contacto y fricción**: el folding falla
principalmente por deslizamiento, no por imprecisión del modelo elástico. Esto refuerza
directamente el hallazgo de [[ClothParamOpt_PBD_JCDE2025]] donde friction es el
parámetro de material más crítico (+7.2% error sin él).

Además, proporciona el blueprint de una trayectoria parametrizable con Bézier cuadrática
que puede usarse con el [[UR10e_2F85_Genesis]] sin necesidad de búsqueda exhaustiva.

## Simulador backend: C-IPC (FEM)

**Codimensional Incremental Potential Contact (C-IPC)** — extensión de IPC a objetos
codimensionales (tela 2D en espacio 3D).
- Tipo: **FEM** (no PBD) — modelo constitutivo continuo, contacto intersection-free
- Fricción: estática y cinética con conmutación rápida entre modos
- Malla: mínimo 20,000 triángulos/m² (~12,000 para camisa manga larga)
- Coste: **~14 min/simulación** en CPU (sin GPU en el paper)
- Sin GPU: el paper afirma ser el primero en aplicar C-IPC a manipulación robótica

Contraste directo con [[ClothParamOpt_PBD_JCDE2025]]:

| Aspecto | Este paper | ClothParamOpt_PBD |
|---------|-----------|-------------------|
| Simulador | C-IPC (FEM) | Taichi (mass-spring PBD) |
| Parámetros reducidos | 2 de **trayectoria** | 4 de **material** |
| Velocidad sim. | ~14 min/sim | ~282 FPS |
| Validación real | No (solo simulación) | Sí (UR10) |
| Aplicabilidad a Genesis | Baja (FEM ≠ PBD) | Alta (misma familia) |

## Los dos parámetros (Bézier cuadrática)

La trayectoria del gripper se define como una **curva de Bézier cuadrática** con
posición fija de inicio/fin y un único punto de control libre, reducida a:

| Parámetro | Definición | Rango | Unidad |
|-----------|-----------|-------|--------|
| **Height ratio** (h) | Altura del pico como fracción de la mitad de la distancia inicio-fin | 0.1 – 1.0 | adimensional |
| **Tilt angle** (θ) | Rotación de la curva alrededor de la línea que conecta inicio con fin | 0° – 60° | grados |

**Espacio de búsqueda total:** 134 trayectorias en una región "en forma de abanico".

**Zona óptima para doblado de manga:** height ratio 0.4–0.8, tilt 15°–45°.

**Por qué solo dos son suficientes:**
- Height ratio controla tensión y clearance durante el movimiento.
- Tilt angle adapta la dirección a la geometría del doblez.
- Juntos codifican la física esencial sin sobre-parametrizar.
- La simplicidad hace las trayectorias más robustas para sim-to-real.

## El argumento central: contacto > constitutiva

> "Arguably even more important for cloth folding is not the constitutive model, but
> how contact and friction are handled. Folding is the act of stacking layers of cloth
> on top of each other, and thus naturally introduces many contacts. Many folding
> failure modes can be attributed to contact and friction."

**Experimento de fricción:**

| μ (fricción) | Resultado |
|-------------|-----------|
| 0.2 (baja) | Fallo concentrado en banda estrecha; deslizamiento domina |
| 0.5 (default) | Región de éxito amplia |
| 0.8 (alta) | Mínimo cambio vs. default; comportamiento robusto |

**Conclusión:** Folding con fricción baja es significativamente más difícil. La fricción
determina si la tela se desliza o queda colocada en la posición deseada — importa más
que la precisión del modelo elástico.

**Convergencia con [[ClothParamOpt_PBD_JCDE2025]]:** ese paper muestra que friction
es el parámetro de **material** más crítico en PBD (+7.2%); este paper muestra que
friction es el factor físico más crítico para el **éxito del folding** en FEM.
Dos enfoques distintos, misma conclusión.

## Escenarios evaluados

**Prenda:** camisa manga larga (modelo paramétrico, 12 parámetros de forma).

**Secuencia de 5 pasos de doblado:**
1. Doblar manga izquierda → dentro.
2. Doblar manga derecha → dentro.
3. Doblar lado izquierdo del cuerpo.
4. Doblar lado derecho del cuerpo.
5. Doblar por la mitad sobre sí misma.

**Variaciones evaluadas:** 3 formas de camisa, 3 espesores de material (×0.2, ×1, ×5),
2 niveles de fricción (0.2, 0.8), 3 velocidades (1s, 4s, 8s de duración).

**Velocidad crítica:** a 1s (rápido), "la trayectoria cambia significativamente — el
gripper se mueve tan rápido que la tela se desplaza demasiado por inercia." Esto
conecta con [[RealToSim_DynamicFabric_2025]]: los movimientos dinámicos son cualitativamente
distintos de los quasi-estáticos y requieren parámetros/trayectorias diferenciadas.

## Métrica de evaluación

**Mean Distance Loss:**
$$L = \frac{1}{n} \sum_{i=1}^n \|v^*_i - v_i\|^2$$

Umbrales cualitativos:
- < 2 mm → pliegue casi perfecto
- ~5 mm → pliegue bueno con arrugas menores
- > 20 mm → fallo significativo

**Sin validación en robot real.** El paper propone como trabajo futuro transferir las
trayectorias simuladas a un robot real usando lookup table o nearest-neighbor sobre
el dataset de simulaciones precomputadas.

## Relación con el proyecto (UR10e + denim)

1. **Trayectoria de referencia:** La parametrización Bézier (h, θ) es directamente
   usable como búsqueda de política en Genesis para el [[UR10e_2F85_Genesis]].
   134 trayectorias × 5 pasos = dataset de referencia en ~400h CPU (o menos con GPU).

2. **Prioridad de friction:** Calibrar μ en [[PBD_Cloth_Config_Denim]] **antes** que
   k'_s o k'_b. Este paper lo confirma independientemente de [[ClothParamOpt_PBD_JCDE2025]].

3. **Velocidad del gripper:** Mantener TCP < ~30 mm/s para que el folding sea
   quasi-estático y los parámetros de [[PBD_Cloth_Config_Denim]] sean válidos.

4. **Denim vs algodón:** Material ×5 más grueso (analogía para denim rígido) muestra
   "higher losses across all trajectories" — folding de denim requerirá trayectorias
   con mayor height ratio para compensar la mayor rigidez.

## Limitaciones reconocidas

- Solo simulación (C-IPC), sin validación real.
- Solo algodón-like; denim, lana, poliéster no probados directamente.
- Prenda de una sola capa (no doble capa cosida como prendas reales).
- C-IPC lento (~14 min/sim); sin GPU acceleration en el paper.
- No desacopla deslizamiento de arrugado en la métrica de pérdida.

## Relacionado
- [[ClothParamOpt_PBD_JCDE2025]] — converge en criticidad de friction; contraste PBD vs FEM
- [[PBD_Cloth_Config_Denim]] — friction debe calibrarse primero según ambos papers
- [[Petrik_GarmentFolding_IROS2016]] — también diseña trayectorias de folding, con beam theory vs Bézier
- [[RealToSim_DynamicFabric_2025]] — velocidad rápida (1s) cambia la dinámica; mismo hallazgo
- [[GarmentLab_NeurIPS2024]] — incluye Fold como tarea; contexto del benchmark
- [[PBD_Cloth_Solver]] — contraste FEM (C-IPC) vs PBD en manejo de contacto
