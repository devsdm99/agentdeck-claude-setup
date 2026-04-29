# Lessons

Errores y descubrimientos reales del proceso de construir agentdeck con un setup multi-agente Claude Code. Cada entrada se publica con el post correspondiente de la newsletter [Multiagente](https://sergiodima.dev/multiagente).

El propósito es no repetir errores — y, si tú estás leyendo esto antes de cometerlos, ahorrarte el tiempo.

---

## 2026-04-29 — Next.js 15 ya viene con su propio `CLAUDE.md` (y no es lo que esperaba)

**Contexto:** primera sesión real de desarrollo de agentdeck. Plan: `npx create-next-app@latest` y luego escribir el `CLAUDE.md` desde cero como punto de partida del setup multi-agente.

**Lo que descubrí:** `create-next-app` (versión 2026-04) ya genera **dos archivos automáticamente**:

```
CLAUDE.md       → 1 línea: @AGENTS.md
AGENTS.md       → bloque protegido con instrucción para agentes
```

El `AGENTS.md` viene con un mensaje literal del equipo de Next.js dirigido a agentes IA:

> *"This is NOT the Next.js you know — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code."*

Es un **delimitador `<!-- BEGIN:nextjs-agent-rules -->` ... `<!-- END -->`** que el equipo de Next.js usa para mantener su instrucción intacta entre actualizaciones.

**Por qué importa:**

1. Yo iba a escribir un `CLAUDE.md` de 200 líneas desde cero. Habría duplicado/contradicho lo que el propio framework ya pone.
2. El patrón nativo `CLAUDE.md → @AGENTS.md` es interesante: `AGENTS.md` es **convención emergente cross-tool** (Cursor, Aider, Claude Code, Copilot Workspace todos lo leen). `CLAUDE.md` es específico de Claude Code. Importar uno en el otro evita escribir las reglas dos veces.
3. **No se debe tocar el bloque `BEGIN:nextjs-agent-rules`** — es la zona donde el framework actualizará sus reglas en futuras versiones.

**La regla que aplico ahora:**

- En proyectos Next.js 15+: respetar el bloque `BEGIN:nextjs-agent-rules` y añadir mi contenido **debajo** del `<!-- END -->`, separado por `---`.
- `CLAUDE.md` se queda como simple `@AGENTS.md`. Toda la instrucción del proyecto vive en `AGENTS.md`.
- Al actualizar Next.js (`npm install next@latest`), si el bloque cambia, mi contenido debajo no se ve afectado.

**Coste:** 0 horas — pillado a tiempo. La alternativa habría sido un `CLAUDE.md` de 200 líneas que en la próxima actualización de Next.js entraría en conflicto con el bloque oficial.

**Post:** [Semana 1 — El primer `CLAUDE.md` de agentdeck](https://sergiodima.dev/multiagente) _(publicación martes 5 mayo 2026)_

---
