---
name: claude-brain
description: >
  Instala un LLM Wiki (knowledge base markdown interconectado, estilo Carpati/Obsidian)
  en cualquier proyecto. Genera la carpeta del wiki con schema operativo, índice, bitácora
  y registro de fuentes, más el slash command (ingest | query | lint) y la configuración
  de Foam para VSCode. Usá este skill cuando el usuario diga "instalá el cerebro",
  "quiero un wiki LLM", "configurá el sistema de memoria", "armá el brain", "montá el
  knowledge base", o cuando pida un sistema de conocimiento acumulativo mantenido por Claude.
---

# claude-brain

Instalador interactivo de un LLM Wiki para cualquier proyecto. Hace preguntas,
genera toda la estructura adaptada al contexto del usuario, e instala la configuración
necesaria para que Claude pueda mantener el wiki con `/brain ingest`, `/brain query`
y `/brain lint`.

El sistema está basado en el patrón Carpati: un knowledge base de markdown interconectado
con wikilinks Foam, nodos de sesión con frontmatter YAML, índice denso para el LLM,
bitácora append-only y registro de fuentes externas.

---

## Fase 1: Recolección de contexto

**Antes de crear ningún archivo**, hacé las siguientes preguntas al usuario. No asumas respuestas — necesitás todas para generar correctamente.

Podés hacer las preguntas en un solo bloque conversacional (no es necesario esperar respuesta una por una salvo que una respuesta cambie drásticamente lo que necesitás preguntar después).

### Preguntas requeridas

**1. Nombre del wiki (directorio)**
> "¿Cómo querés llamar al directorio del wiki? (default: `brain`, alternativas: `cerebro`, `wiki`, `knowledge`, `memoria`)"

**2. Nombre del slash command**
> "¿Cómo querés llamar al slash command? (default: igual al nombre del wiki)"
> Si el usuario elige `cerebro` como nombre del wiki, sugerí `memoria` como comando — es el patrón que usamos en AustralisAI.

**3. Nombre del proyecto**
> "¿Cómo se llama tu proyecto u organización? (se usa en la descripción del comando)"

**4. Idioma del contenido de los nodos**
> "¿En qué idioma vas a escribir el contenido del wiki?"
> Opciones: Español / English / Otro (que especifique)

Según el idioma, las secciones de cada nodo se llaman diferente:
- Español: Contexto, Decisiones, Output, Pendiente, Cross-refs, Fuentes
- English: Context, Decisions, Output, Pending, Cross-refs, Sources
- Otro: el usuario define los nombres

**5. Áreas del proyecto**
> "¿Cuáles son las áreas de tu proyecto? Listá las que necesitás (ej: marketing, ops, dev, research, product)."
> Estas se convierten en el enum `area` del frontmatter y en secciones del índice.

**6. Backend de fuente de verdad**
> "¿Qué herramienta usás como fuente de verdad del conocimiento del proyecto?"
> - **A — Notion**: el wiki guarda punteros con IDs cortos; Claude hace fetch vía MCP cuando necesita contenido fresco
> - **B — Archivos locales**: la fuente de verdad son archivos en el mismo repo (docs, specs, etc.)
> - **C — Otra herramienta**: especificá cuál (Confluence, Linear, Airtable, etc.) y qué instrucciones debería seguir Claude
> - **D — Ninguno**: wiki standalone, sin fuente externa

**7. ¿Tenés sesiones históricas para migrar?**
> "¿Hay archivos de sesiones anteriores (logs, notas, changelogs) que querés migrar al wiki?"
> - Sí → generamos una nota de inmutabilidad en el schema y podés migrar manualmente después
> - No → el wiki arranca limpio

---

## Fase 2: Generación de archivos

Con todas las respuestas, generá los siguientes archivos usando los templates de `references/`.

### Variables a resolver antes de generar

Definí internamente estas variables según las respuestas:

```
WIKI_DIR        = respuesta Q1 (ej: "brain")
COMMAND_NAME    = respuesta Q2 (ej: "brain" o "memoria")
PROJECT_NAME    = respuesta Q3 (ej: "Acme Corp")
AREA_ENUM       = areas separadas por " | " (ej: "marketing | ops | dev")
SECTION_CONTEXT     = "Contexto" | "Context" | custom
SECTION_DECISIONS   = "Decisiones" | "Decisions" | custom
SECTION_OUTPUT      = "Output" (igual en todos los idiomas)
SECTION_PENDING     = "Pendiente" | "Pending" | custom
SECTION_CROSS_REFS  = "Cross-refs" (igual en todos los idiomas)
SECTION_SOURCES     = "Fuentes" | "Sources" | custom
BACKEND_TYPE    = A | B | C | D
HAS_LEGACY      = true | false
```

### Archivos a crear

#### 1. `<WIKI_DIR>/CLAUDE.md`

Tomá el template de `references/brain-claude-md.md` y reemplazá todos los placeholders:

- `{{WIKI_DIR}}` → WIKI_DIR
- `{{COMMAND_NAME}}` → COMMAND_NAME
- `{{AREA_ENUM}}` → AREA_ENUM
- `{{SECTION_*}}` → nombres de secciones según idioma
- `{{SOURCE_PREFIX}}` → según backend:
  - A (Notion): `notion`
  - B (archivos): `repo`
  - C (otro): el prefijo que tenga más sentido para la herramienta
  - D (ninguno): `repo`
- `{{BACKEND_QUERY_RULE}}` → según backend:
  - A: "Si algún nodo cita una fuente Notion (vía `sources.md`) Y la pregunta depende del contenido fresco, resolvé vía `mcp__notion__notion-fetch` usando la URL de `sources.md`. **NO respondas desde el resumen cache de `sources.md`** — ése es sólo un directorio."
  - B: "Si la pregunta depende del contenido actual de un archivo del repo, leelo directamente con Read. El wiki guarda punteros, no copias."
  - C: "Si la pregunta depende de contenido fresco de [nombre herramienta], consultala directamente. El wiki guarda punteros, no copias del contenido."
  - D: "Respondé exclusivamente desde los nodos del wiki. Si el contenido no está en el wiki, decilo explícitamente."
- `{{BACKEND_SECTION_TITLE}}` → según backend:
  - A: "Integración con Notion"
  - B: "Integración con archivos del repo"
  - C: "Integración con [nombre herramienta]"
  - D: *(omitir esta sección completa)*
- `{{BACKEND_SECTION_BODY}}` → según backend:
  - A: "Notion es la fuente de verdad del proyecto. Los nodos referencian páginas de Notion con IDs cortos definidos en `{{WIKI_DIR}}/sources.md`. Claude **debe** hacer `mcp__notion__notion-fetch` con la URL correspondiente cuando la query depende del contenido fresco de una página. El resumen cache en `sources.md` solo sirve para decidir si vale la pena hacer fetch."
  - B: "Los archivos del repo son la fuente de verdad. Los nodos referencian archivos con `repo:<path>` en el frontmatter. Cuando una query depende del contenido actual de un archivo, Claude lo lee directamente con Read. El wiki no duplica el contenido de los archivos — guarda punteros."
  - C: "[Instrucciones de integración provistas por el usuario]"
  - D: *(omitir)*
- `{{LEGACY_NOTE}}` → según HAS_LEGACY:
  - true: "**NUNCA** tocar los archivos de sesiones legacy marcados como 'histórico inmutable' en `{{WIKI_DIR}}/sources.md`. Son histórico congelado. Nuevas sesiones van SOLO a `{{WIKI_DIR}}/sessions/`."
  - false: *(string vacío)*
- `{{LANGUAGE_RULE}}` → según idioma:
  - Español: "- Contenido de nodos: **español**.\n- Tags y slugs: **español sin tildes, kebab-case**.\n- Frontmatter keys: **inglés** (convención técnica)."
  - English: "- Node content: **English**.\n- Tags and slugs: **English, kebab-case**.\n- Frontmatter keys: **English**."
  - Otro: "- Contenido de nodos: **[idioma elegido]**.\n- Tags y slugs: kebab-case.\n- Frontmatter keys: **inglés** (convención técnica)."

#### 2. `<WIKI_DIR>/index.md`

Generá directamente (no hay template separado):

```markdown
# <WIKI_DIR> — Index

> Node catalog. One one-liner per node. Organized by type and then by area.
> To operate this wiki, read `CLAUDE.md` in this directory.
> When querying, read this file first to decide which nodes to open.

**Last updated:** <FECHA DE HOY>
**Total nodes:** 0 sessions | 0 concepts | 0 ADRs

---

## Sessions

<una sección H3 por cada área en AREA_ENUM, en orden alfabético>
### <área-1>

*(empty)*

### <área-2>

*(empty)*

[... una por cada área ...]

---

## Concepts

*(empty — will emerge organically via `/<COMMAND_NAME> lint` when a term appears in 3+ nodes)*

## ADRs

*(empty)*
```

Si el idioma es español, los headers van en español: "Sesiones", "Conceptos".

#### 3. `<WIKI_DIR>/log.md`

```markdown
# <WIKI_DIR> — Operations Log

> Append-only. Most recent entries at the **end**.
> Format: `## [YYYY-MM-DD HH:MM] <operation> | <slug-or-target>`
> Valid operations: `bootstrap`, `ingest`, `update`, `rename`, `merge`, `split`, `lint`, `deprecate`.

---

