# Spec 01 — Pantallas visuales del MVP (App Router)

**Estado:** Aprovado
**Dependencias:** Ninguna (primer spec del proyecto)
**Fecha:** 2026-07-05

**Objetivo:** Implementar como páginas reales de Next.js App Router las 5 pantallas visuales de Arcade Vault definidas en `references/templates/` (Biblioteca, Detalle de juego, Reproductor, Login/Registro y Salón de la Fama), replicando su diseño y comportamiento de interfaz sin construir lógica de juego real.

## Alcance

### Dentro del alcance

- 5 rutas reales de Next.js App Router (slugs en inglés):
  - `/` — Biblioteca (catálogo de juegos, buscador, filtro por categoría)
  - `/games/[id]` — Detalle de juego (info, stats, tabla de mejores puntuaciones)
  - `/games/[id]/play` — Reproductor (HUD, pantalla CRT simulada, modal de fin de partida)
  - `/login` — Login / Registro (tabs iniciar sesión / crear cuenta, invitado, botones sociales decorativos)
  - `/hall-of-fame` — Salón de la Fama (podio + tabla de clasificación por juego)
- Navbar compartida (`Nav`) con estado activo por ruta, contador de créditos estático y menú móvil (hamburguesa + panel deslizante).
- Layout raíz con fondo (`av-bg`, `av-noise`), fuentes de Google Fonts vía `<link>` (Press Start 2P, Courier Prime, JetBrains Mono) reemplazando las fuentes Geist actuales, y footer fijo.
- `styles.css` del template importado tal cual como hoja de estilos global (no se traduce a utilidades Tailwind).
- Datos mock (`GAMES`, `CATS`, `PLAYERS`, `seededScores`) portados a un módulo compartido `lib/data.ts`.
- Interactividad puramente de frontend: búsqueda/filtro en biblioteca, tabs en login y salón de la fama, pausa/reanudar y modal de fin de partida en el reproductor, toggle de menú móvil.
- Login falso: guarda un nombre de usuario en `localStorage` (`av_user`); incluye opción "jugar como invitado"; botones de Google/GitHub son decorativos (no funcionales).
- Reproductor simulado exactamente como el template: puntaje que sube solo, "enemigos" animados por CSS, pausa/reanudar, modal de fin con guardado de puntaje en `localStorage` (`av_scores`).

### Fuera del alcance

- Cualquier juego real (lógica, motor, canvas, colisiones). Las 8 pantallas de juego (Bloque Buster, Caída, etc.) sólo existen como entradas de catálogo con reproductor simulado, no como juegos jugables.
- Backend, base de datos o autenticación real — todo el estado de usuario vive en `localStorage` del navegador.
- Que las puntuaciones guardadas alimenten el ranking mostrado en el Salón de la Fama (esa tabla sigue usando datos mock deterministas `seededScores`, igual que el template).
- Audio/sonido, internacionalización, o una auditoría de accesibilidad más allá de lo ya presente en el markup del template.
- Suite de tests (no hay test runner configurado en el proyecto).

## Modelo de datos

### `lib/data.ts` (mock data compartido, portado de `data.jsx`)

```ts
export interface Game {
  id: string;       // slug, ej. "bloque-buster" — usado en /games/[id]
  title: string;
  short: string;
  long: string;
  cat: "ARCADE" | "PUZZLE" | "SHOOTER" | "VERSUS";
  cover: string;     // clase CSS de fondo, ej. "cover-bricks"
  color: "cyan" | "magenta" | "yellow" | "green";
  best: number;
  plays: string;     // ej. "12.4K"
}

export const GAMES: Game[];               // 8 juegos, igual que el template
export const CATS: string[];              // ["TODOS", "ARCADE", "PUZZLE", "SHOOTER", "VERSUS"]
export const PLAYERS: string[];           // nombres para el leaderboard simulado

export interface ScoreRow {
  rank: number;
  name: string;
  score: number;
  date: string;      // "DD/MM/2026"
}

export function seededScores(seed: number, count?: number): ScoreRow[];
```

