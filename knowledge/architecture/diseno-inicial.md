---
type: architecture
title: Diseño inicial del harness gobernable para agentes de código
description: Visión, arquitectura en tres capas, funcionamiento, memoria anclada al código, enforcement, medición, seguridad, implementación y roadmap de mabbux
tags: [arquitectura, diseno, harness, memoria, enforcement, roadmap]
timestamp: 2026-10-11T00:00:00Z
x-mabbux:
  status: accepted
  owner: "@albarracingarciap"
  provenance:
    author: human
---

# Harness gobernable para agentes de código: diseño e implementación

Oct 11, 2026 · @Pedro

## Resumen ejecutivo

La propuesta es un harness para agentes de código que sea **gobernable, medible y auditable**, e independiente del asistente que use cada desarrollador. Toma la idea de rsc-harness (memoria, conexiones, skills y comprobaciones) pero cambia dónde reside la garantía: en git y en CI, no en las instrucciones al modelo.

Tres diferencias frente a rsc-harness:

- **Memoria anclada al código.** Cada hecho de la wiki se vincula a ficheros, símbolos y commits, y se marca como sospechoso cuando ese código cambia.
- **Enforcement por capas.** Los hooks del asistente dan feedback rápido; git y CI son la garantía que vale igual para Claude Code, Copilot, Cursor o Codex.
- **Valor demostrado.** Telemetría local y evals que comparan el trabajo con y sin harness, en lugar de afirmar el ahorro.

Para quien trabaja solo, aporta continuidad entre sesiones y menos retrabajo con un coste de configuración casi nulo. Para equipos, añade políticas heredables entre repositorios, un registro privado de skills y trazabilidad de lo que hace cada agente. Esto último es lo que piden sectores regulados como banca, utilities o administración pública.

## Estado del arte a octubre de 2026

Los formatos ya están estandarizados; el hueco está en gobierno, verificación y medición. El diseño no debe inventar formatos propios, sino adoptar los estándares y aportar la capa que ninguno cubre.

