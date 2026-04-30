# agentdeck-claude-setup

> El cerebro multi-agente que construye **[agentdeck](https://github.com/devsdm99/agentdeck)** — un micro-SaaS público para devs Claude Code.
> Build-in-public. La app vive en su propio repo (también público). Aquí vive el setup que escribe ese código.

---

## Qué es esto

Soy [Sergio Díaz](https://sergiodima.dev) y estoy construyendo, en público y desde cero, un setup multi-agente con [Claude Code](https://claude.com/claude-code) — y con ese setup estoy construyendo agentdeck, una herramienta para visualizar el setup multi-agente de cualquier repo de Claude Code.

Este repo contiene **solo el setup** — `CLAUDE.md`, subagentes, skills, hooks — sin código de la app. La app vive en [github.com/devsdm99/agentdeck](https://github.com/devsdm99/agentdeck).

Cada martes publico un post en mi newsletter [Multiagente](https://sergiodima.dev/multiagente) explicando qué he cambiado, por qué, y qué he aprendido. Cada post enlaza al commit exacto de la semana.

## Por qué público

Dos razones:

1. **Auditabilidad.** Si alguien lee la newsletter y quiere comprobar que el setup que describo realmente existe y funciona, puede.
2. **Honestidad.** Montarlo a puerta cerrada es una excusa para tomar atajos. Si está abierto, no me queda otra que hacerlo bien.

Lee el [post de la semana 0](https://sergiodima.dev/blog/es/agentdeck-construyo-un-micro-saas-en-publico) si quieres el contexto completo del proyecto.

## Estado actual

🟡 **Semana 2.** Backend del scanner de agentdeck ya operativo (parsea repos públicos y zip uploads en memoria, persiste en Postgres). Aún sin agentes propios definidos en este repo — la disciplina arquitectónica del proyecto vive como **skill** (`arch-guard`) en el repo de la app, que es donde más sentido tiene aplicarse.

Lecciones publicadas (ver [`LESSONS.md`](LESSONS.md)):

- **#1** — Next.js 15 ya viene con su propio `CLAUDE.md` y `AGENTS.md`. Cómo convivir con el bloque protegido del framework.
- **#2** — Supabase direct connection es IPv6-only desde Q4 2024. Por qué `drizzle-kit` se cuelga sin error y cómo evitarlo.
- **#3** — Una skill con triggers automáticos (`arch-guard`) defiende la arquitectura mejor que un README aspiracional.
- **#4** — Cómo escanear un repo de GitHub sin `git clone`: codeload tarball + `tar-stream` en streaming.

## Estructura

```
.claude/
├── agents/      ← definiciones de subagentes (vacío todavía — llegan en semana 3)
├── skills/      ← skills personalizadas (la skill arch-guard vive en el repo de la app)
└── hooks/       ← hooks del lifecycle (vacío todavía — llegan cuando haya orquestación real)

CLAUDE.md        ← contexto principal (próximamente; hoy las reglas viven en AGENTS.md de la app)
CHANGELOG.md     ← qué cambió cada semana, con link al post correspondiente
LESSONS.md       ← errores encontrados, soluciones aplicadas, por qué no se repiten
```

## Stack del proyecto que este setup construye

agentdeck (la app) está hecha en:

- **Framework:** Next.js 15+ (App Router) + TypeScript estricto
- **UI:** Tailwind CSS v4 + shadcn/ui
- **DB:** Postgres en Supabase + Drizzle ORM
- **Auth:** Supabase Auth (`@supabase/ssr`)
- **Hosting:** Vercel
- **Pagos (semana 6+):** Stripe
- **AI calls:** Anthropic SDK donde aporte (scoring, summaries)

Los detalles técnicos vivos están en `AGENTS.md` del [repo de la app](https://github.com/devsdm99/agentdeck/blob/main/AGENTS.md).

## Hipótesis de roles del equipo de agentes

Para una stack **Next.js 15 + React + Tailwind + Supabase + Drizzle + Anthropic SDK + Stripe** la hipótesis inicial es de 6 roles. Hipótesis — los agentes solo se materializan cuando hay trabajo repetitivo real para ellos:

| # | Rol | Para qué |
|---|---|---|
| 1 | **Architect** | Decisiones de alto nivel, fronteras Server/Client, layout entre features |
| 2 | **View Builder** | Pages + Server/Client Components, estilado con Tailwind, shadcn |
| 3 | **State & Data** | Server Actions, queries Drizzle, validación Zod |
| 4 | **Backend & Integrations** | Webhooks (Stripe, GitHub), endpoints API, parsers, AI calls |
| 5 | **Test Writer** | Tests unitarios + integración, Playwright si hace falta |
| 6 | **QA Reviewer** | Lectura crítica antes de commit, lint/typecheck/build |

La regla del proyecto: **no se crea un agente hasta que hay un trabajo repetitivo concreto para él.** El primero (probablemente View Builder) llega en la semana 3, no antes.

## Aviso

Este es el setup **mío** para **mi** proyecto. No es un kit reutilizable (todavía). Si funciona y se estabiliza, llegará un kit empaquetado más adelante — pero no antes de tener métricas reales.

Si quieres seguir el viaje, suscríbete a la newsletter:
**👉 [sergiodima.dev/multiagente](https://sergiodima.dev/multiagente)**
