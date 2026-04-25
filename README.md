# liftiq-claude-setup

> El cerebro multi-agente que ayuda a escribir **LiftIQ — Barbell Tracker** (SwiftUI, iOS).
> Build-in-public. La app es privada. El setup que escribe el código vive aquí.

---

## Qué es esto

Soy [Sergio Díaz](https://sergiodima.dev) y estoy montando, en público y desde cero, un setup multi-agente con [Claude Code](https://claude.com/claude-code) para una app SwiftUI real (LiftIQ).

Este repo contiene **solo el setup** — `CLAUDE.md`, subagentes, skills y hooks — sin código de la app.

Cada semana publico un post en mi newsletter [Multiagente](https://sergiodima.dev/multiagente) explicando qué he cambiado, por qué, y qué he aprendido. Cada post enlaza al commit exacto de la semana.

## Por qué público

Porque montarlo a puerta cerrada es una excusa para tomar atajos. Si está abierto, no me queda otra que hacerlo bien.

Lee el [post de la semana 0](https://sergiodima.dev/blog/es/voy-a-montar-6-agentes-swift-en-publico) si quieres el contexto completo.

## Estado actual

🟡 **Semana 0 — Punto de partida.** Estructura inicial, sin agentes activos todavía. La carne llega en el commit de la semana 1.

## Estructura

```
.claude/
├── agents/     ← definiciones de subagentes
├── skills/     ← skills personalizadas
└── hooks/      ← hooks del lifecycle de Claude Code

CLAUDE.md       ← contexto principal del proyecto LiftIQ (próximamente)
CHANGELOG.md    ← qué cambió cada semana, con link al post
LESSONS.md      ← errores encontrados y por qué no se repiten
```

## Hipótesis de los 6 roles (puede cambiar)

| # | Rol | Para qué |
|---|---|---|
| 1 | **Architect** | Decisiones de alto nivel, refactors, separación de módulos |
| 2 | **View Builder** | Crear y modificar `View` de SwiftUI |
| 3 | **State Manager** | `@Observable`, `@State`, `@Environment`, data flow |
| 4 | **Networking / Persistence** | SwiftData, CloudKit, APIs externas |
| 5 | **Test Writer** | Tests unitarios y de UI con XCTest |
| 6 | **QA Reviewer** | Lectura crítica antes de commit |

## Aviso

Este es el setup **mío** para **mi** app. No es un kit reutilizable (todavía). Si funciona y se estabiliza, llegará un kit empaquetado en la fase 2 — pero no antes de tener métricas reales.

Si quieres seguir el viaje, suscríbete a la newsletter:
**👉 [sergiodima.dev/multiagente](https://sergiodima.dev/multiagente)**

## Licencia

MIT. Cópialo, adáptalo, rómpelo. Solo te pido que si publicas algo basado en este setup, enlaces de vuelta — no por ego, sino porque así otros pueden seguir el hilo de cómo evoluciona.
