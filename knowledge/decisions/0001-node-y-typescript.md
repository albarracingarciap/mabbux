---
type: decision
title: "0001: Node 24 LTS con TypeScript como stack del núcleo"
description: Se elige Node con TypeScript frente a Go y Bun, por neutralidad, aceptación corporativa y perfil del autor; la latencia de los hooks se valida en F0
tags: [adr, stack, node, typescript, runtime]
timestamp: 2026-10-12T12:00:00Z
x-mabbux:
  status: accepted
  owner: "@albarracingarciap"
  review_by: 2027-01-31
  provenance:
    author: human
---

# 0001: Node 24 LTS con TypeScript como stack del núcleo

**Estado:** aceptada · **Fecha:** 2026-10-12

## Contexto

mabbux es una CLI que se ejecuta en tres sitios: el portátil del desarrollador, los hooks de git y CI. Los hooks del asistente se lanzan en cada llamada a herramienta, así que el tiempo de arranque importa. Además, el producto aspira a ser neutral entre asistentes y a venderse a organizaciones reguladas.

## Opciones consideradas

| Opción | A favor | En contra |
| --- | --- | --- |
| Go | Binario único, arranque muy rápido, buen ecosistema (Sigstore, OCI, OTel) | Necesita CGO para tree-sitter y SQLite; lenguaje distinto del panel web futuro; curva para el autor |
| Bun + TypeScript | Ejecutable único, arranque rápido, SQLite integrado | Compatibilidad con Node no total; runtime controlado por un proveedor de asistentes (Anthropic), lo que choca con la neutralidad |
| **Node + TypeScript** | Gobernanza neutral (OpenJS Foundation), LTS predecible, compatibilidad total con npm, TypeScript nativo estable desde 24.12, mismo lenguaje que el panel en Next.js | Arranque algo más lento; el ejecutable único (SEA) aún no es estable |

## Decisión

Node 24 LTS con TypeScript ejecutado mediante *type stripping*, monorepo con pnpm y empaquetado con esbuild en un solo fichero. Se distribuye como el paquete npm `mabbux`.

## Consecuencias

- Solo se usa sintaxis TypeScript borrable: `import type` para tipos, sin `enum` ni decoradores.
- El punto de entrada de los hooks (`mabbux hook <evento>`) no importa nada pesado; carga módulos de forma diferida.
- La telemetría se escribe como JSON local y se exporta a OpenTelemetry en lote, nunca dentro del hook.
- Un benchmark de latencia de hooks forma parte de CI desde la primera semana.
- Para equipos que no usen JavaScript, se evaluará el ejecutable único de Node en F2.

## Criterio de revisión

Se revisa en F0. Si la latencia de los hooks no es aceptable, se reescribe **solo** el ejecutor de hooks en Rust y el resto sigue en TypeScript; la separación en paquetes lo permite.
