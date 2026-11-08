# BlastMath 2 — Iteración de Ideas

> Estado: borrador de ideas, pre-planificación de fases.

## Stack técnico (definido)

| Herramienta | Decisión |
|---|---|
| Bundler | **Vite** |
| Engine | **Phaser 3.88.2** |
| Package manager | **pnpm** |
| Deploy | **GitHub Pages** |
| Lenguaje | **JavaScript** |

---

## Ideas analizadas

### ✅ Claras — listas para planificar

| # | Idea | Notas técnicas |
|---|---|---|
| 1 | 4 operaciones (+-×÷) | `generateQuestion()` necesita una estrategia por operación. Para división: solo divisiones exactas (ej: 24÷6=4). |
| 2 | Sistema de vidas (3) | Estado global `lives`. Phaser facilita esto con eventos. Un corazón visual por vida. |
| 3 | Pantalla de inicio | `Scene: MainMenu` en Phaser. Escena separada. |
| 8 | LocalStorage top 10 scores | `ScoreService` simple. Guardar array de `{score, date, operations}`. |
| 11 | Selector de operaciones | Pantalla antes de jugar. El jugador tilda qué operaciones quiere. Mínimo 1 requerida. |
| 12 | Publicar en GitHub Pages | `vite-plugin-gh-pages` o script en `package.json`. Configurar `base` en `vite.config`. |

---

### ⚠️ Necesitan iteración antes de planificar

#### Idea #4 — Niveles / Progresión ✅ DEFINIDA

Cada nivel cambia **escena, velocidad y comportamiento de los enemigos**. Los cambios se suman progresivamente:

| Nivel | Escena | Velocidad | Movimiento enemigos | Enemigos disparan |
|---|---|---|---|---|
| 1 | Escena A | Base | Descenso recto (como v1) | ❌ |
| 2 | Escena B | +25% | Zigzag horizontal | ❌ |
| 3 | Escena C | +50% | Patrón diagonal | ✅ (lento) |
| 4+ | *(Fase 2+)* | — | — | — |

**Trigger de level up**: cada 10 respuestas correctas (10 → nivel 2, 20 → nivel 3).

**Implicaciones arquitectónicas**:
- Necesitamos un `LevelConfig` por nivel (scene, speed multiplier, movement pattern, can shoot)
- Sistema de proyectiles **bidireccional**: jugador dispara Y recibe disparos
- Múltiples backgrounds / escenas en Phaser
- Sprites de enemigos posiblemente distintos por nivel

> ⚠️ **Nota de balance**: enemigos que disparan + resolver cuentas mentales puede saturar al jugador, especialmente en mobile. Requiere prueba y ajuste de timing.

---

#### Idea #7 — Opciones de respuesta incorrecta

La propuesta de ±5 tiene problemas en los bordes:

| Escenario | Problema |
|---|---|
| `2 + 3 = 5`, opciones incorrectas ±5 | Pueden aparecer `0` o negativos |
| `9 × 9 = 81`, opciones incorrectas ±5 | `76, 77, 78, 79, 83...` son demasiado obvias para un jugador con la tabla aprendida |
| `20 ÷ 4 = 5`, opciones incorrectas ±5 | `1, 2, 3, 4, 6, 7, 8, 9, 10` — rango muy pequeño, puede parecer fácil |

**Propuesta mejorada**:
```
1. Siempre positivos (nunca cero ni negativos)
2. Nunca iguales entre sí ni a la correcta
3. "Plausibles": calculados como ±(10% del resultado, mínimo 2, máximo 10)
4. Para multiplicación: incluir un "trampa" — resultado de números adyacentes (ej: 9×7=63 como distractor de 9×8=72)
```

> ⚠️ Esto vale la pena definirlo bien antes de codear porque afecta directamente la UX del aprendizaje.

---

#### Idea #10 — Assets ✅ DEFINIDA

Fuente principal: **Kenney.nl** (licencia CC0, sin atribución requerida).

