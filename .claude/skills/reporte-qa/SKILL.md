---
name: reporte-qa
description: >-
  Arma el reporte de una corrida de QA con formato fijo, lo publica como página
  de Confluence y refleja el veredicto en la card de Trello. Usar cada vez que
  el agente `qa` termina de correr pytest.
---

Lo que se repite en toda corrida del `qa`: el mismo reporte, el mismo lugar de
publicación, las mismas transiciones de tablero. El juicio es del agente; esto
es el molde.

## 1. El reporte

Plantilla exacta. Las secciones van siempre, aunque queden vacías (una sección
vacía es información: "ningún criterio sin cubrir").

```markdown
# QA — <feature o incidente> — <fecha ISO>

**Veredicto: PASS | FAIL | FAIL (esperado)**

## Resumen
N tests · N pasan · N fallan · Ns

## Cobertura por criterio
| Criterio (del spec) | Test | Estado |
|---|---|---|
| <criterio 1> | `test_<nombre>` | cubierto |
| <criterio 2> | — | SIN CUBRIR — <por qué> |

## Fallos
### `test_<nombre>`
- **Entrada:** <request / args exactos>
- **Esperado:** <valor>
- **Obtenido:** <valor real, copiado de la salida de pytest>
- **Hipótesis:** <causa> (`archivo:línea`)

## Comando
`pytest -q` desde `<dir>` — salida completa adjunta.
```

Reglas del reporte:

- **Obtenido** se copia de la salida real de pytest. Si no corriste pytest, no
  hay reporte.
- Un bloque de `Fallos` por test rojo. Sin agrupar "varios fallos de límites".
- La hipótesis de causa apunta a `archivo:línea`. Es una hipótesis, no un
  diagnóstico: la causa raíz la firma el `investigador`.

## 2. Publicar en Confluence

1. Espacio y padre salen de las variables del repo: `CONFLUENCE_SPACE` y
   `CONFLUENCE_PARENT_PAGE_ID`. No los hardcodees.
2. Título: `QA — <feature> — <fecha ISO>` (ej. `QA — n1-vendedores — 2026-09-11`).
3. `confluence_create_page(space=..., parent_id=CONFLUENCE_PARENT_PAGE_ID,
   title=..., body=<el markdown de arriba>)`.
4. Guardá la URL: va en el comentario de Trello y en la salida del turno.

El reporte también queda en el repo. Confluence es copia publicada, no fuente de
verdad. **Si Confluence no responde** (sin red en CI, token vencido): no falla el
pipeline — dejá el md en el repo, anotá "Confluence no disponible — link
pendiente" y seguí.

## 3. Reflejar en Trello

Los nombres de columna salen de `TRELLO_BOARD_ID` + `get_lists` (nombre → id),
nunca hardcodeados — el board del grupo puede llamarlas distinto.

| Momento | Movimiento |
|---|---|
| el `qa` empieza | `In Progress` → `QA` |
| veredicto `PASS` | `QA` → `Done` |
| veredicto `FAIL` | `QA` → `In Progress` + etiqueta `bloqueado` + comentario con los fallos |
| veredicto `FAIL (esperado)` — incidente | **se queda en `QA`**. No movés, no etiquetás `bloqueado` |

El comentario usa el formato de la skill `registrar-en-card`:

```
[qa] FAIL — 7 tests, 2 rojos
· test_listar_items_no_hace_n_mas_1: esperado ≤2 queries, obtenido 6
reporte: https://<confluence>/QA-n1-vendedores
```

Reglas duras:

- Nunca movés a `Done` con un test rojo que no sea el rojo esperado de un
  incidente.
- Nunca movés un incidente fuera de `QA`: espera el gate humano
  (`aprobado-para-fix` / `descartado`). Ver skill `mover-card`.
- Si Trello no responde: log local y seguís. La observabilidad no bloquea.

Reporte completo de ejemplo, con los tres veredictos: [`ejemplos.md`](ejemplos.md).
