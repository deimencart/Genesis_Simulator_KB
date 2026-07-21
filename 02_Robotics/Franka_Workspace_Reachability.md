---
category: [robotics, simulation]
status: validated
created: 2026-07-09
updated: 2026-07-09
tags: [franka, inverse-kinematics, workspace, reachability, sampling]
---

# Franka — Mapeo de Alcanzabilidad del Workspace (IK Sampling)

Experimento por muestreo para estimar qué región del espacio 3D alrededor del
Franka es realmente alcanzable por el efector final, usando
`inverse_kinematics` en vez de asumir un volumen de trabajo teórico. Scripts
reales: `~/genesis_ws/phase_0_franka/4_1_reachable_franka.py` (generación +
clasificación, versión final) y `4_2_plotting_space.py` (visualización).
Cifras de esta nota verificadas cargando con numpy los `.npy` reales
producidos por esos scripts (`reachable_points_2.npy`,
`reachable_flags_2.npy`), no estimadas.

## Método (versión final validada)

1. Sobre-muestrear `N_oversample=500` puntos candidatos en una caja
   `x∈[-0.8,0.8]`, `y∈[-0.8,0.8]`, `z∈[0.0,1.0]` (metros, uniforme).
2. Filtrar por distancia radial en el plano XY: excluir candidatos con
   `radio_xy ≤ 0.15` (m) — zona de exclusión cilíndrica alrededor del eje de
   la base — **antes** de recortar al tamaño final.
3. Recortar a `N_target=200` puntos.
4. Por cada candidato: `robot.inverse_kinematics(link=hand, pos=candidato,
   quat=orientación_fija, return_error=True)`.
5. Clasificar **alcanzable** si `norm(error[:3]) < 0.01` m (los primeros 3 de
   los 6 componentes del vector de error son el error de posición).
6. Guardar en `reachable_points_2.npy` (200×3) y `reachable_flags_2.npy`
   (200,).
7. Visualizar en una segunda escena sin física: una esfera verde/roja por
   punto.

Código real completo: [[Franka_Reachability_Sampling]] en `06_Code_Snippets/`.

## Iteración previa con bug de sesgo direccional (ya corregido)

La primera versión, `4_ws_franka.py`, muestreaba `x_range=(0.2, 0.8)` — **solo
valores positivos de X**, cubriendo únicamente el cuadrante frente al robot en
vez de todo su alrededor — y no tenía zona de exclusión cilíndrica ni
sobre-muestreo (`N=200` generados directamente). Produjo
`reachable_points.npy`/`reachable_flags.npy` (200 puntos, 134 alcanzables,
67%; verificado cargando ese `.npy`: `x` real observado en `[0.205, 0.799]`).
La versión corregida (`4_1_reachable_franka.py`) usa rangos simétricos en X e
Y, añade la exclusión cilíndrica y el sobre-muestreo. Identificado como
versión final por fecha de modificación (16:56 vs. 16:36 del original) y
porque es la que produce los `.npy` con sufijo `_2` que carga
`4_2_plotting_space.py`.

## Resultados verificados (versión final, N=200)

- **Alcanzables: 113 / 200 (56.5%)**.
- Rango real de candidatos generados: `x∈[-0.795, 0.797]`,
  `y∈[-0.774, 0.785]`, `z∈[0.010, 0.999]`.
- Radio XY de todos los candidatos: `[0.153, 1.119]` — el mínimo (0.153)
  confirma que el filtro de exclusión (0.15) se aplicó correctamente.
- Entre los puntos **alcanzables**: radio 3D `[0.216, 0.963]` m, radio XY
  `[0.168, 0.806]` m.

## Hallazgo 1 — límite máximo de alcance claro; límite mínimo NO verificado más allá del diseño

Agrupando por radio 3D en bins de 0.1m, la fracción de puntos alcanzables es
~100% entre 0.2–0.8m, cae a 87% en [0.8–0.9), a 26% en [0.9–1.0), y a 0% desde
1.0m en adelante — un límite **máximo** de alcance real y bien definido, en la
zona de transición ~0.85–1.0m.