| Estándar o técnica | Estado actual | Implicación para el diseño |
| --- | --- | --- |
| [AGENTS.md](https://openai.com/index/agentic-ai-foundation/) | Donado a la Agentic AI Foundation (Linux Foundation) en diciembre de 2025, junto a MCP y goose | Fuente única de instrucciones del proyecto. Los ficheros propios de cada asistente se generan como adaptadores |
| [Agent Skills](https://agentskills.io/) | Estándar abierto desde diciembre de 2025, con carga progresiva y adopción en Claude, Codex, Copilot, Cursor, Gemini CLI y otros. La especificación no cubre propiedad, revisión, evaluación, firma ni ciclo de vida ([análisis](https://rubinlake.com/en/technology-radar/developer-ai-and-delivery/agent-skills)) | Formato nativo de nuestras skills. El valor añadido es gobernar su ciclo de vida |
| [MCP](https://obot.ai/blog/mcp-is-growing-up-the-2026-roadmap-takes-shape/) | Especificación 2026-07-28 con núcleo sin estado y autorización endurecida. Auditoría, SSO y comportamiento de gateway siguen sin definir en la hoja de ruta | Conexiones vía MCP detrás de un gateway propio. La auditoría la resolvemos nosotros |
| [Open Knowledge Format](https://agentic-ai.readthedocs.io/en/latest/Standards/open-knowledge-format/) | v0.1 publicada por Google Cloud el 12 de junio de 2026: markdown con YAML, enlaces como grafo | Formato de la memoria del proyecto, extendido con campos de anclaje y confianza |
| [Context engineering](https://www.thoughtworks.com/radar/techniques/progressive-context-disclosure) | Thoughtworks Radar sitúa la divulgación progresiva en "Trial" (abril de 2026). El riesgo conocido: el contexto que nunca se carga falla en silencio | Cuidar índices y descripciones, y medir con evals si cada skill se activa cuando debe |
| [Spec-driven development](https://ssojet.com/blog/best-spec-driven-development-tools) | Spec Kit superó 115.000 estrellas en junio de 2026; Kiro lo integra en el IDE. Crítica común: falta un bucle de verificación real ([fuente](https://codemyspec.com/blog/spec-kit-vs-kiro)) | No reinventar SDD: integrar Spec Kit u OpenSpec y aportar la verificación |
| [OpenTelemetry GenAI](https://www.truefoundry.com/blog/opentelemetry-genai-semantic-conventions) | Convenciones movidas a un repositorio propio en junio de 2026, aún en estado Development. Incluyen invoke\_agent, execute\_tool y spans de MCP | Telemetría en OTel desde el primer día, aislando la versión de las convenciones |
| [Seguridad de agentes](https://www.helpnetsecurity.com/2026/06/11/owasp-prompt-injection-ai-security-failures) | La "lethal trifecta" (datos privados, contenido no fiable, salida externa) es el modelo de amenaza de referencia. Hubo campañas con 1.184 skills maliciosas en un marketplace ([fuente](https://www.morphllm.com/ai-coding-agent-security)) | Firma y allowlist de skills, escaneo antes de instalar, y romper una pata de la trifecta por ejecución |
| [Gobierno nativo de los asistentes](https://code.claude.com/docs/en/plugins/org) | Claude Code ya ofrece managed settings, marketplaces privados y hooks solo gestionados. No hay firma binaria de plugins ([fuente](https://dev.to/michaeltuszynski/your-claude-plugin-marketplace-needs-more-than-a-git-repo-5631)) | No competir con ello: traducir una política común a los controles nativos de cada asistente |
| [AI Act (Omnibus)](https://www.gibsondunn.com/eu-ai-act-omnibus-agreement-postponed-high-risk-deadlines-and-other-key-changes/) | Acuerdo provisional: obligaciones de alto riesgo del Anexo III aplazadas al 2 de diciembre de 2027 | Los asistentes de código rara vez son alto riesgo, pero los clientes regulados piden evidencias de uso de IA para ISO/IEC 42001 y auditoría interna |

La consecuencia práctica: el producto es una **capa de control y evidencia** sobre estándares abiertos, no un catálogo más de skills.

## Principios de diseño

Siete principios guían todas las decisiones posteriores. Cuando dos entren en conflicto, gana el de número más bajo.

1. **Lo que importa se comprueba fuera del modelo.** Una regla vinculante tiene un check determinista en git o CI. Las instrucciones al agente solo aceleran el cumplimiento.
2. **Estándares antes que formatos propios.** AGENTS.md, Agent Skills, MCP, OKF y OpenTelemetry. Lo propio se limita a extensiones en el frontmatter.
3. **Todo conocimiento tiene procedencia.** Quién o qué lo escribió, con qué evidencia, anclado a qué código y hasta cuándo es válido.
4. **Contexto mínimo y medido.** Nada se carga siempre si puede cargarse bajo demanda, y cada activación se registra para saber si sirve.
5. **Fricción proporcional al riesgo.** Silencio para cambios triviales, verificación para cambios normales, especificación y aprobación para cambios grandes o sensibles.
6. **Local primero, organización opcional.** Funciona completo en un portátil sin cuenta ni red. La capa de organización se añade sin cambiar el flujo del desarrollador.
7. **Nada se escribe sin plan aceptado.** Se hereda de rsc-harness: el instalador muestra qué añade, qué difiere y qué excluye, con motivo.

## Arquitectura

Tres capas con una sola pieza central: el binario `mabbux`, que corre igual en el portátil, en los hooks de git y en CI.

&#91;embedded content: arquitectura · 3 capas, 11 componentes\]

Los asistentes solo ven adaptadores generados y acceden a herramientas y memoria a través del gateway MCP. El check de CI es el único control que no se puede saltar; la organización aporta políticas y skills firmadas y recibe la trazabilidad de vuelta.

## Funcionamiento

Una tarea recorre siete pasos, desde la instalación hasta el merge. En los asistentes sin hooks, los pasos 2 a 5 se apoyan en instrucciones de AGENTS.md; los pasos 6 y 7 son idénticos para todos.

1. **Instalación (`mabbux init`).** Detecta stack y asistentes presentes, importa AGENTS.md, CLAUDE.md o reglas existentes y hace pocas preguntas. Muestra el plan (añade, difiere, excluye) y escribe solo tras aceptarlo. Genera una primera versión de la memoria a partir del código, marcada como `proposed`.
2. **Inicio de sesión.** Un hook de arranque ejecuta `mabbux context` e inyecta un resumen acotado: punto de control de la rama, tareas abiertas y avisos de conocimiento obsoleto en la zona en la que se va a trabajar.
3. **Clasificación de la tarea.** Se asigna un carril: respuesta (no escribe), cambio (un documento de tarea con evidencia) o especificación (spec aprobada antes de implementar). La política fuerza el carril de especificación en rutas sensibles, por ejemplo `payments/` o `auth/`.
4. **Ejecución.** Las skills se cargan bajo demanda y las herramientas externas pasan por el gateway MCP. Los hooks previos a cada herramienta bloquean lo evidente: secretos, comandos destructivos, rutas prohibidas.
5. **Cierre con evidencia.** El agente declara `mabbux done --evidence "npm test"`. El harness ejecuta él mismo el comando y guarda el resultado en el registro de evidencias. Los cambios de conocimiento quedan como propuestas, no como hechos aceptados.
6. **Commit.** El hook de git verifica evidencia, secretos y políticas, y añade *trailers* al commit: asistente, modelo e identificador de evidencia.
7. **Pull request.** CI vuelve a ejecutar la evidencia, revisa obsolescencia y políticas y publica un resumen de procedencia en la PR. La persona revisora acepta código y conocimiento a la vez; lo aceptado pasa de `proposed` a `accepted`.

En equipo no hace falta una rama de intercambio de conocimiento: la memoria viaja en la misma PR que el código que la justifica, y CODEOWNERS asigna quién aprueba cada área de conocimiento.

## Memoria anclada al código

La memoria es un bundle OKF en `knowledge/` cuyos documentos llevan anclajes a símbolos del código. Cuando un símbolo anclado cambia, el documento pasa a `stale` y el agente lo sabe antes de fiarse de él.

**Tipos de documento.** Decisiones (ADR), convenciones, arquitectura, integraciones, glosario y trampas conocidas. Cada documento contiene un concepto; así los merges entre varios agentes chocan poco.

**Frontmatter.** Los campos OKF estándar más una extensión `x-mabbux` que el resto de consumidores OKF puede ignorar:

```yaml
---
type: convention
title: Paginación por cursor en la API pública
description: Todos los endpoints de listado usan cursor opaco, nunca offset
tags: [api, paginacion]
timestamp: 2026-10-11T18:00:00Z
x-mabbux:
  status: accepted          # proposed | accepted | stale | retired
  owner: "@equipo-api"
  review_by: 2027-04-11
  confidence: 0.9
  provenance:
    author: agent           # agent | human
    agent: claude-code
    evidence: ldg_01JA7Q   # entrada del registro de evidencias
  anchors:
    - path: src/api/pagination.ts
      symbol: encodeCursor
      commit: 4f2c9e1
      hash: sha256:9b1e…     # hash del cuerpo del símbolo
---
```

**Detección de obsolescencia.** En cada commit, `mabbux` calcula qué símbolos cambiaron (tree-sitter, con respaldo por rango de líneas en lenguajes sin gramática) y compara hashes. Un documento con un anclaje roto pasa a `stale`. CI avisa si una PR toca código cuyo conocimiento queda obsoleto y bloquea, según política, si es conocimiento `accepted` en un área crítica.

**Recuperación.** Tres niveles, de más barato a más caro:

- `index.md` jerárquico y AGENTS.md por directorio que apuntan al conocimiento relevante.
- Un servidor MCP local `knowledge.search` con búsqueda híbrida (texto y vectores en SQLite) que devuelve solo títulos y descripciones.
- Lectura del documento completo solo cuando el agente lo pide.

**Ciclo de vida.** Lo que escribe un agente nace como `proposed`. Solo pasa a `accepted` con revisión humana en una PR. Lo que procede de contenido no fiable (issues, web, correos) no puede promocionarse sin revisión, lo que frena el envenenamiento de memoria. El diario de sesión es local, no se comete y no guarda prompts ni respuestas.

## Enforcement en tres niveles

La garantía reside en CI; los otros dos niveles existen para que el agente corrija antes y más barato. Así una regla vale igual con cualquier asistente, y la asimetría de rsc-harness (guardas duros solo en Claude Code) desaparece.

| Nivel | Dónde corre | Asistentes cubiertos | Para qué | Se puede saltar |
| --- | --- | --- | --- | --- |
| 1. Hooks del asistente | Eventos de sesión, prompt, herramienta y parada | Claude Code, Codex, Gemini CLI y OpenCode completos; Cursor asistido; el resto vía AGENTS.md | Feedback en segundos: bloquear secretos, rutas prohibidas, cierre sin evidencia | Sí, si el asistente no tiene hooks |
| 2. Git local | pre-commit y pre-push (lefthook o pre-commit) | Todos | Mismos checks que CI en versión rápida | Sí, con `--no-verify` |
| 3. CI y PR | GitHub Actions, GitLab CI o Azure DevOps, como check obligatorio con protección de rama | Todos | Garantía final y resumen de procedencia en la PR | Solo con permisos de administración |

Todos los niveles llaman al mismo binario (`mabbux verify`), con la misma política, para que no haya reglas que digan una cosa en local y otra en CI.

**Checks del núcleo:**

- Evidencia presente y reproducible: CI re-ejecuta los comandos registrados y compara resultados.
- Secretos: escaneo con gitleaks o equivalente.
- Conocimiento obsoleto en el área tocada.
- Rutas sensibles: exigen spec aprobada enlazada en la PR.
- Integridad del harness: skills y políticas coinciden con los digests de `mabbux.lock`.
- Configuración MCP: solo servidores de la allowlist.
- Trailers de procedencia presentes en commits de agente.

## Capa de organización

La capa de organización define una política común y la traduce a los controles nativos de cada asistente, a CI y al gateway MCP. No sustituye a las managed settings de Claude Code ni a las de otros asistentes: las genera.

**Policy packs.** Un repositorio versionado con la política de la organización. Cada repositorio la hereda con `extends:` y solo puede endurecerla, no relajarla. `mabbux policy compile` produce:

- managed settings de Claude Code (marketplaces permitidos, hooks gestionados, reglas de permisos);
- instrucciones y reglas para Copilot, Cursor y el resto;
- configuración del gateway MCP y del job de CI.

**Registro de skills.** Las skills se publican como artefactos OCI firmados con Sigstore (cosign) en el registro del propio cliente, y se fijan por digest en `mabbux.lock`. El circuito de publicación es: autoría, evals de activación y resultado, escaneo de seguridad, revisión humana y firma. Esto cubre lo que la especificación de Agent Skills deja fuera: propiedad, revisión, evaluación, firma y ciclo de vida.

**Gateway MCP.** Proxy con allowlist de servidores, tokens con alcance mínimo, autenticación vía SSO y registro de cada llamada. Fija el hash de las descripciones de herramientas para detectar cambios silenciosos (un vector conocido de inyección).

**Conocimiento compartido.** Bundles OKF de organización (convenciones, glosario, arquitectura de referencia) que los repositorios referencian por versión. Aquí converge la idea de conocimiento corporativo preparado para IA: el mismo bundle alimenta a RAG y a agentes de código.

**Auditoría y cumplimiento.** El registro de evidencias, los trailers de commit y la telemetría OTel permiten reconstruir qué hizo cada agente, con qué herramientas y quién lo aprobó. `mabbux report` genera un informe periódico pensado como evidencia para un sistema de gestión ISO/IEC 42001: inventario de agentes y skills en uso, controles aplicados, incidencias y supervisión humana.

## Medición

El producto tiene que poder demostrar su valor con datos del propio repositorio; sin eso es una afirmación más. Toda la telemetría sigue las convenciones GenAI de OpenTelemetry (`invoke_agent`, `execute_tool`, spans de MCP) y por defecto no guarda prompts ni respuestas.

| Métrica | Cómo se mide | Qué indica |
| --- | --- | --- |
| Tokens por tarea cerrada | Uso de tokens en spans, dividido por tareas con evidencia | Coste real, no coste por sesión |
| Lecturas redundantes | Mismo fichero leído más de una vez en una sesión | Si la memoria evita exploración inútil |
| Verde a la primera | Evidencia que pasa sin reintentos | Calidad del primer intento |
| Retrabajo a 14 días | Líneas de commits de agente modificadas o revertidas después (git blame) | Si el código aguanta |
| Código duplicado introducido | Detección de clones sobre el diff | Si reutiliza lo existente |
| Precisión de activación de skills | Evals de enrutado por skill | Si el contexto correcto llega cuando toca |
| Conocimiento obsoleto | Porcentaje de documentos `stale` y días hasta su revisión | Salud de la memoria |
| Tasa de desactivación | Desarrolladores que apagan hooks o usan `--no-verify` | Si la fricción es excesiva |

**Evals.** Dos niveles:

- Por skill: casos de enrutado (debe activarse y no debe activarse) y de resultado, ejecutados al publicar.
- Por repositorio: una suite de tareas reales sacada del historial. Se toman PRs ya mergeadas, se revierte el cambio y se pide al agente que lo rehaga; se compara con el original mediante tests y revisión.

**A/B.** `mabbux eval ab` alterna sesiones con y sin harness sobre la misma suite y genera un informe con intervalos de confianza. Es la respuesta a "¿merece la pena?" y el argumento de venta para equipos.

## Seguridad

El modelo de amenaza es la "lethal trifecta": datos privados, contenido no fiable y capacidad de enviar datos fuera. Como la inyección de prompts no se filtra de forma fiable, el diseño rompe al menos una pata en cada ejecución y limita el radio de daño.

| Riesgo | Control |
| --- | --- |
| Inyección desde issues, web o correos | La sesión que lee contenido no fiable queda marcada: el gateway corta la salida externa y no se le entregan secretos |
| Skills maliciosas | Solo skills firmadas y fijadas por digest; escaneo antes de publicar; los scripts de terceros requieren revisión humana |
| Servidores MCP comprometidos | Allowlist, tokens de alcance mínimo y hash de las descripciones de herramientas para detectar cambios |
| Secretos en el repositorio | Las conexiones referencian un almacén (`secret://vault/...`), nunca valores; escaneo en pre-commit y CI |
| Envenenamiento de memoria | Lo que procede de fuentes no fiables nunca pasa a `accepted` sin revisión humana |
| Instalador comprometido | Versiones fijadas (nunca `@latest` en equipos), releases firmadas y SBOM |
| Sandbox desactivado | `mabbux doctor` comprueba que el sandbox nativo del asistente está activo y lo reporta en CI |

## Implementación

El stack es Node 24 LTS con TypeScript, ejecutado directamente por Node (type stripping) y empaquetado con esbuild en un solo fichero. Se eligió Node por neutralidad entre asistentes y por su aceptación en entornos corporativos (decisión 0001). El punto delicado es la latencia de los hooks, que se mide en F0; si no fuera aceptable, solo el ejecutor de hooks se reescribiría en Rust.

**Stack propuesto:**

| Pieza | Elección | Motivo |
| --- | --- | --- |
| Runtime | Node 24 LTS, TypeScript con type stripping | Gobernanza neutral (OpenJS), ciclo LTS predecible, sin compilación en desarrollo |
| Estructura | Monorepo con pnpm: `core`, `cli`, `adapters`, `mcp` | `core` no depende de nadie; el resto depende de `core` |
| Empaquetado | esbuild a un único fichero | Arranque rápido de los hooks y un solo paquete npm, `mabbux` |
| CLI | citty o commander, con @clack/prompts | Subcomandos y asistente de `mabbux init` |
| Configuración | YAML validado con zod, exportando JSON Schema | Errores claros y autocompletado en editores |
| Estado local | `node:sqlite` (o better-sqlite3) con FTS5 | Anclajes, evidencias y búsqueda sin servidor |
| Memoria OKF | unified/remark y gray-matter | Lectura y validación del bundle |
| Anclajes | web-tree-sitter (WASM) | Sin compilación nativa por plataforma |
| Servidor MCP | SDK oficial de TypeScript | Implementación de referencia del protocolo |
| Hooks de git y secretos | lefthook; secretlint, con gitleaks opcional en CI | El mismo check en local y en CI |
| Telemetría | OpenTelemetry JS con exportación diferida en lote | El hook escribe una línea JSON y termina; OTel no penaliza el arranque |
| Distribución | Paquete npm `mabbux`; ejecutable único de Node (SEA) a evaluar en F2 | `npx` para el MVP; instalación sin Node para equipos no JavaScript más adelante |
| Releases | GitHub Actions con trusted publishing y `npm stage publish` | El mantenedor aprueba cada versión con 2FA |
| Tests | `node:test` o vitest, snapshots de adaptadores y benchmark de latencia | Detectar regresiones en lo generado y en el arranque |

**Estructura en el repositorio del usuario:**

```text
AGENTS.md                 # fuente única de instrucciones (generada y editable)
knowledge/                # bundle OKF: la memoria del proyecto
  index.md
  log.md
  decisions/  conventions/  architecture/  integrations/
.mabbux/
  config.yaml             # se comete: asistentes, carriles, política heredada
  mabbux.lock                 # se comete: digests de skills y políticas
  skills/                 # skills instaladas (formato Agent Skills)
  state/                  # NO se comete: evidencias, diario, índices
.claude/  .cursor/  .github/copilot-instructions.md   # adaptadores generados
```

**Configuración mínima:**

```yaml
version: 1
extends: oci://registry.empresa.com/mabbux/policy:2.3.0
assistants: [claude-code, copilot, cursor]
lanes:
  spec_required_paths: ["src/payments/**", "src/auth/**"]
knowledge:
  block_on_stale: [accepted]
  critical_areas: ["src/payments/**"]
telemetry:
  exporter: file            # file | otlp
  capture_content: false
```

**Comandos principales:**

- `mabbux init` · instalación guiada con plan aceptado.
- `mabbux sync` · regenera adaptadores desde `config.yaml` y `mabbux.lock`.
- `mabbux context` · resumen para inicio de sesión.
- `mabbux done --evidence "<cmd>"` · cierre de tarea con evidencia.
- `mabbux verify` · checks del núcleo (hooks, git y CI).
- `mabbux knowledge status|promote|retire` · ciclo de vida de la memoria.
- `mabbux skill add|publish|verify` · skills firmadas.
- `mabbux policy compile` · política a controles nativos.
- `mabbux eval run|ab` · evals y comparación con y sin harness.
- `mabbux report` · informe de evidencias.
- `mabbux doctor` · diagnóstico de la instalación.

## Roadmap

El MVP (F1) se limita a un desarrollador, dos asistentes y el núcleo local; todo lo de equipo y organización espera a que el A/B demuestre valor.

&#91;embedded content: roadmap · 4 fases, 3 gates\]

La fase F0 sirve para matar la idea barata si los anclajes no son fiables o los hooks resultan lentos. El hito de salida de F3 es el primer cliente regulado usando el informe de evidencias.

## Riesgos y decisiones abiertas

El mayor riesgo es que los propios asistentes absorban la capa de gobierno; la defensa es ser la capa común entre asistentes y la fuente de evidencia, no competir con sus controles nativos.

| Riesgo | Mitigación |
| --- | --- |
| Los asistentes integran memoria y gobierno nativos | Integrarse con ellos (`mabbux policy compile`) y centrarse en lo multiasistente: CI, evidencia y auditoría |
| Fricción que lleva a desactivar el harness | Carriles proporcionales y tasa de desactivación como métrica de producto |
| Anclajes costosos en monorepos o lenguajes sin gramática | Cálculo incremental por diff y respaldo por rango de líneas y hash de fichero |
| Estándares inmaduros (OKF v0.x, OTel GenAI en Development) | Capa de adaptación con versión declarada en `config.yaml` |
| Dos públicos con necesidades distintas | Modelo open core: núcleo local abierto, capa de organización comercial |

**Decisiones abiertas:**

- [ ] Nombre del producto y licencia del núcleo.
- [ ] Validar en F0 que la latencia de los hooks en Node es aceptable; si no, ejecutor de hooks en Rust.
- [ ] Primer par de asistentes del MVP (propuesta: Claude Code y GitHub Copilot).
- [ ] Registro de skills en la infraestructura del cliente o como servicio.
- [ ] Integración con Spec Kit u OpenSpec para el carril de especificación.
- [ ] Encaje con la iniciativa de conocimiento corporativo preparado para IA.

## Fuentes

- [rsc-harness (web)](https://ericrisco.github.io/rsc-harness/) y [repositorio](https://github.com/ericrisco/rsc-harness)
- [OpenAI: Agentic AI Foundation y AGENTS.md](https://openai.com/index/agentic-ai-foundation/)
- [agentskills.io](https://agentskills.io/) y [análisis de Agent Skills (Rubin Lake)](https://rubinlake.com/en/technology-radar/developer-ai-and-delivery/agent-skills)
- [MCP 2026 roadmap y especificación de julio (Obot)](https://obot.ai/blog/mcp-is-growing-up-the-2026-roadmap-takes-shape/)
- [Open Knowledge Format](https://agentic-ai.readthedocs.io/en/latest/Standards/open-knowledge-format/)
- [Thoughtworks Radar: progressive context disclosure](https://www.thoughtworks.com/radar/techniques/progressive-context-disclosure)
- [Herramientas de spec-driven development (SSOJet)](https://ssojet.com/blog/best-spec-driven-development-tools) y [Spec Kit frente a Kiro](https://codemyspec.com/blog/spec-kit-vs-kiro)
- [OpenTelemetry GenAI conventions (TrueFoundry)](https://www.truefoundry.com/blog/opentelemetry-genai-semantic-conventions)
- [OWASP y prompt injection en agentes (Help Net Security)](https://www.helpnetsecurity.com/2026/06/11/owasp-prompt-injection-ai-security-failures) y [seguridad de agentes de código (Morph)](https://www.morphllm.com/ai-coding-agent-security)
- [Claude Code: plugins y managed settings](https://code.claude.com/docs/en/plugins/org) y [marketplaces privados](https://dev.to/michaeltuszynski/your-claude-plugin-marketplace-needs-more-than-a-git-repo-5631)
- [AI Act: acuerdo del Omnibus (Gibson Dunn)](https://www.gibsondunn.com/eu-ai-act-omnibus-agreement-postponed-high-risk-deadlines-and-other-key-changes/)
