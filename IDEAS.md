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

## Mecánica core — DEFINIDA (iteración 2)

### Cómo se juega una pregunta

- **Enemigos = opciones.** La cuenta se muestra fija arriba (HUD); cada oleada genera **3 enemigos**, cada uno con una respuesta: 1 correcta + 2 distractores del algoritmo de la idea #7.
- Disparo al enemigo correcto → explota, suma puntos, arranca la próxima oleada.
- Disparo a un enemigo incorrecto → cuenta como respuesta incorrecta (−1 vida) y el enemigo explota (queda fuera de pantalla).
- **Sin timer**: la presión la da el descenso. Si el enemigo con la respuesta correcta llega a la base, la oleada se cae: −1 vida y oleada nueva.
- Los enemigos llevan texto legible (fuente grande, se lee a distancia) — coherente con el público chico.

### Cuándo se pierde una vida — los 3 castigos

1. Disparar a un enemigo con respuesta incorrecta.
2. Que un enemigo llegue a la base.
3. Que un proyectil enemigo impacte al jugador (nivel 3+).

> ⚠️ **Balance**: en nivel 3 se combinan los 3 castigos. Para chicos puede ser demasiado — candidata a suavizar: en nivel 3 errar una cuenta no resta vida (el enemigo explota y nada más), o disparos enemigos lentos y telegrafiados. A ajustar con playtesting.

### Público y rangos numéricos

Público: **chicos aprendiendo tablas**. UI grande y simple, tono amigable.

| Operación | Rango |
|---|---|
| Suma | operandos 1–10, resultado ≤ 20 |
| Resta | resultado entre 0 y 20, nunca negativa |
| Multiplicación | tablas del 1 al 10 |
| División | exacta, dividendo ≤ 100 (inverso de las tablas), símbolo `÷` |

### Score y fairness del leaderboard

- Base: 10 puntos por respuesta correcta (referencia, ajustable).
- **Multiplicador por operaciones elegidas**: 1 op ×1 · 2 ops ×1.25 · 3 ops ×1.5 · 4 ops ×2.
- El top 10 guarda también qué operaciones se usaron (informativo).
- El jugador ingresa sus iniciales al terminar la partida y el top 10 las muestra.
- Idea extra no decida: bonus por racha de aciertos.

## Pendientes menores

- Confirmar si los fondos espaciales vienen en Space Shooter Remastered (idea #10) — decisión diferida: se revisa más adelante.

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
| `GameOverScene` | Score final, ingreso de iniciales, guardar en top 10, botón reiniciar |
| `LeaderboardScene` | Top 10 scores desde localStorage |

#### Funcionalidades

- ✅ 4 operaciones (+-×÷) con división entera siempre
- ✅ Selector de operaciones antes de jugar
- ✅ Sistema de 3 vidas con icono de corazón
- ✅ Algoritmo de respuestas incorrectas mejorado (plausibles, siempre positivas)
- ✅ LocalStorage top 10 scores (con fecha y operaciones usadas)
- ✅ Responsive con Phaser Scale Manager (mobile + desktop), orientación landscape fija
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
| E | ~~¿Cómo se juega una pregunta?~~ | Enemigos = opciones: cuenta fija arriba, cada oleada trae 3 enemigos con las respuestas |
| F | ~~¿Cuándo se pierde una vida?~~ | Los 3: respuesta incorrecta, enemigo en la base, proyectil enemigo (nivel 3+) |
| G | ~~¿Para quién es?~~ | Chicos aprendiendo tablas (rangos ≤ 20 / tablas 1–10) |
| H | ~~¿Top 10 comparable entre partidas?~~ | Multiplicador de score según cantidad de operaciones elegidas |
| I | ~~¿Iniciales del jugador en el top 10 o anónimo?~~ | Con iniciales del jugador |
| J | ~~¿Orientación?~~ | Landscape, pensado también para móvil |