| Paquete | Uso | Estado |
|---|---|---|
| [Space Shooter Remastered](https://kenney.nl/assets/space-shooter-remastered) | Naves, enemigos, balas, explosiones | ✅ Confirmado |
| Fondos espaciales (3 niveles) | 3 backgrounds distintos | ⚠️ "Space Backgrounds" ya no existe — confirmar al descargar si Space Shooter Remastered los incluye; alternativa: [OpenGameArt.org](https://opengameart.org) (CC0) |
| [Digital Audio](https://kenney.nl/assets/digital-audio) | Disparo, explosión, vida perdida, level up | ✅ Confirmado |
| [UI Pack](https://kenney.nl/assets/ui-pack) | Botones, paneles, corazones (vidas) | ✅ Confirmado |

---

#### Idea #13 — Juego descargable (PWA / offline)

Con Vite + `vite-plugin-pwa` esto es **casi gratis** — agrega un Service Worker y un manifest. El juego se puede "instalar" desde el navegador sin App Store.

**Decisión**: queda para **Fase 2**, porque:
- En Fase 1 el foco es que el juego funcione bien
- La PWA es una capa encima que no bloquea nada
- Si los assets cambian mucho en Fase 1, el cache de la PWA complica el desarrollo

---

## Plan de fases definitivo

### Fase 1 — Base jugable (este repo)

**Objetivo**: juego completo y publicado en GitHub Pages.

#### Setup
- Vite + Phaser 3.88.2 + pnpm
- Estructura de carpetas con escenas separadas
- Deploy automático a GitHub Pages

#### Escenas Phaser

| Escena | Descripción |
|---|---|
| `MainMenuScene` | Logo, botón "Jugar", botón "Leaderboard" |
| `OperationSelectScene` | Checkboxes para elegir operaciones (mínimo 1) |
| `GameScene` | Loop principal del juego |
| `LevelUpScene` | Pantalla de transición entre niveles |
| `GameOverScene` | Score final, guardar en top 10, botón reiniciar |
| `LeaderboardScene` | Top 10 scores desde localStorage |

#### Funcionalidades

- ✅ 4 operaciones (+-×÷) con división entera siempre
- ✅ Selector de operaciones antes de jugar
- ✅ Sistema de 3 vidas con icono de corazón
- ✅ Algoritmo de respuestas incorrectas mejorado (plausibles, siempre positivas)
- ✅ LocalStorage top 10 scores (con fecha y operaciones usadas)
- ✅ Responsive con Phaser Scale Manager (mobile + desktop)
- ✅ Sonidos con Kenney assets
- ✅ Deploy en GitHub Pages

#### 3 Niveles

| Nivel | Trigger | Escena/fondo | Velocidad | Movimiento | Disparan |
|---|---|---|---|---|---|
| 1 | Inicio | Espacio profundo | Base | Descenso recto | ❌ |
| 2 | 10 respuestas correctas | Nebulosa | +25% | Zigzag horizontal | ❌ |
| 3 | 20 respuestas correctas | Planeta hostil | +50% | Diagonal | ✅ lento |

---

### Fase 2 — Expansión (futuro)

- Niveles 4+ (nuevas mecánicas: swarm, enemigos más inteligentes)
- PWA / instalable sin conexión
- Música de fondo por nivel
- Dificultad seleccionable (Fácil / Normal / Difícil) que ajusta rangos numéricos
- Más variedad de preguntas (números más grandes en niveles altos)

---

## Decisiones — TODAS RESUELTAS ✅

| # | Decisión | Resultado |
|---|---|---|
| A | ~~¿Qué significa "subir de nivel"?~~ | Escena + velocidad + comportamiento de enemigos |
| B | ~~¿De dónde vienen los assets?~~ | Kenney.nl para Fase 1 |
| C | ~~¿JavaScript o TypeScript?~~ | JavaScript |
| D | ~~¿Cuántos niveles en Fase 1?~~ | 3 niveles |
