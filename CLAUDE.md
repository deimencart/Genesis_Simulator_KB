# CLAUDE.md — Instrucciones para este vault

Este directorio es un **wiki de conocimiento vivo**, no un chat ni un RAG. Antes de hacer
cualquier cosa (ingestar una fuente, responder una pregunta, o revisar el vault), lee:

1. `00_Core_Schema/SCHEMA.md` — reglas de estructura, atomicidad, frontmatter y flujos.
2. `index.md` — catálogo de todas las páginas existentes (léelo ANTES de crear una nota
   nueva, para no duplicar y para saber a qué enlazar).
3. `ROADMAP.md` — estado actual del proyecto, fase activa y tareas pendientes por fase.
4. `log.md` — historial cronológico de ingests/queries/lints (solo las últimas 5-10
   entradas suelen bastar de contexto).

## Reglas duras (no negociables)

- **Nunca leas todo el vault de un jalón.** Usa `index.md` para decidir qué 2-5 archivos
  abrir según la pregunta. Esa es la razón de ser de este sistema: gastar pocos tokens.
- **Nunca escribas en `raw/`.** Esa carpeta son fuentes originales inmutables (PDFs,
  papers, transcripts). Solo se lee de ahí.
- **Cada nota nueva o modificada actualiza `index.md` y agrega una línea a `log.md`.**
  Sin excepción, aunque sea un cambio menor.
- **Nota atómica = un concepto, una entidad, un paper, o un snippet.** Si una nota supera
  ~150-200 líneas o mezcla 2+ temas, se divide y se enlazan entre sí con `[[wikilinks]]`.
- Sigue las convenciones de código y relaciones obligatorias descritas en `SCHEMA.md`.

## Flujo típico de sesión

```
cd ~/vault
claude
> Lee index.md y SCHEMA.md. Voy a ingestar [fuente]. Sigue el workflow de ingest.
```
