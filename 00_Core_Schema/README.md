# Genesis Simulator — Knowledge Base

Wiki de conocimiento vivo sobre el simulador **Genesis**, aplicado al proyecto de tesis:
manipulación de tela tipo mezclilla con un brazo **UR10e + gripper 2F-85**, usando el
solver **PBD** para cloth y un posible entorno de entrenamiento tipo **Gymnasium**.

Este vault no es un chat ni un buscador — es un wiki que se construye y mantiene de forma
incremental. Cada fuente nueva (paper, doc, issue de GitHub) se lee una vez, se extrae lo
importante, y se integra como notas atómicas enlazadas entre sí. La síntesis nunca se
rehace desde cero; se actualiza.

## Cómo está organizado

- `raw/` — fuentes originales, inmutables. Nunca se editan, solo se leen.
- `00_Core_Schema/` — este README + `SCHEMA.md` (reglas de estructura, frontmatter,
  convenciones, flujos de trabajo). Léelo antes de crear cualquier nota.
- `01_Physics_Engines/` — Genesis, solvers (PBD, FEM, MPM) y motores comparables.
- `02_Robotics/` — robots, grippers, cinemática, control.
- `03_AI_Training/` — RL, políticas, entornos gym, algoritmos.
- `04_Integration_ROS/` — integración con ROS2, issues de compatibilidad.
- `05_Literature_Review/` — una nota por cada paper leído.
- `06_Code_Snippets/` — fragmentos de código reutilizables y ya validados.
- `index.md` (raíz) — catálogo de todas las páginas del wiki.
- `log.md` (raíz) — historial cronológico de ingests, queries y lints.

## Regla de oro

Una nota = un concepto. Nada de páginas gigantes que mezclen varios temas — eso es lo
que hace que abrir el vault gaste tokens de más. Las notas "hub" (como
`Genesis_Simulator.md`) solo enlazan a las notas atómicas, no repiten su contenido.

## Flujo básico

1. Se agrega una fuente nueva a `raw/`.
2. Se lee `index.md` para ver qué ya existe.
3. Se crean/actualizan notas atómicas siguiendo `SCHEMA.md`.
4. Se actualiza `index.md` y se agrega una línea a `log.md`.

Ver `SCHEMA.md` para las reglas completas de frontmatter, wikilinks obligatorios y
workflow de replicación de papers.