---
type: index
title: Índice del conocimiento de mabbux
description: Mapa de navegación de knowledge/; punto de entrada para encontrar principios, decisiones, arquitectura, convenciones y dominio
tags: [indice, navegacion, conocimiento]
timestamp: 2026-10-12T12:00:00Z
x-mabbux:
  status: accepted
  owner: "@albarracingarciap"
  provenance:
    author: agent
    agent: claude-code
---

# Índice del conocimiento

Empieza por aquí. Lee solo los documentos que necesite la tarea; cada fila dice de qué
trata cada uno. Al añadir, mover o retirar un documento, actualiza este índice y
`log.md`.

## Raíz

| Documento | Tipo | Estado | De qué trata |
| --- | --- | --- | --- |
| [principles.md](principles.md) | principles | accepted | Las siete reglas del proyecto; en conflicto gana la de número más bajo |
| [log.md](log.md) | log | proposed | Historial con fecha de cambios en el conocimiento y en el harness, y fricciones detectadas |

## decisions/ · decisiones de arquitectura (ADR)

| Documento | Tipo | Estado | De qué trata |
| --- | --- | --- | --- |
| [0001-node-y-typescript.md](decisions/0001-node-y-typescript.md) | decision | accepted | Node 24 LTS con TypeScript como stack del núcleo; revisión en F0 por latencia de hooks |

## architecture/ · arquitectura y diseño

| Documento | Tipo | Estado | De qué trata |
| --- | --- | --- | --- |
| [diseno-inicial.md](architecture/diseno-inicial.md) | architecture | proposed | Diseño completo: capas, funcionamiento en siete pasos, memoria anclada, enforcement, medición, seguridad, stack y roadmap |

## conventions/ · convenciones de código y proceso

Vacía por ahora.

## domain/ · glosario y conceptos del dominio

Vacía por ahora.

## Fuera de knowledge/

No forman parte del bundle, pero usan el mismo frontmatter:

- `specs/`: especificaciones del carril «especificación» (plantilla en `specs/_plantilla/`).
- `work/`: documentos de tarea del carril «cambio» (plantilla en `work/_plantilla.md`).
