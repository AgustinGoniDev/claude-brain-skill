---
description: Opera el wiki interno de {{PROJECT_NAME}}. Subcomandos: ingest (default), query, lint.
argument-hint: "[ingest | query <pregunta> | lint]"
---

Sos el operador del wiki de {{PROJECT_NAME}}. El wiki es un knowledge base markdown vivo
en `{{WIKI_DIR}}/`. **Antes de hacer nada, leé `{{WIKI_DIR}}/CLAUDE.md` completo** — ese archivo
es tu manual operativo con todas las reglas de frontmatter, wikilinks, y operaciones.

---

## Detectar sub-operación

Parseá `$ARGUMENTS`:
- Vacío o `ingest` → modo **INGEST**.
- Empieza con `query` → modo **QUERY** con el resto como pregunta.
- `lint` → modo **LINT**.
- Cualquier otra cosa → pedir aclaración al usuario y abortar.

---

## Modo INGEST

Objetivo: crear (o actualizar) un nodo de sesión en `{{WIKI_DIR}}/sessions/` que capture
lo que pasó en ESTA conversación.

1. Leé `{{WIKI_DIR}}/CLAUDE.md` y `{{WIKI_DIR}}/index.md`.
2. Revisá la conversación actual (todos los turnos previos). Identificá:
   - Área(s) tocadas.
   - Verbo de acción principal (definimos, migramos, decidimos, implementamos, debuggeamos).
   - Output concreto: archivos creados/modificados, URLs.
   - Decisiones importantes con rationale.
   - Pendientes explícitos.
3. Generá el slug: `YYYY-MM-DD-<kebab-case-corto>`. Fecha = hoy. El slug describe el
   RESULTADO (no el proceso), máx. 6 palabras.
4. ¿Existe ya un nodo con esa fecha y tema muy similar?
   - **Sí** → UPDATE: leelo y appendeá a {{SECTION_DECISIONS}}/{{SECTION_OUTPUT}}/{{SECTION_PENDING}} bajo un sub-bloque
     `### Actualización [HH:MM]`. No reescribas lo anterior.
   - **No** → CREATE.
5. CREATE: generá frontmatter completo siguiendo las reglas de `{{WIKI_DIR}}/CLAUDE.md`.
   - `related`: buscá en `{{WIKI_DIR}}/index.md` nodos con tags solapados, misma área o
     proximidad cronológica. Máx. 5.
   - `sources`: IDs de fuentes consultados (de `{{WIKI_DIR}}/sources.md`), paths del repo tocados
     con prefijo `repo:`, URLs externas con prefijo `url:`.
6. Escribí el cuerpo con las 6 secciones obligatorias. En {{SECTION_CROSS_REFS}} usá `[[slug]]` con razón en una línea.
7. **Bidireccionalidad obligatoria**: para cada nodo en `related`, abrilo y agregá un
   bullet recíproco en su sección `## {{SECTION_CROSS_REFS}}`: `[[este-nodo]] — razón`. Sin excepciones.
8. Actualizá `{{WIKI_DIR}}/index.md`: insertá el bullet `[[slug]] — one-liner` al tope de la
   sección del área correspondiente. Si el área no existe, creala en orden alfabético.
9. Append a `{{WIKI_DIR}}/log.md`:
   ```
   ## [YYYY-MM-DD HH:MM] ingest | <slug>
   - Área: <area>
   - Cross-refs: <lista de slugs relacionados o "ninguna">
   ```
10. Reportá al usuario: path del nodo creado/actualizado, cross-refs agregadas, y si
    hubo alguna decisión ambigua.

{{LEGACY_INGEST_NOTE}}

---

## Modo QUERY

Objetivo: responder una pregunta usando el wiki como base, con citations verificables.

1. Leé `{{WIKI_DIR}}/CLAUDE.md` y `{{WIKI_DIR}}/index.md` completos.
2. De los one-liners del índice, seleccioná 1-5 nodos candidatos para la pregunta.
   Si ninguno parece relevante, decilo y sugerí ampliar la búsqueda.
3. Leé esos nodos completos.
4. {{BACKEND_QUERY_STEP}}
5. Sintetizá respuesta en 2-5 párrafos. Citations usando wikilinks:
   `[[slug-del-nodo]]`.
   **NO uses markdown links** `[texto](ruta)` para nodos del wiki.
6. Si detectás un gap (cross-ref obvio faltante, concepto en 3+ nodos sin nodo propio),
   NO lo arreglés — reportalo al final como "Sugerencia para `/{{COMMAND_NAME}} lint`".
7. Si no hay info suficiente en el wiki, decilo explícitamente. **No inventes** ni
   extrapoles más allá de lo que dicen los nodos.

---

## Modo LINT

Objetivo: reportar el estado de salud del wiki **SIN modificar archivos**.

1. Leé `{{WIKI_DIR}}/CLAUDE.md`, `{{WIKI_DIR}}/index.md` y **todos** los archivos en `{{WIKI_DIR}}/sessions/`.
2. Revisá cada categoría según las reglas de lint de `{{WIKI_DIR}}/CLAUDE.md`.
3. Devolvé un reporte markdown estructurado:

   ```markdown
   # Lint report — YYYY-MM-DD HH:MM

   ## Resumen
   - Nodos totales: N
   - Issues críticos: N (broken wikilinks, frontmatter inválido, índice desincronizado)
   - Issues medios: N (orphans, missing cross-refs)
   - Oportunidades: N (conceptos emergentes, stale claims)

   ## Críticos
   ### Broken wikilinks
   - `[[slug-inexistente]]` en `2026-04-XX-nodo.md` → sugerir corrección

   ## Medios
   ### Orphan nodes
   - `[[slug]]` — sin inbound links. Sugerencia: cross-ref desde [[nodo-relacionado]].

   ## Oportunidades
   ### Conceptos emergentes
   - "término" aparece en 3 nodos. Candidato a `type: concept`.
   ```

4. **NO modificar archivos.**
5. Append a `{{WIKI_DIR}}/log.md`:
   ```
   ## [YYYY-MM-DD HH:MM] lint | report
   - Críticos: N | Medios: N | Oportunidades: N
   ```