Las etiquetas de categoría y el contenido (`title`, `short`, `long`) permanecen en español, aunque las rutas usen slugs en inglés.

### Estado de sesión (cliente, sin backend)

Persistido en `localStorage`, mismas claves que el template:

```ts
// clave: "av_user"
type StoredUser = { name: string } | null;

// clave: "av_scores"
type StoredScore = {
  game: string;   // Game.id
  score: number;
  name: string;
  at: number;     // Date.now()
};
type StoredScores = StoredScore[];
```

- `Nav` y la página de reproductor leen `av_user` en el cliente (`useEffect` + `localStorage`) para reflejar sesión iniciada / invitado.
- El modal de fin de partida en `/games/[id]/play` agrega una entrada a `av_scores` al guardar, igual que `handleSaveScore` del template.
- No hay contexto ni store global: cada página/componente cliente que lo necesite lee `localStorage` directamente al montar, ya que no existe un layout de estado compartido tipo SPA (a diferencia del template, aquí la navegación es entre rutas reales).

## Plan de implementación

1. **Capa de datos.** Crear `lib/data.ts` portando desde `data.jsx`: interfaz `Game`, arrays `GAMES`/`CATS`/`PLAYERS`, interfaz `ScoreRow` y función `seededScores`. Sin consumidores todavía; el build sigue pasando.

2. **Estilos y fuentes globales.** Crear `app/arcade.css` con el contenido de `references/templates/styles.css` tal cual; importarlo desde `app/globals.css` (debajo de `@import "tailwindcss"`). Actualizar `app/layout.tsx`: quitar `Geist`/`Geist_Mono`, añadir en el `<head>` los `<link>` de preconnect + stylesheet de Google Fonts (Press Start 2P, Courier Prime, JetBrains Mono), añadir los divs de fondo `av-bg`/`av-noise`, y actualizar `metadata.title` a "Arcade Vault". La app sigue arrancando (con el contenido placeholder de create-next-app encima del nuevo fondo).

3. **Navbar compartida.** Crear `components/Nav.tsx` (client component) portado de `nav.jsx`: estado activo por ruta con `usePathname`, navegación con `next/link`, lectura de `av_user`/logout vía `localStorage`, menú móvil (hamburguesa + panel). Insertarlo junto con el `<footer>` en `app/layout.tsx`. La navbar ya es visible y funcional en cualquier ruta.

4. **Biblioteca (`/`).** Reemplazar `app/page.tsx` con la pantalla portada de `biblioteca.jsx` (`GameCard` + buscador + chips de categoría), usando `GAMES`/`CATS` de `lib/data.ts` y `next/link` hacia `/games/[id]`. La home ya es la biblioteca real y navegable.

5. **Detalle de juego (`/games/[id]`).** Crear `app/games/[id]/page.tsx` portando `detalle.jsx` (info, stats, leaderboard con `seededScores`); `notFound()` si el `id` no existe en `GAMES`; botón "JUGAR AHORA" hacia `/games/[id]/play`. Flujo biblioteca → detalle queda operativo.

6. **Reproductor (`/games/[id]/play`).** Crear `app/games/[id]/play/page.tsx` (client component) portando `reproductor.jsx`: HUD, CRT simulado, ticking de puntaje falso, pausa/reanudar, modal de fin de partida con guardado en `av_scores`, leyendo `av_user` para el nombre por defecto. Flujo completo biblioteca → detalle → jugar → fin de partida queda operativo.

7. **Login / Registro (`/login`).** Crear `app/login/page.tsx` (client component) portando `auth.jsx`: tabs iniciar sesión/crear cuenta, opción invitado, botones sociales decorativos; al enviar guarda `av_user` y redirige a `/`. Login accesible desde el navbar y operativo.

