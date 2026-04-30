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

## 2026-04-29 — Una `skill` (`arch-guard`) defiende la arquitectura mejor que un README

**Contexto:** primer modelo de datos real de agentdeck. Sergio pidió que el schema fuese profesional desde el principio: nada de JSON encubriendo lo que debería ser una tabla, todos los enums tipados con `pgEnum`, y una arquitectura de carpetas que escale (features verticales, capas con reglas de import, server-only declarado, Zod en todos los inputs).

**Lo que normalmente pasa:** escribes esas reglas en un `CLAUDE.md` o `AGENTS.md` enorme, el agente las lee la primera vez, y a las dos semanas alguien (humano o LLM) mete un `any` "solo por ahora" o un campo `jsonb` "para iterar rápido". La arquitectura se erosiona en silencio.

**Lo que decidí:** las reglas de arquitectura y tipado no viven en `AGENTS.md` — viven en una skill propia, `arch-guard`, en `.claude/skills/arch-guard/SKILL.md`. La skill tiene un `description` con triggers explícitos ("use proactively whenever editing or creating .ts/.tsx files") para que el harness la cargue **automáticamente** cada vez que se toca código del repo. El `AGENTS.md` solo apunta a ella.

**Por qué es mejor que un README aspiracional:**

1. **Activación automática.** No depende de que el agente "se acuerde" de leer reglas. La skill se invoca por el contexto del trabajo (estás editando `.ts` → entra `arch-guard`).
2. **Checklist al cierre.** La skill incluye una "pre-edit checklist" que el agente repasa **antes** de cerrar el cambio: ¿está en la capa correcta? ¿respeta los imports entre capas? ¿tiene `import 'server-only'` si toca DB? ¿pasa la regla `no-explicit-any`?
3. **Reglas concretas, no aspiraciones.** En vez de "código limpio" (vacío), reglas verificables: "cero `any`, cero `!` non-null, types de DB derivados de Drizzle, jsonb solo para input heterogéneo".
4. **Defensa activa, no pasiva.** Si una petición empuja a romper la regla ("mete los tools como JSON, ya"), la skill obliga al agente a parar y proponer la alternativa correcta (en este caso, una tabla `scan_agent_tools` separada).

**Combo con ESLint:** las reglas de tipado (`no-explicit-any`, `no-non-null-assertion`, `consistent-type-imports`) están **también** en `eslint.config.mjs` con nivel `error`. La skill define la intención, ESLint la enforza en CI. Doble red.

**Resultado del primer uso:** con `arch-guard` activa desde el primer minuto, el modelo de datos quedó con 11 tablas normalizadas, 7 `pgEnum`, separación correcta de Server Components vs Client Components, env vars validadas con Zod en `lib/env.ts`, y cero `any` en todo el repo. Sin que tuviera que pedírselo dos veces.

**La regla que aplico ahora:**

- Las **políticas que se aplican siempre** (estilo, tipado, layout) van en una skill con triggers automáticos, no en un documento.
- `AGENTS.md` queda como entrada para el contexto de **producto** (qué construyes, por qué). Las reglas de **cómo construirlo** viven en skills.
- Cualquier regla "verificable" tiene un check correspondiente en ESLint o tsconfig. Si no se puede verificar, probablemente es vaga y hay que reescribirla.

**Coste:** 1.5h de diseño y refactor del scaffold a la estructura nueva. Ahorrado a partir de aquí: cada review que no tendré que hacer porque la regla ya estaba puesta antes de escribir el código.

**Post:** [Semana 2 — El equipo de agentes empieza a tomar forma (próximamente)](https://sergiodima.dev/multiagente) _(publicación martes 12 mayo 2026)_

---

## 2026-04-30 — Cómo escanear un repo de GitHub sin clonar nada

**Contexto:** primer scanner de agentdeck. Hay que leer un repo público (URL de GitHub), extraer su `.claude/` y meter el resultado en Postgres. Mi reflejo inicial: instalar `simple-git`, clonar a `/tmp`, leer del filesystem, borrar al acabar. Lo descarté.

**Por qué la solución obvia es mala aquí:**

1. **Vercel no te deja escribir filesystem persistente.** Sí, `/tmp` está disponible (~512MB), pero solo durante la invocación. Cualquier cosa que pase de unos segundos o invoque varias veces es frágil.
2. **`git clone` es un binario.** En Vercel no está. En un container de producción tampoco salvo que lo metas tú.
3. **Solo necesito unos KB del repo (`.claude/` y los `*.md` raíz).** Bajarme el repo entero a disco para usar el 1% es pereza arquitectónica.

**Lo que hice:** descargar el **tarball** del repo desde GitHub y procesarlo en streaming, sin tocar disco.

```ts
const url = `https://codeload.github.com/${owner}/${repo}/tar.gz/refs/heads/${branch}`;
const response = await fetch(url);
const nodeStream = Readable.fromWeb(response.body);
nodeStream.pipe(createGunzip()).pipe(tarExtract);
```

`codeload.github.com` es el subdominio que usa GitHub para servir archivos. No requiere auth para repos públicos, no tiene rate limits agresivos, y devuelve un `.tar.gz`. Lo combino con `tar-stream` (Node, no usa filesystem) y `zlib.createGunzip()` (built-in Node) para descomprimir y parsear entry por entry **mientras el stream llega**. En el `'entry'` del tar, decido si el archivo me interesa (`.claude/...` o `CLAUDE.md` o `AGENTS.md`) y si no, hago `stream.resume()` para descartarlo sin acumularlo en memoria.

**Resultado:** un repo público se escanea en memoria, en uno o dos segundos, sin filesystem, sin git binary, sin cuotas. Funciona igual en local que en Vercel.

**Tres detalles que añadí al diseño y que merecen la pena:**

1. **Fallback `main` → `master` automático.** Algunos repos viejos siguen en `master`. Si el primer fetch a `refs/heads/main` falla, lo intento con `master` antes de devolver error. Cero coste, mucha menos fricción de UX.
2. **Mismo flujo para zip upload.** El usuario puede subir un `.zip` del proyecto y el resto del pipeline (parseClaudeDirectory → persist) es idéntico. Esto solo es posible porque defino una `VirtualFile = { path, bytes: Uint8Array }` como tipo común, y todos los métodos de ingesta producen `VirtualFile[]`. Cambias el origen, no cambias el resto.
3. **Límites de tamaño explícitos.** 10MB por zip, 2MB por archivo. Un setup multi-agente típico ocupa <100KB; cualquier cosa por encima es ruido o ataque. Con `MAX_FILE_BYTES` chequeado dentro del `'data'` handler del tar, nunca se acumula un archivo grande en memoria — se aborta el chunk en cuanto cruza el umbral.

**La regla que aplico ahora:**

- Antes de instalar una librería que opera sobre filesystem (`fs-extra`, `simple-git`, etc.), preguntarme si **realmente necesito el repo en disco** o solo unos archivos. En el 90% de los casos, un fetch + parser de stream es más simple, más rápido y compatible con cualquier runtime.
- Cuando un servicio (GitHub) ya te da el archivo empaquetado, úsalo. No reproduzcas su trabajo con un wrapper de git.

**Coste:** 0 de fricción una vez que pillé la idea. Lo que evité: una dependencia de `simple-git`, el setup de un binario `git` en el runtime, y horas de debugging cuando aquello explote en producción.

**Post:** [Semana 2 — agentdeck ya escanea repos en vivo (próximamente)](https://sergiodima.dev/multiagente) _(publicación martes 12 mayo 2026)_

---
