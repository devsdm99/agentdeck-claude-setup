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

## 2026-04-29 — Supabase direct connection es IPv6-only desde Q4 2024

**Contexto:** primera conexión de Drizzle a Supabase. Tenía configuradas dos URLs siguiendo la práctica recomendada:
- `DATABASE_URL` (pooler `aws-0-eu-central-1.pooler.supabase.com:6543`) — para runtime
- `DIRECT_URL` (host directo `db.xxx.supabase.co:5432`) — para migraciones DDL

`drizzle.config.ts` apuntaba a `DIRECT_URL` porque la documentación clásica decía que el pooler en transaction mode no maneja bien `CREATE TABLE`.

**Lo que pasó:** `npx drizzle-kit push` se quedó colgado indefinidamente en `Pulling schema from database...`. Ningún error, ningún timeout. Silencio.

**Diagnóstico:**

```bash
$ dig +short A db.xxx.supabase.co
(vacío — no hay IPv4)

$ dig +short AAAA db.xxx.supabase.co
2a05:d018:135e:16a4:5935:6f6d:7d9c:f7db  ← solo IPv6

$ ifconfig en0 | grep inet6
fe80::18e5:a7b1:d4d2:5eaa%en0  ← solo IPv6 link-local, NO global
```

Mi Mac tiene IPv6 link-local (la dirección que se autoasigna en LAN) pero **no IPv6 público** porque mi ISP no me lo da. La mayoría de ISPs domésticos en España, Andorra y LatAm están en la misma situación.

Resultado: la resolución DNS devuelve solo una IP IPv6, mi máquina intenta conectar, no encuentra ruta a IPv6 globally routable, y el cliente Postgres se queda esperando hasta el timeout de TCP del kernel (decenas de minutos).

**Por qué pasa esto:** Supabase migró a finales de 2024 los direct connections (puerto 5432, host `db.xxx.supabase.co`) a IPv6-only. La razón oficial es ahorrar costes de IPs IPv4 que escasean. La solución que ofrecen es usar el pooler (Supavisor) — que sí responde por IPv4.

**La regla que aplico ahora:**

- Para agentdeck (y cualquier proyecto Supabase desde una red sin IPv6 público) → **usar `DATABASE_URL` (pooler) tanto para runtime como para `drizzle-kit`**.
- El pooler de Supabase en realidad **sí soporta DDL** (creación de tablas, alteraciones) en su modo session por defecto. La advertencia clásica "el pooler no soporta DDL" se refería al transaction mode estricto, que ya no es el default del pooler nuevo de Supabase (Supavisor).
- `DIRECT_URL` queda como variable opcional para escenarios con IPv6 (deploys en Vercel, GitHub Actions con runners modernos, etc.).

**Coste:** ~20 minutos de diagnóstico. Sin entender que el síntoma "se cuelga sin error" = problema de routing de red.

**Post:** [Semana 1 — El primer `CLAUDE.md` de agentdeck](https://sergiodima.dev/multiagente) _(publicación martes 5 mayo 2026)_

---
