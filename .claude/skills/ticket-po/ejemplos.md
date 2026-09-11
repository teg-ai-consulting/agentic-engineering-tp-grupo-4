# ticket-po — ejemplos

## A. `tipo: feature` bien formado

**Título:** `[P2] items-service, web/ — badge "Nuevo" en items publicados esta semana`

**Cuerpo:**

```
tipo: feature · servicio(s): items-service, web/

## Contexto
Los items recién publicados se pierden entre los viejos en el home y los
vendedores nuevos no reciben visitas la primera semana. Queremos que el
comprador distinga de un vistazo lo que acaba de entrar.

## Criterios de aceptación
- [ ] un item con `publicado_en` de menos de 7 días muestra el badge "Nuevo" en el listado
- [ ] un item con exactamente 7 días NO muestra el badge
- [ ] un item sin `publicado_en` no muestra badge y no rompe el listado
- [ ] el listado sigue devolviendo nombre y reputación del vendedor como hoy

## Fuera de alcance
- ordenar el listado por fecha (sigue por id)
- el badge en la vista de detalle del item

## Señales
listado (afecta `GET /v1/items`)

owner: @lautaro
```

**Checklist de la card** (los mismos 4 criterios, uno por ítem — es lo que lee
`get_acceptance_criteria`).

**Label:** `tipo: feature` · **Columna:** `To-Do`

---

## B. `tipo: incidente` bien formado

**Título:** `[P1] items-service — timeout en GET /v1/items bajo carga`

**Descripción:** el contenido de `incidente/alerta.txt` pegado tal cual.

**Adjuntos:** `alerta.txt`, `metricas.md`, `slow-query.log`.

**Comentario aparte** (hipótesis, no hecho):

```
Hipótesis del SRE, sin confirmar: "es la promo, escalen más réplicas".
Queda registrada como hipótesis — el diagnóstico lo firma el investigador.
```

Sin criterios de aceptación: un incidente lleva evidencia.

**Label:** `tipo: incidente` · **Columna:** `To-Do`

---

## C. Tickets mal formados → qué preguntar

| Lo que llega | El problema | Qué pedís |
|---|---|---|
| "Mejorar el listado del home" | no es accionable, no dice servicio ni qué se observa | título `[P?] <servicio> — <qué>` + el problema concreto |
| "Que ande rápido" | el `qa` no puede escribir un assert | un número: ¿rápido es ≤300ms? ¿con qué `limit`? |
| "Hay un N+1, decile al dev que use `WHERE id IN`" | es la solución, y le habla al agente | el **síntoma** observado; el cómo es del `arquitecto`/ADR |
| "tipo: fix — arreglar el N+1" | el PO no crea cards de fix | si es un problema en prod → `tipo: incidente` con la evidencia |
| feature sin `Fuera de alcance` | el `analista` tiene la sección y la deja vacía | qué NO entra (aunque sea una línea) |
| criterios solo en el cuerpo | `get_acceptance_criteria` lee la **checklist** | cargarlos también como checklist |

---

## D. Qué NO hacer

- Poner la causa raíz supuesta en la descripción de un incidente como si fuera
  un hecho.
- Escribir "urgente, P0" sin que el PO lo haya decidido — la prioridad la pone
  el PO, no el formato.
- Crear la card ya en `In Progress`, o con `en-proceso` / `revisión-adr`. Esos
  labels los maneja el guardia.
- Dejar un criterio en blanco "porque el dev ya sabe".