8. **Salón de la Fama (`/hall-of-fame`).** Crear `app/hall-of-fame/page.tsx` portando `salon.jsx`: tabs por juego, podio, tabla de clasificación, fila "tu mejor marca" si hay `av_user`. Las 5 pantallas quedan completas y navegables entre sí.

9. **Verificación final.** Ejecutar `npm run lint` y `npm run build`; revisión manual de las 5 rutas y sus estados (login real/invitado, pausa/fin de partida y guardado de score, filtro/búsqueda en biblioteca, tabs del salón de la fama, menú móvil).

## Criterios de aceptación

- [ ] `npm run build` y `npm run lint` pasan sin errores.
- [ ] `/` muestra la Biblioteca: buscador por nombre filtra la grilla en tiempo real; los chips de categoría (TODOS/ARCADE/PUZZLE/SHOOTER/VERSUS) filtran correctamente; cada `GameCard` navega a `/games/[id]` al hacer clic en la tarjeta o en "JUGAR".
- [ ] `/games/[id]` muestra info del juego (título, descripción, tags, stats) y una tabla de mejores puntuaciones (`seededScores`); un `id` inexistente produce 404 (`notFound()`); el botón "JUGAR AHORA" navega a `/games/[id]/play`; "VOLVER AL VAULT" navega a `/`.
- [ ] `/games/[id]/play` muestra HUD (jugador, puntuación, vidas, nivel), pantalla CRT simulada; el puntaje sube automáticamente mientras no está en pausa; "PAUSA/REANUDAR" detiene y reanuda el ticking; "FIN" abre el modal de fin de partida con el puntaje final; se puede introducir un nombre y guardar el puntaje (persistido en `localStorage` bajo `av_scores`); "JUGAR DE NUEVO" reinicia el estado; "VOLVER AL VAULT" navega al detalle.
- [ ] `/login` alterna entre tabs "INICIAR SESIÓN"/"CREAR CUENTA" (el campo de correo sólo aparece en "CREAR CUENTA"); enviar el formulario guarda un usuario en `localStorage` (`av_user`) y redirige a `/`; "JUGAR COMO INVITADO" navega a `/` sin usuario guardado; los botones de Google/GitHub son visualmente presentes pero no ejecutan ninguna acción real.
- [ ] `/hall-of-fame` muestra tabs por juego, podio (2º/1º/3º) y tabla de clasificación (`seededScores`); si hay un usuario logueado, se muestra la fila "tu mejor marca"; si no, no se muestra.
- [ ] El navbar (`Nav`) resalta la ruta activa (Biblioteca activo también en detalle/reproductor, según el patrón del template), muestra "Iniciar Sesión" o el nombre del usuario + cerrar sesión según haya o no `av_user`, y el menú móvil (hamburguesa) abre/cierra el panel lateral en viewports pequeños.
- [ ] El fondo (`av-bg`, `av-noise`), las fuentes (Press Start 2P, Courier Prime, JetBrains Mono) y los estilos de `styles.css` se ven aplicados igual que en `references/templates/Arcade Vault.html` al abrir cada ruta.
- [ ] Recargar la página (`F5`) en cualquier ruta mantiene la sesión de usuario si se había iniciado sesión (persistencia real vía `localStorage`, no simulada en memoria).
- [ ] Ninguna pantalla implementa lógica de juego real (colisiones, físicas, input de control): el "juego" del reproductor es únicamente el mock visual descrito arriba.

## Decisiones tomadas y descartadas

- **Routing real de Next.js App Router** en vez del router SPA por hash del template. *Por qué:* es el patrón nativo de App Router y evita mantener un router casero. *Descartado:* replicar `location.hash` + estado único `route` tal cual `app.jsx`.

- **Slugs de ruta en inglés** (`/games/[id]`, `/hall-of-fame`, `/login`), manteniendo todo el copy visible en español. *Por qué:* decisión explícita del usuario. *Descartado:* slugs en español (`/juegos/[id]`, `/salon-de-la-fama`).

