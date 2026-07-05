# SCHEMA — Genesis Simulator Wiki

## Principio de atomicidad (CLAVE para bajo consumo de tokens)

- **Una nota = una idea.** Un solver, un paper, un robot, un experimento, un snippet.
  Nunca "Genesis general" con todo adentro.
- **Límite blando: ~150-200 líneas por nota.** Si crece más, se parte y se enlaza.
- Las notas "hub" (ej. `Genesis_Simulator.md`) son cortas y solo **enlazan** a las notas
  atómicas; no repiten su contenido. Un hub es un índice temático, no un resumen largo.
- Regla práctica: si para responder una pregunta necesitarías citar más de 3 secciones
  distintas de una misma nota, esa nota debería ser 3 notas.

## Estructura de carpetas

```
raw/                    # Fuentes originales, inmutables (PDFs, artículos, transcripts)
00_Core_Schema/          # README + SCHEMA (este archivo)
01_Physics_Engines/      # Genesis, solvers (PBD, FEM, MPM), motores comparables
02_Robotics/             # Robots, grippers, cinemática, control
03_AI_Training/          # RL, políticas, entornos gym, algoritmos
04_Integration_ROS/      # Integración con ROS2, issues de compatibilidad
05_Literature_Review/    # Una nota por paper leído
06_Code_Snippets/        # Fragmentos de código reutilizables, validados
index.md                 # Catálogo de todas las páginas (raíz)
log.md                    # Historial cronológico append-only (raíz)
```

## Convenciones de código

- Bloques de código siempre con el formato: ` ```python (genesis_env) `
- Si el snippet ya fue probado y funciona, márcalo con `status: validated` en frontmatter.
  Si es propuesto pero no probado, `status: draft`.

## Frontmatter obligatorio

```yaml
---
category: [simulation, training, ros, robotics]  # una o más
status: draft | validated | issue
created: YYYY-MM-DD
updated: YYYY-MM-DD
source: "[[nombre-de-la-nota-en-05_Literature_Review]]"   # si aplica
tags: []
---
```

- Si el contenido trata sobre cinemática o dinámica, incluye siempre las fórmulas en LaTeX.
- Si hay un error de integración con ROS, etiqueta la nota como `status: issue` hasta que
  se resuelva, y ponla en `04_Integration_ROS/`.

## Relaciones obligatorias (Wiki Links)

- Toda nota de **"Policy"** (política de RL) debe enlazar a su **Robot** de origen y a su
  **Paper** de origen mediante `[[wikilinks]]`.
- Toda nota de **paper** (`05_Literature_Review/`) debe enlazar a los conceptos que usa
  (ej. `[[PBD_Cloth_Solver]]`) y, si reporta parámetros físicos usados en tu proyecto,
  a la nota de configuración correspondiente en `06_Code_Snippets/`.
- Toda nota de **solver/motor físico** debe enlazar a los papers que la respaldan y a los
  snippets de código que lo configuran.

## Workflow de ingest (fuente nueva)

1. Copia la fuente cruda a `raw/` (nunca la edites ahí).
2. Lee `index.md` para ver qué ya existe y evitar duplicar notas.
3. Extrae: 1) entidades nuevas (robot, solver, paper), 2) parámetros/datos concretos,
   3) contradicciones con notas existentes.
4. Crea o actualiza notas atómicas (una por concepto tocado — normalmente 3-8 notas).
5. Actualiza `index.md` (agrega fila) y `log.md` (agrega línea con formato de abajo).
6. Si la fuente contradice algo existente, no borres lo viejo: agrega una sección
   `## Contradicción / actualización (YYYY-MM-DD)` en la nota afectada.

## Workflow de replicación de papers

1. Definir arquitectura del paper.
2. Identificar el "Physics Core" necesario en Genesis.
3. Crear el script de entrenamiento.
4. Documentar brechas entre el paper y tu implementación local (nota con
   `status: issue` si hay brecha sin resolver).

## Formato de log.md

```
## [YYYY-MM-DD] ingest | Título de la fuente
- Notas creadas: [[...]], [[...]]
- Notas actualizadas: [[...]]
- Contradicciones encontradas: ninguna / descripción breve
```

## Formato de index.md

Tabla por carpeta, con: nombre de nota, una línea de resumen, status, fecha.