**El límite mínimo no está verificado por este experimento.** La exclusión de
radio 0.15m fue una decisión de diseño tomada *antes* de generar candidatos
(comentario en el código: *"zona de exclusión cilíndrica alrededor de la
base"*), no un límite descubierto por fallos de IK. Como nunca se generaron
candidatos dentro de ese cilindro, los datos no permiten confirmar ni refutar
si existe un verdadero radio mínimo de alcance más cercano a la base — de
hecho, el bin más cercano al límite de exclusión (radio 3D 0.2–0.3m) tiene
100% de alcanzabilidad (aunque con solo 4 puntos, muestra chica). Corrección
respecto a una versión anterior de esta nota: no hay evidencia en los datos de
auto-colisión o límite articular como causa de un "radio mínimo" — esa
hipótesis no fue probada por este experimento tal como está diseñado.

## Hallazgo 2 — la exclusión cilíndrica debe aplicarse antes de recortar al tamaño final

En el código, el orden es: generar `N_oversample` candidatos → calcular
distancia radial XY → filtrar por máscara → **recién ahí** recortar a
`N_target`. Aplicar el filtro después de recortar (o no sobre-muestrear)
arriesga terminar con menos de `N_target` puntos útiles, o con puntos dentro
del volumen físico del robot donde IK nunca converge (ver iteración previa,
que no tenía este filtro en absoluto).

## Hallazgo 3 — sobre-muestrear y filtrar en vez de definir rangos asimétricos a mano

`4_1_reachable_franka.py` genera 500 candidatos en una caja simple de rangos
simétricos y filtra por distancia radial, en vez de intentar adivinar a mano
qué rangos por eje aproximan la forma real del workspace (que, por el
Hallazgo 1, no se conoce de antemano).

## Hallazgo 4 — limitación metodológica: orientación fija

Los 200 puntos se evaluaron con **una sola orientación**, capturada una vez
con `end_effector_link.get_quat()` inmediatamente después de `scene.build()`
(la pose neutral del efector, sin mover el robot). El resultado es el
workspace **geométrico puro** (dónde puede llegar el origen del link `hand`),
no necesariamente el workspace **útil para manipulación**, que depende de con
qué orientación llega el gripper (p. ej. apuntando hacia abajo).

## Hallazgo 5 — errores encontrados y resueltos durante el desarrollo

- **Sesgo de cámara de un solo punto de vista** — una sola posición de cámara
  no permite inspeccionar todos los ángulos del workspace muestreado a la vez.
- **Sesgo direccional del rango de muestreo** — `x_range=(0.2, 0.8)` en la
  primera versión solo cubría el semiespacio +X (ver "Iteración previa"
  arriba; verificado directamente en `4_ws_franka.py` y en el `.npy`
  resultante).
- **Las entidades deben crearse antes de `scene.build()`** — confirmado en el
  propio código de `4_2_plotting_space.py` (comentario: *"crear una esfera
  por cada punto, ANTES de scene.build()"*); los 200 marcadores se agregan en
  un loop previo a `scene.build()`, no dinámicamente después.

## Documentación exhaustiva (fuera del vault)

El código completo, la explicación línea por línea, y el proceso de
depuración de este experimento (incluyendo el bug de sesgo direccional de la
primera versión) están documentados en `~/genesis_ws/tutor_notes/` — capa de
aprendizaje exhaustiva, separada del vault, sin límite de tamaño. Esta nota es
la versión concisa para el vault, siguiendo `SCHEMA.md`.

## Pendiente

- [ ] Repetir el mapeo con orientación fija de manipulación (gripper hacia abajo) en vez de orientación neutral, para acotar el workspace *útil*.
- [ ] Verificar si existe un radio mínimo real de alcance, reduciendo o
  eliminando la exclusión cilíndrica de diseño y observando si IK empieza a
  fallar de verdad cerca de la base (el Hallazgo 1 dejó esto sin probar).
- [ ] Repetir este mismo método de mapeo para el **UR10e** en la Fase 3 del
  ROADMAP — el slippage de agarre documentado en
  [[UR10e_Gripper_Cloth_Issues]] podría estar relacionado, parcial o
  totalmente, con pedirle al gripper posiciones cerca del borde de su
  workspace real, en vez de ser puramente un problema de fricción.

## Relacionado

- [[Genesis_Simulation_Interface]] — patrón de IK (`inverse_kinematics`)
  usado como base de este experimento.
- [[UR10e_Gripper_Cloth_Issues]] — candidato a repetir este mapeo para
  descartar/confirmar workspace como causa contribuyente del slippage.
- [[Franka_Reachability_Sampling]] — código fuente real de este experimento.
