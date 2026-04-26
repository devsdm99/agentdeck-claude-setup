# agentdeck-claude-setup

> El cerebro multi-agente que construye **[agentdeck](https://github.com/devsdm99/agentdeck)** — un micro-SaaS de pago para devs Claude Code.
> Build-in-public. La app vive en su propio repo (también público). Aquí vive el setup que escribe ese código.

---

## Qué es esto

Soy [Sergio Díaz](https://sergiodima.dev) y estoy construyendo, en público y desde cero, un setup multi-agente con [Claude Code](https://claude.com/claude-code) — y con ese setup estoy construyendo agentdeck, una herramienta para visualizar el setup multi-agente de cualquier repo de Claude Code.

Este repo contiene **solo el setup** — `CLAUDE.md`, subagentes, skills y hooks — sin código de la app. La app vive en [github.com/devsdm99/agentdeck](https://github.com/devsdm99/agentdeck).

Cada martes publico un post en mi newsletter [Multiagente](https://sergiodima.dev/multiagente) explicando qué he cambiado, por qué, y qué he aprendido. Cada post enlaza al commit exacto de la semana en este repo Y en el de la app.

## Por qué público

Dos razones:

1. **Auditabilidad.** Si alguien lee la newsletter y quiere comprobar que el setup que describo realmente existe y funciona, puede.
2. **Honestidad.** Montarlo a puerta cerrada es una excusa para tomar atajos. Si está abierto, no me queda otra que hacerlo bien.

Lee el [post de la semana 0](https://sergiodima.dev/blog/es/agentdeck-construyo-un-micro-saas-en-publico) si quieres el contexto completo del proyecto.

## Estado actual

🟡 **Semana 0 — Punto de partida.** Estructura inicial, sin agentes activos todavía. La carne llega en el commit de la semana 1.

## Estructura

```
.claude/
├── agents/     ← definiciones de subagentes
├── skills/     ← skills personalizadas
└── hooks/      ← hooks del lifecycle de Claude Code

CLAUDE.md       ← contexto principal del proyecto agentdeck (próximamente)
CHANGELOG.md    ← qué cambió cada semana, con link al post
LESSONS.md      ← errores encontrados y por qué no se repiten
```

## Hipótesis de los 6 roles (puede cambiar)

Para una web app **Astro + React islands + Tailwind + Anthropic SDK + Stripe** la hipótesis inicial es:

| # | Rol | Para qué |
|---|---|---|
| 1 | **Architect** | Decisiones de alto nivel, fronteras Astro/React, qué meter en backend |
| 2 | **View Builder** | Componentes Astro y React, estilado con Tailwind |
| 3 | **State Manager** | Estado React (Zustand), data flow cliente ↔ servidor |
| 4 | **Backend & Integrations** | Endpoints, parser markdown, Stripe, Anthropic SDK |
| 5 | **Test Writer** | Tests unitarios + e2e con Playwright |
| 6 | **QA Reviewer** | Lectura crítica antes de commit |

## Aviso

Este es el setup **mío** para **mi** proyecto. No es un kit reutilizable (todavía). Si funciona y se estabiliza, llegará un kit empaquetado más adelante — pero no antes de tener métricas reales.

Si quieres seguir el viaje, suscríbete a la newsletter:
**👉 [sergiodima.dev/multiagente](https://sergiodima.dev/multiagente)**

## Licencia

MIT. Cópialo, adáptalo, rómpelo. Solo te pido que si publicas algo basado en este setup, enlaces de vuelta — no por ego, sino porque así otros pueden seguir el hilo de cómo evoluciona.

Nota: la **app agentdeck** (en su propio repo) usa una licencia distinta (BSL 1.1) — esto es solo el setup multi-agente, que se mantiene libre.
