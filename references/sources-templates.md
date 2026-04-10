# Templates de sources.md por backend

Elegí el bloque correspondiente al backend seleccionado por el usuario.
Reemplazá `{{WIKI_DIR}}` con el nombre del directorio del wiki.

---

## TEMPLATE A — Notion

```markdown
# {{WIKI_DIR}} — Registro de fuentes externas

> IDs cortos para referenciar desde frontmatter (`sources:`) y cuerpos de nodos.
> Notion es **source of truth**. Cuando un nodo cite una fuente de aquí, Claude DEBE
> consultar Notion vía `mcp__notion__notion-fetch` antes de afirmar su contenido.
> El campo "Resumen cache" sólo sirve para decidir si vale la pena hacer fetch — NO es autoritativo.

---

## Notion

<!-- Agregar una entrada por cada página de Notion relevante al proyecto -->
<!-- Formato: ID corto en kebab-case, título, URL completa, resumen corto -->

### ejemplo-pagina
- **Título:** Nombre de la página
- **URL:** https://www.notion.so/<id-de-la-pagina>
- **Resumen cache:** Descripción breve del contenido (para decidir si hacer fetch).
- **Regla:** Resolver contenido real vía MCP antes de cualquier afirmación.
- **Última verificación:** YYYY-MM-DD

<!-- Duplicar el bloque anterior para cada fuente Notion -->

---

## Repo (archivos del proyecto que son referenciados como fuentes)

<!-- Archivos importantes que NO son nodos del wiki pero se citan en sesiones -->

### ejemplo-doc
- **Path:** `ruta/relativa/al/archivo.md`
- **Descripción:** Qué contiene este archivo.
```

---

## TEMPLATE B — Archivos locales

```markdown
# {{WIKI_DIR}} — Registro de fuentes externas

> Registro de archivos del repo que sirven como fuente de verdad.
> Referenciar desde frontmatter con prefijo `repo:<path>`.
> Para el contenido actual siempre leer el archivo directamente — este registro
> es sólo un directorio de qué existe y dónde.

---

## Archivos del repo

<!-- Listar archivos importantes que son fuente de verdad para las sesiones -->
<!-- Referenciar desde nodos con: sources: [repo:ruta/al/archivo.md] -->

### ejemplo-doc
- **Path:** `ruta/relativa/al/archivo.md`
- **Descripción:** Qué contiene y para qué se usa.

<!-- Agregar más entradas según sea necesario -->

---

## URLs externas

<!-- Documentación, APIs, referencias externas que no son Notion -->

### ejemplo-url
- **URL:** https://ejemplo.com/docs
- **Descripción:** Para qué se referencia.
```

---

## TEMPLATE C — Ninguno / standalone

```markdown
# {{WIKI_DIR}} — Registro de fuentes externas

> Este wiki opera sin fuentes externas configuradas.
> Para referenciar archivos del proyecto, usar prefijo `repo:<path>` en el frontmatter.
> Para URLs externas, usar prefijo `url:<url>` en el frontmatter.

---

## Archivos del repo referenciados

<!-- Agregar cuando una sesión dependa de un archivo clave del proyecto -->

---

## URLs externas

<!-- Agregar cuando una sesión dependa de una URL externa -->
```

---

## TEMPLATE D — Backend personalizado

```markdown
# {{WIKI_DIR}} — Registro de fuentes externas

> Registro de fuentes externas del proyecto.
> Backend configurado: {{CUSTOM_BACKEND_NAME}}
>
> {{CUSTOM_BACKEND_INSTRUCTIONS}}

---

## {{CUSTOM_BACKEND_NAME}}

<!-- Listar fuentes según la convención de tu herramienta -->

### ejemplo
- **ID / Path / URL:** ...
- **Descripción:** ...

---

## Archivos del repo referenciados

### ejemplo-doc
- **Path:** `ruta/relativa/al/archivo.md`
- **Descripción:** ...
```
