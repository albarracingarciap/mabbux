# AGENTS.md

mabbux es un harness gobernable para agentes de código: memoria anclada al código,
enforcement en git y CI, y evidencia de cada cambio. CLI en Node 24 LTS con TypeScript.
Este repositorio es su primer usuario: se construye a mano siguiendo el proceso que
mabbux automatizará. Responde en español; código y comentarios en inglés.

## Antes de actuar

1. Lee `knowledge/principles.md`. Si una propuesta choca con un principio, dilo antes.
2. Clasifica la tarea en un carril. Por defecto, «cambio».
3. Explica qué vas a hacer y espera aprobación antes de crear o modificar ficheros.

## Carriles

| Carril | Cuándo | Qué exige |
| --- | --- | --- |
| Respuesta | Preguntas, análisis, revisión. No escribe ficheros | Nada |
| Cambio | Cualquier modificación que no sea trivial ni toque rutas de spec | Documento en `work/AAAA-MM-DD-slug.md` desde `work/_plantilla.md`, con evidencia |
| Especificación | Cambios grandes o que tocan una ruta de la lista de abajo | `specs/NNNN-slug/` desde `specs/_plantilla/`, con `status: accepted` antes de implementar |

**Trivial** = erratas, formato y comentarios, sin cambio de comportamiento: sin documento
de tarea, pero con commit descriptivo. **Nunca es trivial** cambiar `AGENTS.md`, `CLAUDE.md`
o las plantillas (`specs/_plantilla/`, `work/_plantilla.md`): carril «cambio».

**Rutas que exigen spec** (`lanes.spec_required_paths` en `.mabbux/config.yaml`, que manda
si difiere de esta lista):

- `packages/core/src/policy/**` y `packages/core/src/evidence/**`
- `scripts/mabbux/**`, `.github/workflows/**`, `.mabbux/config.yaml`

**Ciclo de un documento de tarea:** el plan se aprueba en el chat; el agente implementa y
anota la evidencia en el documento; la persona revisa el resultado y pasa `status` a `accepted`.
En `work/`, `specs/` y `knowledge/`, `accepted` significa siempre «revisado y aprobado por
una persona». El agente nunca pone `accepted`.

## Evidencia

Hasta que exista `done.sh`, la evidencia se anota en el documento de tarea o en la spec:
comando, código de salida y resumen de la salida. No escribas en
`.mabbux/state/evidencias.jsonl`: ese registro solo lo escribe el harness.
Hoy ningún check de git ni de CI hace cumplir estas reglas (principio 1); no son opcionales.

## Git

- Una rama por tarea: `feat/…`, `fix/…`, `docs/…`. Nunca trabajes en `main`.
- No hagas commit ni push salvo que la persona lo pida.
- Mensajes de commit en inglés, con formato Conventional Commits.

## Comandos

Aún no existen. Se definirán en la sesión de scripts (`scripts/mabbux/`, que exige spec).

## Dónde está cada cosa

| Qué | Dónde |
| --- | --- |
| Mapa de todo el conocimiento | `knowledge/index.md` (empieza siempre por aquí) |
| Principios · decisiones (ADR) | `knowledge/principles.md` · `knowledge/decisions/` |
| Arquitectura · convenciones · dominio | `knowledge/architecture/` · `conventions/` · `domain/` |
| Historial del conocimiento y fricciones | `knowledge/log.md` |
| Configuración del harness | `.mabbux/config.yaml` |
| Skills y subagentes | `.mabbux/skills/`, `.mabbux/agents/` |

Todo documento de `knowledge/`, `specs/` y `work/` lleva frontmatter OKF con `x-mabbux`
(ejemplo: `knowledge/principles.md`). Lo que escribe un agente nace con `status: proposed`
y `provenance.author: agent`. Al añadir o mover un documento de `knowledge/`, actualiza
`knowledge/index.md` y añade una entrada en `knowledge/log.md`.