## [<FECHA HOY> <HORA ACTUAL>] bootstrap | wiki-initialized
- Created: CLAUDE.md, index.md, log.md, sources.md, sessions/
- Areas: <AREA_ENUM>
- Backend: <BACKEND_TYPE description>
- Generated by: skill claude-brain v1
```

#### 4. `<WIKI_DIR>/sources.md`

Usá el template correspondiente de `references/sources-templates.md`:
- Backend A (Notion) → TEMPLATE A
- Backend B (archivos) → TEMPLATE B
- Backend C (otro) → TEMPLATE D (completando `{{CUSTOM_BACKEND_NAME}}` e instrucciones)
- Backend D (ninguno) → TEMPLATE C

Reemplazá `{{WIKI_DIR}}` con WIKI_DIR.

#### 5. `<WIKI_DIR>/sessions/.gitkeep`

Archivo vacío para que git trackee el directorio vacío:
```
```
*(archivo completamente vacío)*

#### 6. `.claude/commands/<COMMAND_NAME>.md`

Tomá el template de `references/brain-command.md` y reemplazá:
- `{{WIKI_DIR}}` → WIKI_DIR
- `{{COMMAND_NAME}}` → COMMAND_NAME
- `{{PROJECT_NAME}}` → PROJECT_NAME
- `{{SECTION_DECISIONS}}`, `{{SECTION_OUTPUT}}`, `{{SECTION_PENDING}}`, `{{SECTION_CROSS_REFS}}` → nombres de secciones según idioma
- `{{BACKEND_QUERY_STEP}}` → misma regla que `{{BACKEND_QUERY_RULE}}` en CLAUDE.md
- `{{LEGACY_INGEST_NOTE}}` → según HAS_LEGACY:
  - true: "**NUNCA** toques los archivos de sesiones legacy marcados como inmutables. Son histórico congelado."
  - false: *(string vacío)*

#### 7. `.vscode/settings.json`

Este paso requiere lógica de merge:

**Si `.vscode/settings.json` NO existe:**
```json
{
  "foam.files.ignore": [
    "**/.claude/**",
    "**/node_modules/**"
  ]
}
```
(Si HAS_LEGACY=true, agregar también los paths de las carpetas legacy que el usuario indicó.)

**Si `.vscode/settings.json` YA existe:**
1. Leer el archivo actual con Read
2. Verificar si `foam.files.ignore` existe en el JSON
   - Si existe: agregar los nuevos paths al array existente (sin pisar los que ya están)
   - Si no existe: agregar el key `foam.files.ignore` con los paths nuevos
3. Usar Edit para hacer el cambio quirúrgico (NO sobreescribir el archivo completo)

Paths a agregar al array:
- `**/.claude/**`
- `**/node_modules/**`
- Si HAS_LEGACY=true, por cada carpeta de sesiones legacy que el usuario mencione: `**/<carpeta>/sessions/**`

---

## Fase 3: Resumen de setup

Después de crear todos los archivos, mostrá este resumen al usuario:

```
## claude-brain instalado

Wiki en: <WIKI_DIR>/
Comando: /<COMMAND_NAME> (ingest | query | lint)

### Archivos generados
<WIKI_DIR>/CLAUDE.md          ← abrí este para ver las reglas del wiki
<WIKI_DIR>/index.md           ← catálogo de nodos (vacío por ahora)
<WIKI_DIR>/log.md             ← bitácora de operaciones
<WIKI_DIR>/sources.md         ← registro de fuentes externas
<WIKI_DIR>/sessions/          ← acá van a vivir los nodos
.claude/commands/<COMMAND_NAME>.md  ← slash command instalado
.vscode/settings.json         ← configuración Foam (creado o actualizado)

### Próximos pasos
1. Instalar la extensión Foam en VSCode (si no la tenés):
   Marketplace: "Foam" de Foam team, o buscar foam-vscode
<SI BACKEND=A>
2. Completar brain/sources.md con las URLs de tus páginas de Notion
3. Asegurate de tener el MCP de Notion configurado en .mcp.json
</SI BACKEND=A>
<SI HAS_LEGACY=true>
2. Podés migrar sesiones históricas manualmente: copiá el contenido, agregá
   el frontmatter YAML y guardá en <WIKI_DIR>/sessions/YYYY-MM-DD-slug.md
</SI HAS_LEGACY>
4. Al final de tu primera sesión de trabajo, corré `/<COMMAND_NAME> ingest`
```

---

## Notas para el LLM

- Los templates en `references/` usan placeholders con doble llave `{{PLACEHOLDER}}`. Reemplazalos todos antes de escribir los archivos.
- No dejes ningún placeholder sin reemplazar en los archivos generados.
- Si el usuario da nombres en mayúsculas o con espacios para las áreas, convertirlos a kebab-case lowercase para el enum del frontmatter (ej: "Marketing Digital" → `marketing-digital`), pero mostrar el nombre original en el índice.
- El archivo `.vscode/settings.json` es el único que requiere lógica de merge — todos los demás se crean nuevos.
- Si `.claude/commands/` no existe, crearlo también.
- Verificar antes de crear que `<WIKI_DIR>/` no existe ya para evitar sobreescribir un wiki existente. Si existe, avisar al usuario y preguntar si quiere continuar.
