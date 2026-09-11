# Escribir el agente de QA (y su skill)

En este proyecto el agente `qa` **no viene hecho**. Lo escribís vos. Es el mismo
movimiento de la Fase A.7 de la Clase 2 (instrucción repetida → `SKILL.md`),
ahora aplicado a un agente entero + su skill.

Referencia de dónde partir: [`agentes/qa.referencia.md`](agentes/qa.referencia.md)
(la de la Clase 2, adaptada). **No la entregues tal cual.**

## 1. El agente — `.claude/agents/qa.md`

Frontmatter:
- `name: qa`, `description` (cuándo delegar en él).
- `tools`: `Read, Write, Edit, Bash, Grep, Glob` + las MCP que necesita para
  Trello (`mcp__trello__move_card`, `add_comment`, `get_lists`,
  `set_active_board`) y Confluence (`mcp__atlassian__confluence_create_page`).
  **Nada más.** Sin `Task`, sin tools de escritura sobre servicios.
- `skills: reporte-qa`.

Cuerpo (system prompt), en este orden:
1. **Rol** en una frase: "encontrás dónde el código no cumple; no lo arreglás".
2. **Método**: leer spec/criterios → derivar casos (los explícitos + los
   implícitos) → adversarial (cero, negativos, ±1, tipos, orden) → escribir
   `test_*.py` → correr `pytest -q` y leer la salida real.
3. **Restricción dura**: solo edita `test_*.py`. Si toca implementación, falló
   la tarea.
4. **Formato de salida** fijo: resumen (N tests, N pasan/fallan), cobertura por
   criterio, fallos (entrada / esperado vs obtenido / hipótesis de causa),
   veredicto `PASS`/`FAIL`.

## 2. La skill — `.claude/skills/reporte-qa/SKILL.md`

Es lo que se repite en cada corrida del `qa`. Metelo en la skill, no en el
system prompt.

`SKILL.md` (corto, con progressive disclosure — el ejemplo largo va en un
archivo aparte):

- **Formato del reporte** (la plantilla exacta).
- **Publicar a Confluence**: crear una página bajo
  `CONFLUENCE_PARENT_PAGE_ID` (variable del repo) con título
  `QA — <feature> — <fecha>`; devolver el link.
- **Actualizar Trello** — qué transición en qué momento:

  | Momento | Movimiento |
  |---|---|
  | el `qa` empieza | `In Progress` → `QA` |
  | veredicto PASS | `QA` → `Done` |
  | veredicto FAIL | `QA` → `In Progress` + etiqueta `bloqueado` + comentario con los fallos |

  Los nombres de columna salen de la variable del repo, no hardcodeados.

## 3. Lo que dejás en el repo

- `.claude/agents/qa.md` (el tuyo)
- `.claude/skills/reporte-qa/SKILL.md` (+ el archivo de ejemplo largo)

## 4. Cómo se integra al CI

No tocás YAML. `claude-review.yml` ya viene con el prompt de orquestación hecho
y, en el paso de QA, invoca al subagente `qa` por nombre — el que vos escribís.
Con solo tener `.claude/agents/qa.md` en el repo, el pipeline lo usa.
