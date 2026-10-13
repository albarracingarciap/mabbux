---
type: principles
title: Principios del proyecto mabbux
description: Las siete reglas que guían cualquier decisión de diseño o implementación; en caso de conflicto gana la de número más bajo
tags: [principios, constitucion, gobierno]
timestamp: 2026-10-12T12:00:00Z
x-mabbux:
  status: accepted
  owner: "@albarracingarciap"
  provenance:
    author: human
---

# Principios del proyecto

Estas reglas no se negocian en una tarea concreta. Si una propuesta choca con alguna, se señala antes de implementarla. Cuando dos principios entren en conflicto, gana el de número más bajo.

1. **Lo que importa se comprueba fuera del modelo.**
   Toda regla vinculante tiene un check determinista en git o en CI. Las instrucciones al agente solo ayudan a cumplirla antes; nunca son la garantía.

2. **Estándares antes que formatos propios.**
   Se usan AGENTS.md, Agent Skills, MCP, Open Knowledge Format y OpenTelemetry. Lo propio se limita a extensiones en el frontmatter (`x-mabbux`), que otros consumidores pueden ignorar.

3. **Todo conocimiento tiene procedencia.**
   Cada documento de `knowledge/` dice quién o qué lo escribió, con qué evidencia, a qué código está anclado y hasta cuándo es válido. Lo que escribe un agente nace como `proposed` y solo una persona lo pasa a `accepted`.

4. **Contexto mínimo y medido.**
   Nada se carga siempre si puede cargarse bajo demanda. AGENTS.md apunta al conocimiento en lugar de repetirlo, y no se añade una skill hasta que una tarea se ha repetido tres veces.

5. **Fricción proporcional al riesgo.**
   Silencio para lo trivial, evidencia para los cambios normales, spec aprobada para lo grande o sensible. Un harness que molesta acaba desactivado.

6. **Local primero, organización opcional.**
   Todo funciona en un portátil sin cuenta ni red. La capa de organización se añade sin cambiar el flujo del desarrollador.

7. **Nada se escribe sin plan aceptado.**
   Antes de crear o modificar ficheros relevantes, el agente explica qué va a hacer y espera aprobación.
