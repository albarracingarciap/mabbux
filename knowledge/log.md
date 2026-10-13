---
type: log
title: Diario del conocimiento y del harness de mabbux
description: Entradas con fecha, lo más reciente arriba, sobre cambios en knowledge/ y en el harness, supuestos tomados y fricciones del proceso
tags: [log, historial, friccion]
timestamp: 2026-10-12T12:00:00Z
x-mabbux:
  status: accepted
  owner: "@albarracingarciap"
  provenance:
    author: agent
    agent: claude-code
---

# Diario

Lo más reciente, arriba. Cada entrada lleva fecha y, si aplica, una etiqueta:
`[harness]`, `[conocimiento]` o `[fricción]`.

## 2026-10-12 · [fricción] Rutas de spec duplicadas entre AGENTS.md y config.yaml

`AGENTS.md` repite la lista `lanes.spec_required_paths` de `.mabbux/config.yaml` para que
el agente la vea sin abrir la configuración. Son dos fuentes de verdad: si alguien cambia
`config.yaml` (que exige spec) y no actualiza `AGENTS.md`, el agente seguirá unas rutas
equivocadas sin que nada lo detecte. Hoy lo mitiga solo una frase de `AGENTS.md`: «manda
`config.yaml` si difiere».

Solución esperada: que el harness genere ese bloque de `AGENTS.md` a partir de
`config.yaml` (`mabbux sync`) y que un check compruebe que coinciden (principios 1 y 2).
Tarea: `work/2026-10-12-revision-agents-md.md`.

## 2026-10-12 · [harness] Revisión de AGENTS.md antes del primer commit

Se añaden las secciones «Git» y «Comandos», el ciclo de vida de `work/` y la definición
de `accepted` («revisado y aprobado por una persona») para `work/`, `specs/` y
`knowledge/`. `AGENTS.md` pasa a 68 líneas. Tarea:
`work/2026-10-12-revision-agents-md.md`.

## 2026-10-12 · [fricción] El registro de evidencias no puede escribirlo el modelo

Mientras no exista `done.sh`, la evidencia se anota en el documento de tarea o en la spec,
y no se toca `.mabbux/state/evidencias.jsonl`.

Por qué: el principio 1 exige que lo que importa se compruebe fuera del modelo. Si el
agente escribiera el registro, la evidencia sería una afirmación del propio modelo, no la
salida verificada de un comando: podría inventar, resumir mal o copiar resultados
antiguos, y nada lo distinguiría de una ejecución real. El registro solo vale si lo
escribe el harness tras ejecutar él mismo el comando (paso 5 de «Funcionamiento» en
`architecture/diseno-inicial.md`). Mientras tanto, la evidencia anotada a mano es
orientativa: una persona la revisa en la PR.

Coste: hasta tener `done.sh`, no hay garantía de que la evidencia sea reproducible.

## 2026-10-12 · [harness] Capa inicial del harness creada a mano

Hecho (por claude-code, pendiente de revisión):

- `AGENTS.md` (60 líneas) con los tres carriles, la definición de «trivial», las rutas
  que exigen spec y el mapa de dónde está cada tipo de conocimiento. `CLAUDE.md` solo
  contiene `@AGENTS.md`.
- `knowledge/index.md`, este `log.md` y las carpetas `conventions/` y `domain/`.
- Plantillas `specs/_plantilla/{spec,plan,tasks}.md` y `work/_plantilla.md`, todas con
  frontmatter `x-mabbux` y una sección de evidencia esperada.
- `.mabbux/skills/` y `.mabbux/agents/` vacías.
- `.gitignore`: `.env`, `.env.*`, `!.env.example`, `.claude/settings.local.json`.
- Frontmatter OKF añadido a `architecture/diseno-inicial.md` (`type: architecture`,
  `status: proposed`, `author: human`). El cuerpo no cambia: se ha comprobado con su hash.

Decisiones de la persona responsable en esta sesión:

- Trivial = erratas, formato y comentarios, sin cambio de comportamiento: sin documento
  de tarea, con commit descriptivo.
- `AGENTS.md`, `CLAUDE.md` y las plantillas no exigen spec, pero cambiarlos nunca es
  trivial: carril «cambio» con documento de tarea.
- `specs/` y `work/` usan el mismo frontmatter. Aprobar significa que una persona cambia
  `status` a `accepted`.

Supuestos tomados:

- `type: index` y `type: log` para estos dos documentos, y `type: spec`, `plan`, `tasks`
  y `task` en las plantillas. Son valores propios: OKF v0.1 no fija una lista cerrada que
  yo conozca.
- `owner: "@albarracingarciap"` en todo lo creado. Sin `evidence` ni `anchors`, porque no
  hay registro de evidencias y estos documentos no se anclan a código.
- `domain/` cubre el «glosario» del diseño. `integrations/` no se crea hasta que haga
  falta.
- No se crea `.github/copilot-instructions.md`, porque Copilot lee `AGENTS.md`.
- `timestamp: 2026-10-11T00:00:00Z` en `diseno-inicial.md`, tomado de la fecha de su
  cabecera; la hora es desconocida.
- `done.sh` se cita sin ruta. Se supone que vivirá en `scripts/mabbux/`, que exige spec.
- Nombres de las instancias: `work/AAAA-MM-DD-slug.md` y `specs/NNNN-slug/`.
- Este cambio no tiene documento en `work/` porque crea las propias plantillas.

Pendiente: ningún check determinista hace cumplir todavía los carriles ni la evidencia
(principio 1). Hoy solo lo pide `AGENTS.md`.
