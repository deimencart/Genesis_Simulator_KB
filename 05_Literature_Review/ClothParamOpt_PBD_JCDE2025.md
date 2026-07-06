---
category: [literature, simulation]
status: draft
created: 2026-07-06
updated: 2026-07-06
tags: [paper, real-to-sim, pbd, cloth-parameters, parameter-optimization, bayesian, ur10]
---

# Real-to-Sim High-Resolution Cloth Modeling: Physical Parameter Optimization Using Particle-Based Simulation (JCDE 2025)

DOI: https://doi.org/10.1093/jcde/qwaf065
Journal: Journal of Computational Design and Engineering, Vol. 12 No. 8, pp. 29-44, agosto 2025
Autores: Kang-il Yoon, Soo-Chul Lim (Dongguk University, Seoul)

## Por qué importa para el proyecto

**El único paper de los ingresados que optimiza parámetros reales sobre un simulador
particle-based (PBD-like, Taichi).** A diferencia de [[RealToSim_DynamicFabric_2025]]
que usa FEM (DiffSim) o MPM (DiffTaichi), este paper trabaja con resortes y amortiguadores
entre partículas — la misma arquitectura que [[PBD_Cloth_Solver]] en Genesis.

Los cuatro parámetros optimizados (stretch, bending, damping, friction) corresponden
directamente a los knobs de [[PBD_Cloth_Config_Denim]]. El robot usado es **UR10**
(mismo brazo que [[UR10e_2F85_Genesis]]). El método es reproducible con Genesis.

## Simulador backend: Taichi mass-spring

**Framework:** Taichi (lenguaje de alto rendimiento para GPU)
**Tipo:** mass-spring particle system con 3 familias de restricciones (no MPM)
**Malla:** 80 × 80 nodos = **6,400 vértices**
**Velocidad:** ~282 FPS en RTX 4090 (dt = 5×10⁻⁴ s, Explicit Euler)

> Distinción importante: **Taichi ≠ DiffTaichi**. DiffTaichi usa MPM (Material
> Point Method). Este paper usa Taichi como framework de cómputo para un modelo
> mass-spring, que es la misma familia que PBD de Genesis.

### Tres familias de springs (análogo a Genesis PBD)

| Restricción | Color en paper | Equivalente Genesis |
|-------------|---------------|---------------------|
| Structural | Azul | `stretch_compliance` |
| Shear | Verde | `stretch_compliance` (diagonal) |
| Bending | Roja | `bending_compliance` |

## Cuatro parámetros optimizados (por unidad de masa)

| Parámetro | Símbolo | Rango de búsqueda | Fenómeno |
|-----------|---------|-------------------|----------|
| Stretching stiffness | k'_s | [1, 1×10³] | Resistencia tensión / sag gravitacional |
| Bending stiffness | k'_b | [1, 5×10⁴] | Curvatura, arrugas, stiffness flexional |
| Damping | c' | [1, 3×10³] | Supresión vibraciones post-movimiento |
| Friction | μ | [1, 5×10²] | Deslizamiento en superficie |

**Nota de normalización:** k' = k_raw / m. Relación con Genesis: `compliance ∝ 1/k'`,
por lo que tela rígida (k'_b alto) → `bending_compliance` bajo en Genesis.

**Acoplamiento empírico:** Friction es el parámetro más crítico globalmente (+7.2%
error sin él) y está acoplado con bending — optimizar sin friction hace colapsar la
solución de flexión.

## Pipeline de optimización

```
Datos reales (UR10 + 2× RealSense L515)
        ↓  point clouds 3D
Pérdida: L = CD(V_sim, P_real) + λ·HD(V_sim, P_real)
        ↓  unidireccional sim→real (robusto a oclusiones)
[Etapa 1] Bayesian Optimization (50 iters, GP surrogate, 10 random init)
        → mejor punto inicial
