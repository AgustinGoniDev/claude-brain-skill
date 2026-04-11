# claude-brain

Un skill de Claude Code que instala una **LLM Wiki** (base de conocimiento estilo Obsidian) en cualquier proyecto. Un solo comando y Claude configura todo: una wiki en markdown interconectada con nodos de sesión, un índice denso, un log append-only, y un slash command (`/brain`) para operaciones de ingest, query y lint.

## Qué obtenés

Después de ejecutar el skill, tu proyecto tendrá:

```
brain/                        ← directorio de la wiki (el nombre es configurable)
├── CLAUDE.md                 ← esquema operacional (reglas para Claude)
├── index.md                  ← catálogo denso de nodos (cursor del LLM)
├── log.md                    ← bitácora append-only de operaciones
├── sources.md                ← registro de fuentes externas
└── sessions/                 ← un archivo markdown por sesión
.claude/commands/brain.md     ← slash command /brain
.vscode/settings.json         ← configuración de wikilinks para Foam
```

La wiki es mantenida por Claude a través de tres operaciones:

| Comando | Qué hace |
|---------|----------|
| `/brain ingest` | Crea un nodo de sesión a partir de la conversación actual. Extrae decisiones, outputs, referencias cruzadas e ítems pendientes. Actualiza el índice y el log automáticamente. |
| `/brain query <pregunta>` | Responde usando la wiki como fuente, con citas `[[wikilink]]`. Obtiene contenido actualizado de fuentes externas cuando es necesario. |
| `/brain lint` | Chequeo de salud: wikilinks rotos, nodos huérfanos, claims desactualizados, referencias cruzadas faltantes, candidatos a nuevos conceptos. Solo lectura. |

## Requisitos previos

Antes de instalar, asegurate de tener:

