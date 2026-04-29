# Changelog

Histórico de cambios del setup multi-agente que construye agentdeck. Cada entrada se publica con el post correspondiente de la newsletter [Multiagente](https://sergiodima.dev/multiagente).

---

## [Semana 0] — 2026-04-28

### Estado inicial

- Estructura del repo creada (`.claude/agents/`, `.claude/skills/`, `.claude/hooks/`)
- README con hipótesis de los 6 roles para el stack de agentdeck (Next.js 15 + React + Tailwind + Supabase + Drizzle + Anthropic SDK + Stripe)
- `LESSONS.md` vacío (aún no hay lecciones)
- **No hay `CLAUDE.md`** todavía — la versión real llega en la semana 1
- **No hay agentes activos** — la primera definición llega en la semana 3
- Repo de la app (agentdeck) creado en paralelo: [github.com/devsdm99/agentdeck](https://github.com/devsdm99/agentdeck)

### Post asociado

[Voy a construir un micro-SaaS en público con un equipo de agentes Claude Code. Esta es la semana 0 de agentdeck.](https://sergiodima.dev/blog/es/agentdeck-construyo-un-micro-saas-en-publico)

---

_Próxima entrada: semana 1 — `CLAUDE.md` real para agentdeck_

---

## [Semana 1 — work in progress] — desde 2026-04-29

### Cambios en curso (no publicados aún)

- App `agentdeck` inicializada con Next.js 15 + TypeScript + Tailwind v4 + shadcn/ui (estilo new-york)
- Stack DB instalado: Drizzle ORM + `postgres` driver + `@supabase/supabase-js` + `@supabase/ssr`
- Estructura `src/lib/db/` con `schema.ts` (tabla `profiles` mínima) y `index.ts` (cliente Drizzle)
- Estructura `src/lib/supabase/` con `client.ts` y `server.ts` (patrón oficial `@supabase/ssr`)
- `drizzle.config.ts` apuntando a `DATABASE_URL`
- `.env.example` con las variables necesarias (Supabase + Drizzle + Anthropic)
- **Decisión clave (LESSON #1):** respetar el `AGENTS.md` que genera Next.js 15 y añadir el contenido del proyecto **debajo** del bloque protegido `BEGIN:nextjs-agent-rules`. `CLAUDE.md` se mantiene como simple `@AGENTS.md`.
- ✅ Cuenta Supabase + proyecto `agentdeck` creados (Europe, RLS automático ON, expose tables OFF). Tras un incidente de seguridad (claves pegadas en chat), proyecto recreado con keys nuevas.
- ✅ `.env.local` rellenado (NUNCA se commitea).
- ✅ Primera tabla `profiles` creada en Supabase vía `npx drizzle-kit push`.
- **LESSON #2:** Supabase direct connection (puerto 5432) es IPv6-only desde Q4 2024. Desde redes sin IPv6 público (la mayoría de ISPs domésticos), `drizzle-kit` se cuelga sin error. Solución: usar siempre `DATABASE_URL` del pooler (puerto 6543, IPv4-routable), tanto para runtime como para migraciones. `drizzle.config.ts` ajustado.

### Post asociado

[Semana 1 — El primer `CLAUDE.md` de agentdeck (próximamente)](https://sergiodima.dev/multiagente)