[Etapa 2] Gradient Descent (50 epochs de refinamiento local)
        → parámetros optimizados {k'_s, k'_b, c', μ}
```

**Por qué CD + HD:** CD captura similitud global de forma; HD captura errores locales
(arrugas). Sin HD el método pierde detalle en Drape (+1.5% CD).

## Robot y hardware

| Componente | Especificación |
|------------|----------------|
| Arm | Universal Robots **UR10** (mismo que UR10e en el proyecto) |
| Gripper | OnRobot RG2-FT (force-torque) |
| Fuerza agarre | 0.5–4.5 N (adaptativo por material) |
| Velocidad TCP | 28.9 mm/s media, 41.1 mm/s máx — **quasi-estático** |
| Cámaras | 2× Intel RealSense L515 (RGB-D) |
| Muestras | 240 × 240 mm (calibración), 180 × 180 mm (test escala) |

## 9 materiales evaluados (sin denim explícito)

| Tejido | Espesor (mm) | Densidad (GSM) | Más parecido a |
|--------|-------------|---------------|----------------|
| Cloth4 | 1.22 | 229 | **Denim (más cercano)** |
| Cloth9 | 0.65 | 217 | Sarga media-pesada |
| Cloth1 | 0.56 | 155 | Algodón pesado |
| Cloth3 | 0.16 | 50 | Seda/voile |

Rango: 49–229 GSM. Denim real (~300–450 GSM) está fuera del rango pero Cloth4 es
el mejor proxy disponible — sus parámetros optimizados pueden usarse como punto de
partida para calibrar [[PBD_Cloth_Config_Denim]].

## Resultados numéricos (Chamfer Distance promedio)

| Método | Tasks vistas | Drape (no visto) | Escala no vista |
|--------|-------------|-----------------|-----------------|
| **Este paper** | **0.059** | **0.151** | **0.055** |
| DIFFCLOUD (baseline) | 0.246 | 0.268 | 0.274 |
| Sin bending | 0.061 (+3.1%) | 0.163 | — |
| Sin friction | 0.063 (+7.2%) | 0.157 | — |

**Mejora sobre DIFFCLOUD: ≈4.8× en CD**. Generaliza bien a escala no vista
(0.055 ≈ 0.059) — evidencia de que los parámetros capturan propiedades intrínsecas.

## Comparación directa con [[RealToSim_DynamicFabric_2025]]

| Aspecto | ClothParamOpt_PBD (este) | RealToSim_DynamicFabric |
|---------|--------------------------|------------------------|
| Backend | Taichi mass-spring (PBD) | DiffSim (FEM) / DiffTaichi (MPM) |
| Parámetros | k'_s, k'_b, c', μ | C₁₁–C₃₃ ó E, ν |
| Aplicabilidad a Genesis | **Alta** (misma arquitectura) | Baja (requiere traducción FEM→PBD) |
| Escenarios | Quasi-estático (v<41 mm/s) | Cuasi + **dinámico (fling, shaking)** |
| Validación dinámica | **No** | Sí |
| Mejor CD en folding | 0.049 | PINN: 0.015 (escala diferente) |

**Síntesis:** Este paper resuelve la brecha PBD que [[RealToSim_DynamicFabric_2025]]
dejaba abierta, pero hereda su limitación: sin validación dinámica. Para nuestro
proyecto: usar este método para calibrar k'_s, k'_b, c', μ en quasi-estático, y
luego validar en dinámico siguiendo los escenarios de shaking/fling del otro paper.

## Implicaciones directas para [[PBD_Cloth_Config_Denim]]

1. **Punto de partida de calibración:** usar Cloth4 (229 GSM, 1.22mm) como proxy de
   denim para los rangos [k'_s, k'_b, c', μ] del paper.
2. **Pipeline reproducible con Genesis:** sustituir el backend Taichi por Genesis PBD
   + misma pérdida CD+HD + mismo BO→GD. Posible contribución técnica de la tesis.
3. **Parámetro crítico:** optimizar friction (μ) primero — mayor impacto y acoplado con bending.
4. **Resolución:** 80×80 partículas es más denso que configuraciones por defecto de
   Genesis; verificar que el solver PBD escala bien a esa densidad.

## Limitaciones reconocidas

- Solo quasi-estático (v < 41 mm/s) — validar en dinámico es trabajo futuro.
- 9 materiales; denim (>300 GSM) fuera del rango evaluado.
- Parámetros globales (homogéneos) — no modela heterogeneidad espacial.
- Sin oclusión severa — CD unidireccional mitiga pero no elimina el problema.

## Relacionado
- [[PBD_Cloth_Config_Denim]] — calibración directamente mejorable con este método
- [[PBD_Cloth_Solver]] — mismo tipo de simulador (particle-based, spring constraints)
- [[RealToSim_DynamicFabric_2025]] — complementario: FEM/MPM + validación dinámica
- [[Unphased_Wrinkles_2022]] — valores FEM de referencia para denim (E, μ, b)
- [[UR10e_2F85_Genesis]] — mismo brazo (UR10) usado en los experimentos
