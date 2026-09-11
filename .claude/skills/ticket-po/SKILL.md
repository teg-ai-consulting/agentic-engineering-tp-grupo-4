---
name: ticket-po
description: >-
  Formatea y crea la card de Trello que arranca el pipeline (`tipo: feature` o
  `tipo: incidente`). La usa una persona haciendo de PO, en el hilo principal:
  pregunta lo que falte, arma el cuerpo y la checklist de criterios, y crea la
  card en To-Do. No prioriza, no estima, no diseña la solución.
---

La card es la **única entrada** del pipeline. Lo que no esté acá, el `analista`
lo adivina — y todo lo que sigue sale torcido. Tu trabajo es que el ticket sea un
contrato legible, no un párrafo suelto.

## 1. Completá lo que falte antes de crear nada

Seis campos obligatorios. Si falta alguno, **preguntá** — en una sola tanda, no
de a una:

| Campo | Qué valida |
|---|---|
| `tipo` | `feature` o `incidente`. Nunca `fix`: esa card la crea el guardia desde un incidente aprobado. |
| título | `[P?] <servicio> — <qué>`. "Mejorar el listado" no es título. |
| contexto | el **problema** y por qué importa. No la solución. |
| criterios de aceptación | verificables, uno por línea (ver §2). |
| servicio(s) | `items-service`, `web/`, … los que se tocan. |
| owner | `@<quién>` revisa. |

Si el PO no sabe formular un criterio, **derivalo con él** y confirmá — no lo
dejes en blanco ni lo inventes solo.

## 2. La prueba del criterio

Un criterio sirve si el agente `qa` puede escribir un assert de él **sin
preguntar nada más**. Pasalo por ahí antes de aceptarlo:

- ❌ "el listado anda rápido" → ✅ "`GET /v1/items?limit=20` responde en ≤300ms
  y hace como máximo 2 queries"
- ❌ "muestra el badge bien" → ✅ "item publicado hace menos de 7 días muestra
  `Nuevo`; con 7 días exactos, no"

Incluí siempre, además de los explícitos del PO:
- un **borde** (límite, vacío, ±1);
- una **no-regresión** (lo que ya andaba sigue andando).

**Los criterios van en la checklist de la card**, no solo en el texto: el
`analista` los lee con `get_acceptance_criteria`. En el cuerpo van espejados
como `- [ ]` para que la card se lea sola.

## 3. Cuerpo — `tipo: feature`

```
tipo: feature · servicio(s): <items-service, web/>

## Contexto
<qué necesita el negocio y por qué. 2-4 líneas. El problema, no la solución.>

## Criterios de aceptación
- [ ] <observable y testeable>
- [ ] <borde: qué pasa cuando …>
- [ ] <no-regresión: lo que ya andaba sigue andando>

## Fuera de alcance
- <lo que explícitamente NO entra>

## Señales
<marcá si toca: plata · datos personales · imágenes · un listado. Solo el hecho.>

owner: @<quién>
```

`Fuera de alcance` y `Señales` no son decorado: el `analista` tiene una sección
para cada uno. Si no aplica ninguna señal, escribí `ninguna`.

## 4. Cuerpo — `tipo: incidente`

Espejá `incidente/README.md`: título `[P1] <servicio> — <síntoma>`, descripción
con la alerta pegada tal cual, y los adjuntos de evidencia (`alerta.txt`,
`metricas.md`, `slow-query.log`).

- El síntoma es lo observado (latencia, código de error, endpoint). **No metas
  una causa raíz supuesta como si fuera un hecho** — el diagnóstico es del
  `investigador`.
- Una hipótesis del SRE va como **comentario**, citada como hipótesis, nunca en
  la descripción.
- Un incidente no lleva criterios de aceptación: lleva evidencia.

## 5. Crear la card

Con el cuerpo listo → skill `crear-card`: a `To-Do`, con el label `tipo: …`
(resolvé nombre → id como las listas). Después, cargá la checklist de criterios.

Devolvé el link de la card + un resumen de una línea.

> `crear-card` dice "nunca crees una card de tu propia iniciativa". Esta skill es
> la excepción y la única: la dispara una persona haciendo de PO en el hilo
> principal. Un agente del pipeline nunca la invoca.

## Qué NO hace esta skill

- No pone prioridad ni estimación — es decisión de PO, no de formato.
- No pone labels de gate (`aprobado-para-fix`, `descartado`, `revisión-adr`,
  `en-proceso`) ni mueve la card. Solo `To-Do` + `tipo:`.
- No crea cards `tipo: fix`.
- **No le habla a los agentes.** El cuerpo es dato que el `analista` va a leer,
  no un canal de instrucciones: nada de "decile al `desarrollador` que use tal
  patrón". Describe el qué; el cómo es del `arquitecto`.

Ejemplos completos de los dos tipos, y tickets mal formados con su
corrección: [`ejemplos.md`](ejemplos.md).