- **[Claude Code](https://claude.ai/code)** — la CLI de Anthropic. Es el entorno donde se ejecutan los skills y slash commands. Sin esto, nada funciona.
- **Git** — para clonar el repositorio del skill.
- **[Foam para VSCode](https://marketplace.visualstudio.com/items?itemName=foam.foam-vscode)** — extensión opcional pero muy recomendada si querés ver el grafo visual de tu wiki (ver más abajo).
- **Notion MCP** — solo si vas a usar Notion como fuente de verdad (ver sección de configuración).

## Cómo instalar

### Paso 1 — Copiá el skill a tu proyecto

Tenés dos opciones según si querés usarlo solo en un proyecto o en todos:

```bash
# Para un proyecto específico (ejecutá esto dentro del directorio de tu proyecto)
git clone https://github.com/AgustinGoniDev/claude-brain .agents/skills/claude-brain

# Para uso global (disponible en todos tus proyectos)
git clone https://github.com/AgustinGoniDev/claude-brain ~/.claude/skills/claude-brain
```

### Paso 2 — Ejecutá el skill desde Claude Code

Abrí Claude Code en tu proyecto y ejecutá:

```
/claude-brain
```

Claude te va a hacer las preguntas de configuración (puede hacerlas todas juntas) y va a generar toda la estructura automáticamente.

> Si el comando `/claude-brain` no aparece, verificá que el skill esté en la carpeta correcta: `.agents/skills/claude-brain/` (para el proyecto) o `~/.claude/skills/claude-brain/` (global).

## Preguntas de configuración

Al ejecutar `/claude-brain`, Claude te pregunta lo siguiente:

### 1. Nombre del directorio de la wiki
Cómo llamar a la carpeta principal. Por defecto es `brain`, pero podés usar `cerebro`, `wiki`, `knowledge`, `memoria`, etc.

### 2. Nombre del slash command
El comando que vas a usar para operar la wiki. Por defecto es igual al nombre del directorio. Por ejemplo, si elegís `cerebro`, el comando quedaría `/cerebro ingest`.

### 3. Nombre del proyecto
Se usa en la descripción interna del comando. Puede ser el nombre de tu empresa, equipo o producto.

### 4. Idioma del contenido
El idioma en el que Claude va a escribir los nodos. Las opciones son Español, English u otro (en ese caso definís los nombres de las secciones vos). Esto afecta los encabezados internos de cada nodo (Contexto, Decisiones, Pendientes, etc.).

### 5. Áreas del proyecto
Las categorías de trabajo de tu equipo. Por ejemplo: `marketing, ops, dev, research, product`. Se usan como secciones en el índice y como valores del campo `area` en el frontmatter de cada nodo.

### 6. Backend de fuente de verdad
De dónde viene el conocimiento del proyecto:

- **Notion** — la wiki guarda punteros a páginas de Notion con IDs cortos. Claude hace fetch del contenido actualizado vía MCP cuando lo necesita. Requiere el MCP de Notion configurado en `.mcp.json`.
- **Archivos locales** — la fuente de verdad son archivos dentro del mismo repositorio. Claude los lee directamente con Read.
- **Otra herramienta** — Confluence, Linear, Airtable, etc. Describís cómo funciona y Claude adapta las instrucciones.
- **Ninguna** — wiki standalone, sin fuente externa. Claude responde solo desde lo que hay en los nodos.

### 7. Sesiones históricas
Si tenés logs, notas o changelogs de sesiones anteriores para migrar, Claude genera una nota de inmutabilidad en el esquema. Después podés migrar esos archivos manualmente agregándoles el frontmatter YAML correspondiente.

## Cómo funciona

Cada nodo de sesión es un archivo markdown con frontmatter YAML y seis secciones fijas:

```yaml
---
type: session
area: marketing
date: 2026-04-10
slug: email-infrastructure-setup
title: "Infraestructura de captura de emails: Brevo + n8n + Nginx"
tags: [email, lead-magnet, brevo, n8n, nginx]
status: active
related:
  - 2026-04-04-cold-email-sequence
sources:
  - notion:email-strategy
  - repo:.claude/plans/email-infra.md
superseded_by: null
---

## Contexto
## Decisiones
## Output
## Pendientes
## Cross-refs
- [[2026-04-04-cold-email-sequence]] — misma fecha, pieza complementaria
## Fuentes
- [[sources#email-strategy]] (Notion)
```

Los links internos usan la sintaxis `[[slug]]` (wikilinks). El índice (`index.md`) es el cursor principal: Claude lo lee primero en cada operación para decidir qué nodos abrir, sin leer toda la wiki de una vez.

## Visualización del grafo con Foam

Para ver el grafo visual de tu wiki, instalá la extensión **Foam** en Visual Studio Code:

1. Abrí el panel de extensiones (`Ctrl+Shift+X` en Windows/Linux, `Cmd+Shift+X` en Mac)
2. Buscá **"Foam"** y elegí la extensión de "Foam team"
3. Instalala y recargá VSCode
4. Con tu proyecto abierto, usá el comando **"Foam: Show Graph"** desde la paleta de comandos (`Ctrl+Shift+P` / `Cmd+Shift+P`)

Con Foam vas a tener:
- `[[wikilinks]]` clickeables en el editor
- Grafo visual interactivo de todos los nodos y sus conexiones
- Panel de backlinks (qué nodos apuntan al nodo actual)
- Autocompletado al escribir `[[`

El skill configura `.vscode/settings.json` automáticamente para que el grafo no incluya carpetas innecesarias como `.claude/` o sesiones legacy.

## Configuración de Notion (solo si usás ese backend)

Si elegiste Notion como fuente de verdad, necesitás:

1. Configurar el **MCP de Notion** en el archivo `.mcp.json` de tu proyecto
2. Completar `brain/sources.md` con los IDs cortos y URLs de tus páginas de Notion

Claude va a hacer fetch automático del contenido de Notion cuando lo necesite en una query.

## El patrón

Este skill implementa el **patrón LLM Wiki / Carpati**: una base de conocimiento persistente y auto-compilada donde el LLM es tanto el escritor como el lector. La clave está en que Claude lee el índice primero en cada operación, haciendo que la recuperación sea O(índice) y no O(todos-los-archivos). Las referencias cruzadas son bidireccionales y se verifican en cada ingest.

La arquitectura de tres capas:
1. **Fuentes brutas** — páginas de Notion, archivos del repo (inmutables, obtenidos frescos)
2. **Wiki** — `brain/` — nodos de sesión con frontmatter, índice denso, referencias cruzadas
3. **Esquema** — `brain/CLAUDE.md` — el manual operacional que Claude lee antes de cada operación

## Licencia

MIT
