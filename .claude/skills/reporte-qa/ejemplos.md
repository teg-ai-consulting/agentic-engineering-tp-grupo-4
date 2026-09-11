# reporte-qa — ejemplos

Tres corridas reales del pipeline, una por veredicto. Copiá la forma, no el
contenido.

---

## A. `FAIL (esperado)` — incidente del N+1

El caso del incidente de la Parte 1: el `investigador` ya firmó la causa raíz y
el `qa` escribe la regresión que la deja clavada. El rojo **es** el entregable.

```markdown
# QA — n1-vendedores — 2026-09-11

**Veredicto: FAIL (esperado)**

## Resumen
3 tests · 2 pasan · 1 falla · 0.18s

## Cobertura por criterio
| Criterio (del diagnóstico) | Test | Estado |
|---|---|---|
| `GET /v1/items` resuelve vendedor sin una query por item | `test_listar_items_no_hace_n_mas_1` | cubierto (rojo esperado) |
| el listado sigue devolviendo nombre y reputación | `test_lista_items_ok` | cubierto |
| `GET /v1/items/{id}` inexistente → 404 | `test_item_no_encontrado` | cubierto |

## Fallos
### `test_listar_items_no_hace_n_mas_1`
- **Entrada:** `GET /v1/items?limit=5`
- **Esperado:** `count() <= 2` (1 query de listado + 1 batch de vendedores)
- **Obtenido:** `6` — `AssertionError: el listado hizo 6 queries — hay un N+1`
- **Hipótesis:** una query a `vendedores` por iteración del loop del listado
  (`items-service/main.py:77`). Se arregla con un solo `WHERE id IN (...)`.

## Comando
`pytest -q` desde la raíz del repo — salida completa adjunta.
```

Trello: la card **se queda en `QA`**. No se etiqueta `bloqueado`; espera
`aprobado-para-fix` / `descartado` de un humano.

Comentario:

```
[qa] FAIL (esperado) — 3 tests, 1 rojo: la regresión del N+1 queda clavada
· test_listar_items_no_hace_n_mas_1: esperado ≤2 queries, obtenido 6
· se pone verde cuando se implemente el ADR
reporte: https://<confluence>/QA-n1-vendedores
```

---

## B. `FAIL` — pull-back de una feature

```markdown
# QA — badge-novedad — 2026-09-11

**Veredicto: FAIL**

## Resumen
7 tests · 5 pasan · 2 fallan · 0.41s

## Cobertura por criterio
| Criterio (del spec) | Test | Estado |
|---|---|---|
| item publicado hace <7d muestra badge "Nuevo" | `test_badge_presente_si_reciente` | cubierto |
| item de 7d exactos NO muestra badge | `test_badge_ausente_en_el_limite` | cubierto |
| el badge no aparece en el detalle | — | SIN CUBRIR — el spec no define el comportamiento en `GET /v1/items/{id}` |

## Fallos
### `test_badge_ausente_en_el_limite`
- **Entrada:** item con `publicado_en` = ahora − 7 días exactos
- **Esperado:** `badge is None`
- **Obtenido:** `'Nuevo'`
- **Hipótesis:** comparación `<=` donde el criterio dice "menos de 7 días"
  (`web/items.js:64`). Off-by-one en el límite.

### `test_badge_con_fecha_sin_timezone`
- **Entrada:** `publicado_en = "2026-09-06T10:00:00"` (sin `Z`)
- **Esperado:** 200 con `badge` calculado en UTC
- **Obtenido:** `500 Internal Server Error`
- **Hipótesis:** parseo asume sufijo `Z` (`web/items.js:58`).

## Comando
`pytest -q` desde `items-service/` — salida completa adjunta.
```

Trello: `QA` → `In Progress` + etiqueta `bloqueado`.

---

## C. `PASS`

```markdown
# QA — n1-vendedores-fix — 2026-09-11

**Veredicto: PASS**

## Resumen
3 tests · 3 pasan · 0 fallan · 0.16s

## Cobertura por criterio
| Criterio (del ADR) | Test | Estado |
|---|---|---|
| el listado hace ≤2 queries | `test_listar_items_no_hace_n_mas_1` | cubierto |
| la respuesta no cambia de forma | `test_lista_items_ok` | cubierto |
| vendedor inexistente → campos en `null`, no 500 | `test_vendedor_huerfano_no_rompe` | cubierto |

## Fallos
(ninguno)

## Comando
`pytest -q` desde `items-service/` — salida completa adjunta.
```

Trello: `QA` → `Done`.

---

## Qué NO hacer

- Reportar `Obtenido:` con un valor que no salió impreso por pytest.
- Poner `xfail` o `skip` sobre el rojo de un incidente "para que el CI quede
  verde" — el rojo es el punto.
- Mover un incidente a `Done` o a `In Progress`. Se queda en `QA`.
- Hardcodear `"QA"` / `"Done"` en vez de resolver con `get_lists`.
- Editar `main.py` porque "el fix era de una línea". Eso es tarea fallida.