- **`styles.css` del template importado tal cual** como hoja global (`app/arcade.css`, importado desde `globals.css`), sin traducirlo a utilidades/tema de Tailwind. *Por qué:* preserva fidelidad visual (glow, scanlines, efecto CRT) y evita el riesgo/esfuerzo de reescribir 950 líneas de CSS a mano. *Descartado:* migrar el diseño a `@theme inline` y clases utilitarias de Tailwind v4.

- **Fuentes vía `<link>` de Google Fonts** (Press Start 2P, Courier Prime, JetBrains Mono), no vía `next/font/google`. *Por qué:* decisión explícita del usuario para este spec; el CSS del template ya referencia los nombres de fuente literalmente, así que no hace falta cablear variables. Esto es una **excepción puntual** a la convención de fuentes descrita en `CLAUDE.md` (que indica `next/font/google` para Geist). *Descartado:* reemplazar Geist por Press Start 2P/etc. usando `next/font/google`.

- **Reproductor (`/games/[id]/play`) replica el mock del template tal cual** (puntaje que sube solo, "enemigos" animados por CSS, pausa/reanudar, modal de fin). *Por qué:* ese mock ya era solo un placeholder visual en el template, no lógica de juego real, así que replicarlo no viola "no implementar ningún juego". *Descartado:* la opción B — quitar el ticking automático y dejar el HUD/CRT estáticos.

- **Persistencia real en `localStorage`** para sesión falsa (`av_user`) y puntuaciones guardadas (`av_scores`), igual que el template. *Por qué:* decisión explícita del usuario, para que el MVP se sienta "vivo" entre recargas aunque no haya backend. *Descartado:* no persistir nada y resetear todo al recargar.

- **El Salón de la Fama no lee de `av_scores`**; su tabla sigue usando `seededScores` (datos mock deterministas), igual que el template. *Por qué:* conectar puntuaciones guardadas al ranking real requeriría un backend, fuera de alcance de este spec. *Descartado:* mezclar `av_scores` guardados en el ranking mostrado.

- **Sin Context/store global de sesión**; cada componente cliente que lo necesita (`Nav`, reproductor, salón de la fama) lee `localStorage` directamente al montar. *Por qué:* al usar rutas reales de Next.js (no SPA), ya no existe el componente `App` que en el template levantaba el estado `user` una sola vez; cada página se monta de forma independiente. *Descartado:* introducir un `UserContext` — se deja para un spec futuro si la complejidad de sesión crece.

## Riesgos identificados

- **Conflicto de convención de fuentes.** Este spec deja fuentes vía `<link>` de Google Fonts, mientras `CLAUDE.md` documenta `next/font/google` (Geist) como convención del proyecto. Un desarrollador o agente futuro podría "corregir" esto sin saber que fue una decisión explícita. *Mitigación:* la excepción queda registrada en la sección de decisiones de este spec; si se retoma la convención de `CLAUDE.md` en un spec posterior, debe hacerse explícitamente.

- **Colisión de reset CSS entre Tailwind y `styles.css`.** Tailwind v4 (`@import "tailwindcss"`) aplica su propio preflight (box-sizing, márgenes, etc.), que puede pisar o entrar en conflicto con las reglas base de `styles.css` (que también define `* { box-sizing: border-box }`, estilos de `body`, etc.). *Mitigación:* revisar visualmente cada pantalla contra `references/templates/Arcade Vault.html` tras la migración (paso 9 del plan); si hay conflictos, ajustar el orden de los `@import` o acotar el preflight de Tailwind.

- **Hidratación con `localStorage`.** Leer `av_user`/`av_scores` durante el render de un Server Component o en el primer render de un Client Component puede causar mismatches de hidratación (el servidor no tiene acceso a `localStorage`). *Mitigación:* todos los componentes que leen sesión (`Nav`, reproductor, salón de la fama, login) deben ser `"use client"` y leer `localStorage` dentro de `useEffect`, nunca durante el render inicial.
