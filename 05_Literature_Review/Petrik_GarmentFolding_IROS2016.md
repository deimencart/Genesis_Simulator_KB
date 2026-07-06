---
category: [literature, simulation]
status: draft
created: 2026-07-06
updated: 2026-07-06
tags: [paper, cloth-model, beam-theory, folding-path, physics-based, quasi-static]
---

# Physics-Based Model of a Rectangular Garment for Robotic Folding (IROS 2016)

IEEE Xplore: https://ieeexplore.ieee.org/document/7759164
PDF (acceso directo, requiere red CTU): https://cmp.felk.cvut.cz/ftp/articles/petrik/PetrikIROS2016.pdf
Autores: Vladimír Petrík, Vladimír Smutný, Pavel Krsek, Václav Hlaváč (CTU Prague — CMP)
Venue: IEEE/RSJ IROS 2016, Daejeon

> **Nota de ingest:** el PDF original fue ilegible (binario comprimido). El contenido
> de esta nota se construyó a partir de la búsqueda del paper en IEEE Xplore, de la
> descripción del modelo en el paper de seguimiento (arXiv 1904.01298) y de la extensión
> de estabilidad estática (arXiv 1902.11021) por los mismos autores.

## Por qué importa para el proyecto

Introduce el modelo físico más citado para diseño de trayectorias de doblado robótico
de prendas rectangulares. Contrasta directamente con el enfoque PBD de [[PBD_Cloth_Solver]]:
en lugar de simular partículas, resuelve la deformación con teoría de vigas elásticas.
El parámetro central (relación peso/rigidez) sirve de puente conceptual con los valores
de [[Unphased_Wrinkles_2022]] para calibrar nuestro entorno.

## Modelo físico: Euler-Bernoulli beam + BVP

El paper modela la prenda como una **viga elástica de Euler-Bernoulli**:

$$EI \frac{d^4 w}{dx^4} = q(x)$$

donde:
- **E** = módulo de Young del material (Pa)
- **I** = segundo momento de área de la sección transversal (m⁴)
- **K = E·I** = rigidez a la flexión (bending stiffness)
- **q(x)** = carga distribuida (peso por unidad de longitud)

La deformación se resuelve como un **problema de valor en la frontera (BVP)**, no
mediante integración temporal. El resultado es la forma de equilibrio estático de la tela
bajo gravedad y la restricción del agarre del robot.

## Supuestos del modelo

| Supuesto | Detalle |
|----------|---------|
| Geometría rectangular | La prenda es rectangular y homogénea |
| Material homogéneo | Densidad y rigidez constantes en toda la pieza |
| Un solo parámetro | **Relación peso/rigidez** (ηᵦ = ρg/K) resume el comportamiento |
| Cuasi-estático | Se calcula equilibrio de fuerzas, no dinámica |
| Fold line fija | Alineada con un borde de la prenda rectangular |
| Single-arm | Un único brazo con gripper |

## Parámetro clave: relación peso/rigidez

$$\eta_b = \frac{\rho g}{K} = \frac{\rho g}{EI}$$

- A mayor ηᵦ → prenda más flexible (cae más bajo su propio peso).
- A menor ηᵦ → prenda más rígida (denim, cuero).
- Este parámetro unifica la física del modelo en un escalar que puede estimarse
  experimentalmente, conectando con los valores de E en [[Unphased_Wrinkles_2022]].

## Algoritmo de trayectoria de doblado

1. Definir la línea de doblez (fold line) alineada con un borde.
2. Resolver la forma de equilibrio de cada segmento de la tela bajo gravedad.
3. Calcular la trayectoria del end-effector que mantiene la tela en equilibrio estático
   durante todo el movimiento (evita deslizamiento y colapso).

La trayectoria resultante es determinista dado ηᵦ — no requiere RL ni demostraciones.

## Limitaciones (documentadas en papers de seguimiento)

1. **No modela fricción interna** entre hilos (urdimbre/trama). Para mezclilla (sarga 3×1)
   esto es significativo — la fricción direccional afecta el doblado.
2. **Rigidez lineal**: no captura comportamiento no lineal a grandes deformaciones.
3. **Sin inestabilidad estática (snap-through)**: no detecta el colapso repentino de la
   tela cuando el doblez supera un ángulo crítico. Resuelto en arXiv 1902.11021.
4. **Requiere ηᵦ conocido a priori**: si el material es desconocido, el modelo falla.
   Abordado con control por retroalimentación en arXiv 1904.01298.
5. **Solo prendas rectangulares**: no generaliza a formas irregulares (camisas, pantalones).

## Contraste con PBD (relevante para Genesis)

| Aspecto | Euler-Bernoulli (este paper) | PBD ([[PBD_Cloth_Solver]]) |
|---------|------------------------------|---------------------------|
| Formulación | Ecuación diferencial continua | Simulación de partículas discreta |
| Parámetro central | K = E·I (bending stiffness física) | `bending_compliance` (adimensional) |
| Tipo de solución | BVP estático | Integración temporal |
| Velocidad | Muy rápida (analítica) | Más lenta (iterativa) |
| Realismo dinámico | No (solo equilibrio) | Sí (permite dinámica) |
| Calibración | E medible con instrumentos | Requiere calibración empírica |

**Implicación:** Para obtener ηᵦ de denim y usarlo como referencia de calibración en PBD,
partir de E_denim ≈ 0.35-0.45 GPa (ver [[Unphased_Wrinkles_2022]]) y la geometría de la
prenda para calcular I.

## Relevancia para el proyecto

- El **algoritmo de trayectoria** puede usarse como *baseline* o como generador de
  demostraciones para imitation learning, dado que es determinista y analítico.
- La **relación ηᵦ** es un indicador de dificultad: denim tiene ηᵦ bajo (rígido),
  lo que implica trayectorias de doblado más estables pero mayor fuerza requerida.
- La **limitación de no modelar fricción** aplica igualmente a [[PBD_Cloth_Solver]]
  (sin fricción anisotrópica en Genesis por defecto).

## Papers de seguimiento del mismo grupo

| Paper | Año | Aportación |
|-------|-----|-----------|
| Single arm robotic garment folding path generation | 2017 (Advanced Robotics) | Extensión con workspace más amplio |
| Static Stability of Robotic Fabric Strip Folding (arXiv 1902.11021) | 2020 | Detección de snap-through e inestabilidad estática |
| Feedback-based Fabric Strip Folding (arXiv 1904.01298) | 2020 | Control por feedback sin conocer ηᵦ a priori |

## Relacionado
- [[PBD_Cloth_Solver]] — enfoque alternativo (partículas) vs. beam theory
- [[Unphased_Wrinkles_2022]] — valores de E para denim, necesarios para calcular K y ηᵦ
- [[GarmentLab_NeurIPS2024]] — benchmark que incluye Fold/Unfold con modelos más complejos
- [[SoftGym_CoRL2020]] — benchmark que aborda FoldCloth con RL en lugar de path planning
