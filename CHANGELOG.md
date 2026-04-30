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
- **Refactor de arquitectura completo (29 abril, sesión tarde):**
  - Estructura de carpetas profesional: `app/` · `components/{ui,layout,features}/` · `features/<slice>/{actions,queries,schemas,service}.ts` · `lib/{db,supabase,env,utils}` · `shared/{types,constants,errors}` · `utils/` · `hooks/`.
  - Schema dividido por dominio en `src/lib/db/schema/`: `enums.ts`, `profiles.ts`, `repos.ts`, `scans.ts`, `relations.ts`, `index.ts` (re-export).
  - **Modelo de datos completo (11 tablas + 7 enums tipados):** `profiles`, `repos`, `repo_sources`, `scans`, `scan_stats`, `scan_files`, `scan_agents`, `scan_agent_tools`, `scan_skills`, `scan_skill_triggers`, `scan_hooks`. Todas las listas estructuradas son tablas hijas (cero JSON-as-table). `jsonb` reservado solo para `frontmatter` (input heterogéneo del usuario).
  - `repo_sources` permite **múltiples métodos de ingesta** por repo (URL pública, zip upload, GitHub OAuth futuro, local). Cada `scan` referencia explícitamente la `source` que lo originó.
  - Validación de `process.env` con Zod en `src/lib/env.ts` (server + client schemas separados).
  - **`import 'server-only'`** en módulos que tocan DB o service-role keys (`lib/db`, `lib/supabase/server`, futuras `features/*/queries|actions`).
  - `lib/utils.ts` (cn de shadcn) intacto por convención del framework de UI.
  - Tabla `profiles` previa **dropeada (estaba vacía)** y recreada con la migración nueva. Migración aplicada limpiamente: archivo único `0000_acoustic_titania.sql` con 7 enums + 11 tablas + 14 FKs + 12 índices (5 únicos para integridad).
- **`.claude/skills/arch-guard`:** primera skill del repo de la app, defensiva. Triggers automáticos en cualquier `.ts`/`.tsx`. Codifica las reglas de layer/import/typing y obliga a un checklist pre-cierre. La arquitectura deja de ser un README aspiracional.
- **ESLint 9 flat config** con plugin oficial de Next + `typescript-eslint`. Reglas a `error`: `no-explicit-any`, `no-non-null-assertion`, `consistent-type-imports`. Scripts `lint` y `typecheck` añadidos. `npm run build`, `lint` y `typecheck` pasan en verde.
- **LESSON #3:** una skill con triggers automáticos defiende la arquitectura mejor que un documento aspiracional. Ver `LESSONS.md`.

### Post asociado

[Semana 1 — El primer `CLAUDE.md` de agentdeck (programado para martes 5 mayo)](https://sergiodima.dev/multiagente)

---

## [Semana 2 — work in progress] — desde 2026-04-30

### Cambios en curso (no publicados aún)

- **Scanner básico end-to-end (jueves 30 abril):**
  - `features/scans/` con vertical slice completo: `service.ts` (parseClaudeDirectory pura), `parsers/{frontmatter,agent,skill,hook}.ts`, `ingest/{from-zip,from-url}.ts`, `persist.ts` (transacción Drizzle), `actions.ts`, `schemas.ts`.
  - **Ingesta sin filesystem ni git binary:** URL pública de GitHub se baja como `tar.gz` desde `codeload.github.com`, se descomprime con `zlib.createGunzip()` y se parsea con `tar-stream` en streaming. Solo se retienen archivos en `.claude/` + `CLAUDE.md` + `AGENTS.md`. Fallback `main → master` automático.
  - **Mismo pipeline para zip upload:** `JSZip` produce los mismos `VirtualFile[]` que el tarball, así que el resto del pipeline es idéntico. Cambias el origen, no cambias el parser ni la persistencia.
  - **Parsers tipados:** frontmatter YAML con `js-yaml`. Agents (con `tools[]` normalizado a tabla hija). Skills (con `triggers[]` opcional). Hooks desde `settings.json` con normalización de nombres (`PreToolUse → pre_tool_use`).
  - **Persistencia transaccional:** un scan crea N archivos + N agents (+ tools) + N skills (+ triggers) + N hooks en una sola transacción. Cero JSON-as-table en runtime. `sha256Hex` por archivo para diff futuro.
  - **API routes** `POST /api/scans/from-url` (JSON) y `POST /api/scans/from-zip` (multipart). Auth temporal con `SCANNER_BOOTSTRAP_TOKEN` — se sustituye por Supabase Auth la semana que viene (viernes 1 mayo).
  - **Dogfood real:** scanner verificado contra el propio repo `agentdeck` (1 skill `arch-guard` detectada con frontmatter completo) y contra un zip sintético (1 agent con 3 tools, 1 skill, 2 hooks con event normalizado). Idempotencia confirmada: 3 scans del mismo repo, 3 rows distintos sin sobrescribir.
- **LESSON #4:** cómo escanear un repo de GitHub sin clonar nada (codeload + tar-stream en memoria). Ver `LESSONS.md`.
- **Bug pillado en runtime:** `ANTHROPIC_API_KEY=""` (var presente pero vacía) hacía explotar la validación Zod en `lib/env.ts`. Fix: helper `optional()` que mapea `""` → `undefined` antes de la validación. Aplicado a todas las variables opcionales del schema.
- **Auth completo (jueves 30 abril, sesión tarde — simulando viernes 1):**
  - Middleware Supabase que refresca cookie de sesión en cada request y redirige rutas privadas a `/login`. Las API routes `/api/scans/*` quedan exentas: devuelven 401 JSON.
  - `features/auth/` slice completo: `schemas.ts` Zod, `queries.ts` (`getUser`/`requireUser` server-only), `actions.ts` (`signupAction`, `loginAction`, `logoutAction` como Server Actions con form state tipado).
  - Páginas `/login` y `/signup` separadas con `AuthForm` Client Component compartido (useActionState + pending state nativo de React 19). Layout `(app)/` con `requireUser()` y header con email + logout. Home rediseñada con CTAs según haya o no sesión.
  - Trigger Postgres `handle_new_user` (`auth.users → public.profiles`) como migración manual `drizzle/manual/0001_handle_new_user_trigger.sql` + script `apply-trigger.mjs`.
  - API routes `/api/scans/*` ahora usan `requireUser()` y verifican `repo.userId === user.id` (403 si no). `SCANNER_BOOTSTRAP_TOKEN` eliminado del schema env y de `.env.example`.
  - shadcn primitives añadidos: `input`, `label`, `card`. Helpers en `scripts/dev/`: `apply-trigger`, `audit-auth-users`, `db-ping`, `show-all`.
  - Verificado end-to-end con email real (`djsdm99@gmail.com`): signup → trigger crea profile → dashboard cargando con sesión activa.
- **LESSON #5:** `email rate limit exceeded` en Supabase no significa lo que parece. Ver `LESSONS.md`.

### Post asociado

[Semana 2 — agentdeck ya escanea repos en vivo (programado para martes 12 mayo)](https://sergiodima.dev/multiagente)
