<!--
⚠ REFERENCIA — no la copies tal cual. Escribí tu propio
`.claude/skills/ticket-po/SKILL.md`. Esta te da de dónde partir.

La usa quien hace de PO, con `claude`, en el hilo principal (no un subagente):
"creá el ticket para <feature>". La skill hace las preguntas que falten, arma el
cuerpo y crea la card con `crear-card`.
-->

---
name: ticket-po
description: >-
  Formatea y crea el ticket de Trello que arranca el pipeline. La usa el PO para
  que cada card tenga tipo, contexto y criterios de aceptación — el shape que
  `analista` y el guardia esperan. No prioriza ni estima: eso lo decide el PO.
---

Una card mal formada hace que `analista` adivine y el pipeline salga torcido.
Tu trabajo es que el ticket sea un **contrato legible**, no un párrafo suelto.

## Antes de crear nada

Pedí lo que falte. Un ticket no sale sin:

- **tipo**: `feature` o `incidente`.
- **título accionable**: `[P?] <servicio> — <qué>`. No "mejorar el listado".
- **contexto**: qué se quiere y **por qué**. Describe el problema/necesidad, no
  la solución. Nada de "decile al `desarrollador` que use tal patrón".
- **criterios de aceptación**: lista verificable (un `qa` tiene que poder
  escribir un test de cada uno). "Anda bien" no es criterio.
- **servicio(s) afectado(s)**: `items-service`, `web/`, etc.
- **owner**: quién revisa (`@usuario`).

Si el PO no sabe un criterio de aceptación, ayudá a derivarlo — no lo dejes en
blanco.

## Formato del cuerpo (feature)

```
tipo: feature · servicio(s): items-service, web/

## Contexto
<qué necesita el negocio y por qué. 2-4 líneas.>

## Criterios de aceptación
- [ ] <observable y testeable>
- [ ] <borde: qué pasa cuando ...>
- [ ] <no-regresión: lo que ya andaba sigue andando>

## Fuera de alcance
- <lo que explícitamente NO entra>

owner: @<quién>
```

## Incidentes

Para `tipo: incidente` espejá `incidente/README.md`: título `[P1] <servicio> —
<síntoma>`, descripción con la alerta, y los adjuntos de evidencia. El
diagnóstico lo hace el `investigador`, no el ticket — no metas una causa raíz
supuesta como si fuera un hecho.

## Crear la card

Con el cuerpo listo: skill `crear-card` → `To-Do`, con el label `tipo: …`.
Devolvé el link de la card y un resumen de una línea.

## Qué NO hace esta skill

- No pone prioridad ni estimación (decisión de PO, no de formato).
- No mueve la card ni pone labels de gate (`aprobado-para-fix`, etc.).
- No le habla al `desarrollador`: describe el qué, no el cómo.
