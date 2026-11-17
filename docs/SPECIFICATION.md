# Especificación Técnica y Funcional — BalanceAct

> Única fuente de verdad técnica y funcional de **BalanceAct**. El flujo e interfaz funcional están alineados con [wireframe-balanceact.html](wireframe/wireframe-balanceact.html), que prevalece ante cualquier divergencia funcional mientras no exista el diseño visual definitivo. La estética (colores, tipografía, íconos, diseño de las cartas y sus ilustraciones) la define el diseñador externo. Las decisiones que no venían dadas por el contexto están marcadas y reunidas en [§16 Supuestos para revisar](#16-supuestos-para-revisar).

---

## 1. Propósito y alcance

**BalanceAct** es una aplicación de escritorio educativa en la que el estudiante administra tres áreas de su vida universitaria —**Estudios**, **Amigos** y **Descanso**— eligiendo entre dos opciones ante situaciones presentadas en cartas. Cada decisión sube unas áreas y baja otras; la partida termina cuando alguna llega a un extremo o cuando se completa el semestre. La actividad estimula:

- **La toma de decisiones con varias prioridades:** ninguna opción es buena para las tres áreas a la vez.
- **La anticipación de consecuencias:** antes de elegir se ve la dirección del cambio de cada área, pero no su magnitud.
- **La organización del tiempo personal:** al final de cada partida se muestra qué área quedó en segundo plano con más frecuencia.

### Contexto operativo y restricciones de producto

- **Uso:** PC con Windows sin conexión a internet, en aula, laboratorio o en casa. Varios estudiantes pueden compartir el mismo equipo mediante perfiles locales aislados.
- **Revisión:** presencial en la propia pantalla junto al docente o tutor.
- **Alcance no clínico:** herramienta de entrenamiento y reflexión; **no** es una prueba psicométrica ni ofrece diagnósticos.
- **Tono e idioma:** el nombre del producto (`BalanceAct`) y la carpeta de datos (`%LOCALAPPDATA%/BalanceAct`) se mantienen siempre en inglés; toda la interfaz, cartas, instrucciones y retroalimentación se redactan en español con tono formativo, claro y **sin juzgar las prioridades del estudiante**: no hay área "correcta", llegar a un extremo no es un fracaso y el exceso se describe igual que el descuido.
- **Sin red ni exportación:** prohibido exportar datos en CSV u otros archivos externos, y prohibido enviar información por red.

### Glosario

| Término | Definición |
| :--- | :--- |
| **Área** | Cada una de las tres dimensiones que el estudiante administra: `Estudios`, `Amigos` y `Descanso`. Tiene un valor entero entre `0` y `100`. |
| **Carta** | Situación de la vida universitaria con dos opciones. Su texto y efectos los redacta el equipo de programación; su diseño e ilustración, el diseñador. |
| **Opción** | Cada una de las dos respuestas de una carta. Aplica un efecto (cambio con signo) a cada área. |
| **Pista de dirección** | Indicación visible antes de elegir: por cada área, si la opción la **sube**, la **baja** o la **deja igual**, sin mostrar la magnitud. |
| **Semana** | Una carta decidida. Una partida tiene como máximo **16 semanas** (un semestre). |
| **Zona de riesgo** | Valor de un área `≤ 20` o `≥ 80`. Se avisa con texto y forma, no solo con color. |
| **Extremo** | Valor `0` (mínimo, vacía) o `100` (máximo, desbordada). Alcanzar cualquiera termina la partida. |
| **Área cedida** | Área que la opción elegida hace bajar. Es la base de la medida "áreas que el estudiante descuida con más frecuencia". |
| **Partida** | Ciclo completo: autorreporte inicial + hasta 16 semanas + autorreporte final + resultados. |

---

## 2. Stack y dependencias aprobadas

Las siguientes dependencias y versiones mayores (alineadas con TheirView y TracePath) quedan **expresamente aprobadas** para este proyecto a efectos de la puerta **G3** de [AGENTS.md](../AGENTS.md). BalanceAct **no tiene escena 3D**: no se usa Three.js ni ninguna otra librería gráfica.

| Componente | Tecnología / Paquete | Versión mayor aprobada | Rol |
| :--- | :--- | :--- | :--- |
| Runtime de escritorio | `electron` | `^31.0.0` (v31) | Proceso principal, ventana y preload |
| Empaquetado | `electron-builder` | `^24.13.3` (v24) | Instalador NSIS por usuario y portable |
| Interfaz | `react`, `react-dom` | `^18.3.1` (v18) | Vistas y componentes del renderer |
| Bundler | `vite`, `@vitejs/plugin-react` | `^5.3.1` (v5) / `^4.3.1` (v4) | Compilación de renderer y preload |
| Lenguaje | `typescript` | `^5.5.2` (v5, modo estricto) | Tipado estricto en main, preload, lógica y renderer |
| Testing | `vitest`, `jsdom` | `^2.0.0` (v2) / `^24.1.0` (v24) | Pruebas unitarias y de persistencia |
| Tipos auxiliares | `@types/node`, `@types/react`, `@types/react-dom` | `^20.14.0` / `^18.3.x` | Definiciones de tipos |
| Gestor de paquetes | `pnpm` | Único permitido | Prohibido `npm` o `yarn` y sus lockfiles |

Las ilustraciones, íconos y el logo que entrega el diseñador se empaquetan localmente dentro del bundle, sin dependencias de red ni CDN.

---

## 3. Estructura de carpetas

```text
balance-act/
├── build/                      # Ícono oficial y recursos de empaquetado
├── docs/
│   ├── SPECIFICATION.md        # Esta especificación técnica y funcional
│   ├── capturas/               # Capturas PNG generadas por pnpm docs:screenshots
│   └── wireframe/              # Wireframe HTML de referencia funcional y su prompt
├── electron/
│   ├── main.ts                 # Proceso principal, ventana e IPC handlers
│   ├── preload.ts              # contextBridge seguro (window.balanceActApi)
│   ├── ipcValidation.ts        # Validación estricta de argumentos IPC
│   └── storage/
│       └── atomicStore.ts      # Persistencia atómica (.tmp -> rename) con lock por archivo
├── scripts/
│   ├── dev.mjs                 # Arranque de desarrollo
│   ├── capture-screenshots.mjs # Orquestador de capturas con perfil temporal
│   └── capture-screenshots-main.cjs # Entrada Electron para capturas
├── src/
│   ├── shared/
│   │   ├── types.ts            # Contratos globales de datos e IPC
│   │   ├── ipcChannels.ts      # Constantes de canales IPC
│   │   ├── deck.ts             # Mazo de 24 cartas: textos, efectos y referencia a su ilustración
│   │   └── feedback.ts         # Textos de retroalimentación formativa por caso
│   ├── logic/                  # Lógica pura (sin Electron ni DOM)
│   └── renderer/
│       ├── main.tsx            # Entrypoint React 18
│       ├── App.tsx             # Orquestador de las 10 pantallas
│       ├── assets/             # Ilustraciones, íconos y logo entregados por el diseñador
│       ├── audio/              # Motor de efectos sonoros
│       ├── components/         # Componentes reutilizables (BackButton, AreaMeter, Card, diálogos)
│       ├── hooks/              # Hooks de estado global sincronizado con IPC
│       ├── screens/            # Las 10 pantallas de la aplicación
│       └── styles/             # Tokens CSS y reglas de alto contraste / texto grande
├── tests/                      # Tests unitarios (logic/, storage/, deck/, etc.)
├── electron-builder.yml
├── package.json
├── pnpm-lock.yaml
├── tsconfig.json
└── vite.config.ts
```

**Archivos compartidos que obligan a secuenciar worktrees:** `package.json`, `pnpm-lock.yaml`, `src/shared/types.ts` y `src/shared/deck.ts`.

---

## 4. Flujo y pantallas

La aplicación consta de **10 pantallas** (`ScreenId`). No hay mapa de sesiones: una partida es una unidad corta y se inicia directamente desde `MENU`.

```text
SPLASH ──(~2.5 s)──▶ USER_SELECT ──(seleccionar / crear)──▶ MENU
                                                              │
       ┌──────────────────┬───────────────────┬───────────────┤
       ▼                  ▼                   ▼               ▼
   [Continuar]        [Jugar]            [Ver avance]     [Ajustes]
   (si hay en curso)      │                   │               │
       │       (diálogo si hay partida        ▼               ▼
       │        en curso: descartar)       RECORDS         SETTINGS
       │                  │                                   │
       │                  ▼                                   │
       │            GUIDE (1.ª vez) ◀─────────────────────────┘ (Ver guía → GUIDE → SETTINGS)
       │                  │
       │                  ▼
       └──────────▶  PRE ──▶ GAME (hasta 16 semanas: carta → decisión → retroalimentación) ──▶ POST ──▶ RESULTS ──▶ MENU
```

- **Sin callejones sin salida:** las pantallas secundarias (`GUIDE`, `SETTINGS`, `RECORDS`) incluyen un `BackButton` arriba a la izquierda para regresar a `MENU` (o a `SETTINGS` cuando `GUIDE` se abrió desde allí). `PRE` tiene `Volver al menú`.

| `ScreenId` | Propósito y comportamiento clave |
| :--- | :--- |
| `SPLASH` | Logotipo centrado de `BalanceAct` con fundido breve (respeta `reduceMotion` y `prefers-reduced-motion`). Sin botones; avanza automáticamente a `USER_SELECT` tras ~2.5 s. En paralelo precarga por IPC usuarios, perfil activo, ajustes e historial; ante cualquier fallo usa valores por defecto sin bloquear el arranque. |
| `USER_SELECT` | Lista perfiles locales aislados (nombre y última actividad) con botón eliminar por ícono y `aria-label`. Campo único «Ingresar o crear perfil» (máx. 32 caracteres; botón deshabilitado si está vacío o inválido): alterna entre `Ingresar` (si coincide) y `Crear perfil` (nuevo). Seleccionar con clic, `Enter` o `Espacio` lleva a `MENU`. Eliminar abre diálogo modal (`role="dialog"`, `aria-modal="true"`, `aria-labelledby`, foco atrapado, cierre con `Escape`); si se elimina el perfil activo, ajustes e historial vuelven a valores por defecto. |
| `MENU` | Botón superior izquierdo `Cambiar usuario` con el nombre del perfil activo. Marca `BalanceAct` centrada y una fila única centrada de botones cuadrados con ícono: `Continuar` (visible y destacado solo si existe partida `in_progress`, con la línea «Semana N de 16»), `Jugar`, `Ver avance` y `Ajustes`. Si hay partida en curso, `Jugar` abre el diálogo de descarte. Incluye el aviso de alcance no clínico. |
| `GUIDE` | Guía de 3 pasos (`Paso N de 3`) con `Anterior`, `Siguiente` / `Empezar`, `Omitir` y `BackButton`: (1) las tres áreas, su rango y la zona de riesgo; (2) la carta, las dos opciones y las pistas de dirección; (3) cómo termina la partida (extremos y semestre completo) y qué se muestra al final. Al finalizar u omitir por primera vez marca `hasSeenGuide = true` y avanza a `PRE`; si se abrió desde `SETTINGS`, regresa a `SETTINGS`. |
| `SETTINGS` | Ajustes por perfil aplicados y guardados al instante (sin botón guardar): **Alto contraste**, **Tamaño de texto** (`normal` / `large`), **Reducir movimiento**, **Sonido** (interruptor) y **Volumen** (deslizador `0..1` con `aria-label`, deshabilitado con el sonido apagado), **Ver guía de nuevo** y **Vista previa en vivo** (carta de ejemplo con los tres medidores, que refleja al instante contraste y tamaño de texto). Botón `Volver al menú`. Sin ajustes propios (ver §7). |
| `RECORDS` | Historial del perfil activo: resumen acumulado («área que más cediste en tus partidas», con conteo por área) y tabla con fecha, estado (`COMPLETADA` / `EN CURSO`), semanas jugadas sobre 16, cierre (área y extremo, o semestre completo), área más cedida, tiempo de partida y variación de ánimo y estrés (`—` si aún no hay post). Seleccionar una fila (clic, `Enter` o `Espacio`) muestra el detalle por semana (`Semana`, `Situación`, `Opción elegida`, `Cambios`, `Valores tras decidir`, `Tiempo de decisión`). Sin exportación de archivos. |
| `PRE` | Dos escalas Likert obligatorias de 1 a 5: `Estado de ánimo` y `Nivel de estrés`. Botón `Empezar semestre` (`Enter`) deshabilitado hasta responder ambas; botón `Volver al menú`. Al confirmar se crea la partida, se elige la primera carta y se guarda un checkpoint. |
| `GAME` | Pantalla central de juego, con tres zonas fijas: (1) barra superior con `Semana N de 16`, sonido y `Pausar y salir al menú`; (2) fila de tres **medidores** (ícono, nombre, valor `N / 100`, barra con marcas de zona de riesgo y estado textual `Equilibrio` / `En riesgo · bajo` / `En riesgo · alto`); (3) la **carta** centrada (ilustración, situación) con las dos **opciones** debajo, cada una con su texto y sus pistas de dirección por área. Elegir con clic en una opción, arrastrando la carta hacia el lado de la opción (con el mouse, como alternativa), con `←` / `→` o con `Tab` + `Enter`. Tras elegir, la **retroalimentación inmediata** reemplaza las opciones en el mismo lugar: opción elegida, cambio con magnitud por área (p. ej. `Estudios +10 · Amigos −5 · Descanso −10`), medidores actualizados y botón `Siguiente semana` (`Enter` / `Espacio`). Si la decisión lleva un área a un extremo o completa la semana 16, el botón pasa a `Ver cierre` y se muestra el estado de fin de partida (§5) antes de `POST`. Al aplicar cada decisión invoca `saveCheckpoint`. |
| `POST` | Dos escalas Likert obligatorias de 1 a 5: `Estado de ánimo` y `Nivel de estrés`, con una línea que recuerda cómo terminó la partida. Botón `Ver resultados` (`Enter`) habilitado al responder ambas; marca la partida como `completed`. |
| `RESULTS` | Retroalimentación de la partida: cuatro tarjetas (`Semanas sostenidas` sobre 16, `Cierre`, `Tiempo de partida`, `Tiempo medio por decisión`), valores finales de las tres áreas, **áreas cedidas** (conteo por área y cuál fue la más frecuente), evolución de las tres áreas semana a semana, texto formativo (§5) con una pregunta de reflexión, variación inicial → final de ánimo y estrés, tabla de decisiones y botones `Jugar otra vez` y `Volver al menú`. |

---

## 5. Reglas del ejercicio

- **Áreas y rango:** `Estudios`, `Amigos` y `Descanso`, cada una con un valor entero en `0..100`. **Valor inicial: `50`** en las tres.
- **Efecto de una opción:** un cambio entero por área (`delta`), múltiplo de 5, entre `−20` y `+20` (puede ser `0`). Cada opción tiene al menos un área que sube y al menos una que baja, de modo que siempre hay algo que se cede. El nuevo valor se acota a `0..100`.
- **Fin por extremo:** la partida termina en cuanto un área llega a **`0` (mínimo)** o a **`100` (máximo)**. Ambos extremos terminan la partida: el exceso también desequilibra. Si en una misma decisión varias áreas llegan a un extremo, se registran todas.
- **Fin por semestre completo:** si se deciden **16 semanas** sin alcanzar ningún extremo, la partida termina como «Semestre completo». Si la decisión de la semana 16 lleva un área a un extremo, cuenta como fin por extremo.
- **Zona de riesgo:** un área con valor `≤ 20` o `≥ 80` muestra el estado textual `En riesgo · bajo` o `En riesgo · alto` y una marca de forma en su medidor. No bloquea nada; solo avisa.
- **Pistas de dirección antes de elegir:** cada opción muestra, por área, `sube` (`↑`), `baja` (`↓`) o `igual` (`=`), siempre con texto. **No se muestra la magnitud** antes de elegir; sí después, en la retroalimentación inmediata.
- **Sin límite de tiempo por carta:** no hay cronómetro visible ni presión de reloj. El tiempo se mide pero no se muestra durante la partida.
- **Mazo:** **24 cartas** en `src/shared/deck.ts`, repartidas en tres tramos de intensidad de 8 cartas cada uno:
  - **Tramo 1 (semanas 1–5):** efectos de `±5` a `±10`, normalmente sobre dos áreas.
  - **Tramo 2 (semanas 6–11):** efectos de `±10` a `±15`.
  - **Tramo 3 (semanas 12–16):** efectos de `±15` a `±20`, y cada opción afecta a las tres áreas.
- **Elección de la siguiente carta:** se toma una carta **no usada en la partida** del tramo correspondiente a la semana, al azar con pesos: peso `2` para las cartas que tienen algún efecto distinto de `0` sobre un área en zona de riesgo, peso `1` para las demás. Así las situaciones que tocan el área en riesgo aparecen con más frecuencia, sin forzar una salida. La carta elegida se guarda en el checkpoint (`pendingCardId`) para que cerrar y reabrir la app no cambie la carta.
- **Medida de duración:** se registran (1) **semanas jugadas** (cartas decididas, `1..16`), (2) **tiempo de decisión** de cada carta en milisegundos (desde que la carta queda visible hasta la elección) y (3) **tiempo de partida** (`activeMs`): tiempo activo en `GAME`, sumando decisión y retroalimentación y **excluyendo** pausas y el tiempo con la app cerrada. `RESULTS` muestra semanas, tiempo de partida y tiempo medio por decisión.
- **Áreas cedidas:** en cada decisión, toda área con `delta < 0` en la opción elegida cuenta como **cedida** una vez. El **área más cedida** de la partida es la de mayor conteo; ante empate se muestran todas las empatadas. `RECORDS` acumula el conteo de todas las partidas completadas del perfil.
- **Decisiones de reequilibrio:** decisión tomada con al menos un área en zona de riesgo en la que la opción elegida acerca esa área a `50`. Se registra como indicador de anticipación (`rebalanceCount`) y se menciona en la retroalimentación sin calificar.
- **Retroalimentación al final de la partida:** texto formativo armado con plantillas de `src/shared/feedback.ts` según el cierre (extremo mínimo, extremo máximo o semestre completo) y el área más cedida, más una pregunta de reflexión abierta. Nunca usa "ganaste", "perdiste", "mal" o "bien"; describe lo ocurrido («Descanso fue el área que más veces quedó en segundo plano: 7 de 12 decisiones») e invita a pensar («¿Qué semana habrías repartido distinto?»).
- **Autorreportes pre y post:** escalas Likert de 1 a 5 de estado de ánimo y nivel de estrés antes (`PRE`) y después (`POST`) de cada partida, como en el resto de la familia de productos.

### Contenido de las cartas

Los textos (situación y opciones) y los efectos de las 24 cartas los redacta el equipo de programación y conviene que los revise el docente o el cliente. Cada carta referencia una ilustración por identificador (`illustrationId`) que entrega el diseñador; varias cartas pueden compartir ilustración. Situaciones breves (máx. 140 caracteres) y opciones cortas (máx. 48 caracteres), en segunda persona y sin juicios de valor.

---

## 6. Perfiles

- **Campo de perfil:** acepta un nombre o un código de estudiante.
- **Validación de formato:**
  - Longitud: `1` a `32` caracteres tras recortar espacios extremos (`trim()`).
  - Caracteres permitidos: letras (incluidas tildes y `ñ`/`Ñ`), dígitos (`0–9`), espacios, guion (`-`) y punto (`.`).
  - Cualquier otro carácter se rechaza antes de crear el perfil, mostrando un mensaje claro al usuario.
- **Coincidencia:** sin distinguir mayúsculas ni minúsculas (case-insensitive) tras recortar espacios extremos. Si coincide con un perfil existente, la acción ingresa a ese perfil; si no existe y es válido, lo crea.
- **Aislamiento y eliminación:** cada perfil tiene su propia carpeta `users/<userId>/` con sus ajustes e historial. Eliminar un perfil exige confirmación en diálogo modal y borra su carpeta; si era el perfil activo, `activeUserId` pasa a `null` y el estado en memoria vuelve a valores por defecto.

---

## 7. Ajustes

Todos los ajustes se aplican al instante en toda la aplicación y se persisten automáticamente en el perfil activo (sin botón guardar):

| Ajuste | Tipo / Valores | Comportamiento |
| :--- | :--- | :--- |
| **Alto contraste** | `boolean` (`data-contrast="high"` en raíz) | Activa la variante de alto contraste definida sobre los tokens CSS. Los colores concretos los define el diseñador. |
| **Tamaño de texto** | `'normal' \| 'large'` | Escala la tipografía de toda la interfaz, incluidas las cartas. |
| **Reducir movimiento** | `boolean` (`reduceMotion`) | Suprime o simplifica la entrada y salida de las cartas, la animación de los medidores y el fundido de `SPLASH`. Respeta además `prefers-reduced-motion` del sistema. |
| **Sonido y volumen** | `audioEnabled: boolean`, `audioVolume: number` (`0..1`) | Silencia o activa los efectos sonoros (sincronizado con el control de la barra de juego) y gradúa el volumen; el deslizador se deshabilita cuando el sonido está apagado. |
| **Ver guía de nuevo** | Acción | Abre `GUIDE` y regresa a `SETTINGS` al completar o pulsar volver. |

- **Vista previa en Ajustes:** tarjeta en vivo con una carta de ejemplo y los tres medidores que refleja de inmediato los cambios de alto contraste y tamaño de texto.
- **Sin ajustes propios:** no se agregan ajustes específicos del juego. Las pistas de dirección forman parte de las reglas y no se pueden ocultar, para que las partidas de un perfil sean comparables entre sí.

---

## 8. Persistencia

Toda la persistencia reside exclusivamente en `%LOCALAPPDATA%/BalanceAct/`:

```text
%LOCALAPPDATA%/BalanceAct/
├── users.json
└── users/
    └── <userId>/
        ├── settings.json
        └── games.json
```

- **`users.json`:** lista de perfiles (`UserProfile[]`) y `activeUserId` (`string | null`).
- **`users/<userId>/settings.json`:** preferencias aisladas del perfil (`UserSettings`).
- **`users/<userId>/games.json`:** historial de partidas (`GameRecord[]`) del perfil, incluida la partida `in_progress` si existe (como máximo una).
- **Escritura atómica y concurrencia:** cada escritura crea un archivo temporal en el mismo directorio y ejecuta `rename` atómico, serializado mediante un lock por ruta de archivo para evitar corrupción ante escrituras rápidas consecutivas (cambios en `SETTINGS` o checkpoints).
- **Checkpoint por decisión:** se invoca `saveCheckpoint` al crear la partida (tras `PRE`, con la primera carta elegida) y al aplicar cada decisión (con valores, decisión registrada, `activeMs` acumulado y la siguiente carta ya elegida en `pendingCardId`). Al retomar:
  - si la partida no terminó, se entra en `GAME` mostrando `pendingCardId` en la semana `weeksPlayed + 1`;
  - si terminó (`endReason !== null`) pero falta el autorreporte final, se entra en `POST`.

---

## 9. Contrato de datos (`src/shared/types.ts`) e IPC

### Tipos compartidos (`src/shared/types.ts`)

```ts
export type ScreenId =
  | 'SPLASH'
  | 'USER_SELECT'
  | 'MENU'
  | 'GUIDE'
  | 'SETTINGS'
  | 'RECORDS'
  | 'PRE'
  | 'GAME'
  | 'POST'
  | 'RESULTS';

export type TextSize = 'normal' | 'large';
export type GameStatus = 'in_progress' | 'completed';
export type AreaId = 'studies' | 'friends' | 'rest';
export type EndReason = 'limit' | 'semester';
export type Extreme = 'min' | 'max';
export type OptionSide = 'left' | 'right';

export type AreaValues = Record<AreaId, number>;   // 0..100
export type AreaDelta = Record<AreaId, number>;    // múltiplo de 5 en -20..20

export interface CardOption {
  label: string;           // máx. 48 caracteres
  delta: AreaDelta;
}

export interface CardDefinition {
  id: string;              // p. ej. 'c01'
  tier: 1 | 2 | 3;
  situation: string;       // máx. 140 caracteres
  illustrationId: string;  // recurso entregado por el diseñador
  left: CardOption;
  right: CardOption;
}

export interface LikertPair {
  mood: number;            // 1..5
  stress: number;          // 1..5
}

export interface DecisionRecord {
  week: number;            // 1..16
  cardId: string;
  side: OptionSide;
  before: AreaValues;
  delta: AreaDelta;
  after: AreaValues;
  ceded: AreaId[];         // áreas con delta < 0
  rebalance: boolean;
  decisionMs: number;
}

export interface LimitHit {
  area: AreaId;
  extreme: Extreme;
}

export interface UserProfile {
  id: string;
  name: string;
  createdAt: string;
  lastActiveAt: string;
}

export interface UsersData {
  version: 1;
  activeUserId: string | null;
  users: UserProfile[];
}

export interface UserSettings {
  version: 1;
  highContrast: boolean;
  textSize: TextSize;
  reduceMotion: boolean;
  audioEnabled: boolean;
  audioVolume: number;
  hasSeenGuide: boolean;
}

export interface GameRecord {
  id: string;
  startedAt: string;
  endedAt: string | null;
  status: GameStatus;
  pre: LikertPair;
  post: LikertPair | null;
  values: AreaValues;
  weeksPlayed: number;
  pendingCardId: string | null;
  decisions: DecisionRecord[];
  endReason: EndReason | null;
  limitHits: LimitHit[];
  cededCounts: Record<AreaId, number>;
  rebalanceCount: number;
  activeMs: number;
  meanDecisionMs: number | null;
}
```

### API IPC (`window.balanceActApi`) y validación (`electron/ipcValidation.ts`)

Todos los canales expuestos por `preload.ts` validan argumentos, tipos y rangos en `ipcValidation.ts` antes de acceder al sistema de archivos:

| Método en `window.balanceActApi` | Propósito |
| :--- | :--- |
| `getUsers(): Promise<UsersData>` | Lee `users.json` con la lista de perfiles y `activeUserId`. |
| `getActiveUser(): Promise<UserProfile \| null>` | Resuelve el perfil activo actual o `null`. |
| `selectUser(userId: string): Promise<UserProfile>` | Valida `userId`, actualiza `activeUserId` y `lastActiveAt`. |
| `createUser(name: string): Promise<UserProfile>` | Recorta espacios extremos, valida longitud (`1..32`) y caracteres permitidos, crea `users/<id>/` y archivos iniciales. |
| `deleteUser(userId: string): Promise<UsersData>` | Elimina el perfil y su carpeta `users/<id>/`; si era el activo deja `activeUserId = null`. |
| `getSettings(): Promise<UserSettings>` | Lee `users/<activeUserId>/settings.json` (o valores por defecto si no hay perfil activo). |
| `updateSettings(patch: Partial<UserSettings>): Promise<UserSettings>` | Valida tipos y rangos (`audioVolume` en `0..1`, `textSize` en `'normal' \| 'large'`, booleanos), aplica y persiste atómicamente. |
| `getGames(): Promise<GameRecord[]>` | Lee `users/<activeUserId>/games.json`. |
| `saveCheckpoint(game: GameRecord): Promise<GameRecord[]>` | Valida el `GameRecord` (valores en `0..100`, `weeksPlayed` en `0..16`, `cardId` existentes en el mazo, Likert en `1..5`) y persiste atómicamente. |
| `discardGame(): Promise<GameRecord[]>` | Descarta la partida `in_progress` del perfil activo cuando el usuario confirma empezar una nueva, conservando las completadas. |
| `getStoragePath(): Promise<string>` | Devuelve la ruta efectiva de `%LOCALAPPDATA%/BalanceAct`. |

---

## 10. Lógica pura (`src/logic/`)

Las reglas del juego, la elección de cartas, las métricas, la reanudación y la validación de perfil se implementan como funciones puras en `src/logic/` (sin dependencias de Electron ni DOM). Las que usan azar reciben un generador (`rng: () => number`) para poder probarse con semilla fija:

- **`applyDecision(values, delta)`:** suma y acota cada área a `0..100`.
- **`getAreaStatus(value)`:** `'min'` (`0`), `'risk_low'` (`1..20`), `'balanced'` (`21..79`), `'risk_high'` (`80..99`), `'max'` (`100`).
- **`getDirection(delta)`:** `'up' | 'down' | 'same'` por área, para las pistas de dirección.
- **`checkEnd(values, weeksPlayed)`:** devuelve `{ endReason, limitHits }` según §5 (extremo primero; semestre si `weeksPlayed === 16`).
- **`getTierForWeek(week)`:** `1` (semanas `1..5`), `2` (`6..11`), `3` (`12..16`).
- **`pickNextCard(deck, usedIds, week, values, rng)`:** elige una carta no usada del tramo con los pesos de §5.
- **`isRebalance(before, after)`:** `true` si alguna área en zona de riesgo antes de decidir queda más cerca de `50`.
- **`summarizeGame(decisions)`:** `cededCounts`, área o áreas más cedidas, `rebalanceCount` y `meanDecisionMs`.
- **`buildFeedback(game)`:** selecciona las plantillas de `src/shared/feedback.ts` según cierre y área más cedida.
- **`getResumeState(games)`:** busca la partida `in_progress`; si `endReason === null` devuelve `{ screen: 'GAME', week: weeksPlayed + 1, cardId: pendingCardId }`, si no `{ screen: 'POST' }`.
- **`getStartDestination(settings)`:** devuelve `'GUIDE'` si `!settings.hasSeenGuide`, o `'PRE'` en caso contrario.
- **`validateProfileName(name)`:** reglas de §6.
- **`validateDeck(deck)`:** 24 cartas, 8 por tramo, identificadores únicos, deltas múltiplos de 5 en el rango del tramo, cada opción con al menos un área que sube y una que baja, y en el tramo 3 las tres áreas con delta distinto de `0`.

---

## 11. Convenciones técnicas

- **Tokens CSS:** los colores y variables visuales de la interfaz se consumen exclusivamente mediante tokens CSS (capa de abstracción para soportar `data-contrast="high"`, no para múltiples temas).
- **Frontera estética:** ninguna decisión estética (colores, tipografía, íconos, grosores, diseño de cartas, ilustraciones) se inventa en el código; proviene del diseñador externo. Mientras no llegue, se usan marcadores neutros.
- **Accesibilidad (mouse y teclado):**
  - Todos los controles son operables con teclado. En `GAME`, `←` y `→` eligen la opción izquierda o derecha, y `Enter` / `Espacio` avanzan a la siguiente semana; en el resto, `Enter` y `Espacio` activan el control con foco.
  - Foco visible en todos los elementos interactivos; diálogos modales con `role="dialog"`, `aria-modal="true"`, `aria-labelledby`, foco atrapado y cierre con `Escape`.
  - Los medidores exponen `role="meter"` (o `progressbar`) con `aria-valuenow`, `aria-valuemin="0"`, `aria-valuemax="100"` y su estado textual; los cambios tras una decisión se anuncian en una región `aria-live="polite"`.
  - Contraste mínimo nivel AA en textos, etiquetas y controles.
  - La información visual nunca depende exclusivamente del color: cada área tiene nombre e ícono propios, la zona de riesgo se indica con texto y marca de forma, y las pistas de dirección llevan flecha y palabra (`sube` / `baja` / `igual`).
  - Al ser una aplicación de escritorio operada con mouse y teclado, no existe requisito de área táctil mínima.
- **Seguridad en Electron:** `contextIsolation: true`, `nodeIntegration: false`, `sandbox: true` y validación estricta de todo mensaje IPC en `electron/ipcValidation.ts`.
- **TypeScript estricto:** `strict: true` en `tsconfig.json`, sin usos de `any` injustificados.
- **Sin elevación:** ningún proceso de la aplicación ni del instalador solicita permisos de administrador.

---

## 12. Empaquetado

Configuración en `electron-builder.yml` para Windows (`win.requestedExecutionLevel: asInvoker`), generando artefactos en `release/`:

- **Instalador NSIS por usuario sin elevación:** `perMachine: false`, `allowElevation: false`, `oneClick: true` (`BalanceAct-Setup-${version}.exe`).
- **Portable:** ejecutable autónomo (`BalanceAct-${version}-portable.exe`).
- Ambos comparten la misma carpeta de datos `%LOCALAPPDATA%/BalanceAct`.

Scripts en `package.json`:

| Script | Propósito |
| :--- | :--- |
| `dev` | Arranca Vite y Electron en modo desarrollo |
| `dev:renderer` | Arranca solo el servidor de Vite para el renderer |
| `typecheck` | Verifica tipos de renderer y main sin emitir archivos |
| `test` | Ejecuta la suite de pruebas en Vitest (`vitest run`) |
| `build` | Compila renderer, main y preload |
| `pack` | Compila y genera el paquete sin instalador (`electron-builder --dir`) |
| `dist` | Compila y genera los instaladores NSIS y portable en `release/` |
| `docs:screenshots` | Genera capturas programáticas en `docs/capturas/` |

---

## 13. Capturas programáticas

El comando `pnpm docs:screenshots` (implementado en `scripts/capture-screenshots.mjs` y `scripts/capture-screenshots-main.cjs`, siguiendo el patrón de TheirView y TracePath) realiza lo siguiente:

1. Verifica que exista la compilación en `dist-electron/`.
2. Crea un directorio temporal en `os.tmpdir()` y redirige `appData` / `userData` de Electron hacia él (nunca toca `%LOCALAPPDATA%/BalanceAct` real).
3. Siembra un perfil de ejemplo con una semilla fija y lanza la aplicación compilada; recorre las pantallas y estados relevantes (alto contraste, texto grande, carta con pistas, retroalimentación inmediata, área en riesgo, fin por extremo, semestre completo, resultados, historial y diálogos) y guarda los PNG en `docs/capturas/`.
4. Limpia el directorio temporal al finalizar.

---

## 14. Criterios de aceptación

1. Navegación completa entre las 10 pantallas sin callejones sin salida; toda pantalla secundaria cuenta con `BackButton` hacia `MENU` (o `SETTINGS` si `GUIDE` viene de allí).
2. Creación, selección y eliminación de perfiles por nombre o código de estudiante con validación de caracteres permitidos (`1..32`, letras con tildes/`ñ`, dígitos, espacios, `-` y `.`) y coincidencia sin distinguir mayúsculas.
3. Aislamiento total entre perfiles: cambiar de perfil carga sus propios ajustes e historial sin contaminar otros perfiles.
4. Reglas del juego: áreas en `0..100` empezando en `50`; cada decisión aplica su efecto acotado; la partida termina al llegar un área a `0` o `100` o al completar 16 semanas; ninguna carta se repite en una partida y el tramo corresponde a la semana.
5. Pistas de dirección visibles antes de elegir, sin magnitud; retroalimentación inmediata con magnitud por área tras elegir; zona de riesgo indicada con texto.
6. Métricas registradas por partida: decisiones (carta, opción, antes, cambio, después, tiempo), semanas jugadas, tiempo de partida sin pausas, tiempo medio por decisión, áreas cedidas y área más cedida, decisiones de reequilibrio y cierre; `RESULTS` y `RECORDS` las muestran.
7. Alto contraste (`data-contrast="high"`), tamaño de texto (`normal` / `large`), `reduceMotion` (cartas, medidores y fundido) y audio con volumen (`0..1`) se aplican al instante, se reflejan en la vista previa de `SETTINGS` y persisten tras reiniciar.
8. Checkpoint por decisión y reanudación: cerrar la aplicación a mitad de una partida y volver a abrirla ofrece `Continuar` en `MENU`, retomando con la misma carta pendiente y los mismos valores (o en `POST` si la partida ya había terminado).
9. Descartar partida en curso: pulsar `Jugar` existiendo una `in_progress` exige confirmación en diálogo modal e invoca `discardGame()`.
10. Suite de tests en Vitest cubriendo lógica pura (reglas, fin, elección de cartas con semilla, métricas, reanudación, validación de perfiles), validación del mazo, aislamiento entre perfiles y escritura atómica con lock por archivo.
11. Verificación funcional con carpeta de datos temporal en los tres escenarios: (1) desde cero, (2) con datos previos y (3) reingreso con el mismo perfil, además de `pnpm typecheck`, `pnpm test`, `pnpm build` y `pnpm docs:screenshots` en verde.

---

## 15. Decisiones tomadas

1. **Sin 3D ni zona generada por el programa:** toda la parte visual (cartas, ilustraciones, medidores, íconos, logo) la diseña el diseñador; el equipo de programación arma las pantallas, redacta el contenido de las cartas y hace que todo funcione.
2. **Ambos extremos terminan la partida:** `0` y `100` cierran la partida y se describen con el mismo tono; no hay extremo "bueno".
3. **Persistencia por decisión (`saveCheckpoint`):** se invoca al crear la partida y al aplicar cada decisión, guardando también la siguiente carta elegida.
4. **Descarte de partida en IPC (`discardGame`):** se agrega `discardGame(): Promise<GameRecord[]>` a `window.balanceActApi`; iniciar una partida nueva con otra en curso exige confirmación explícita en diálogo modal.
5. **Sin mapa de sesiones:** la partida es corta y se inicia desde `MENU`; la primera vez pasa por `GUIDE`.
6. **Vista previa en `SETTINGS`:** carta de ejemplo con los tres medidores que refleja al instante el contraste y el tamaño de texto.
7. **Ajuste de movimiento (`reduceMotion`):** afecta la entrada y salida de cartas, la animación de los medidores y el fundido de `SPLASH`, respetando además `prefers-reduced-motion`.

---

## 16. Supuestos para revisar

Decisiones tomadas sin consulta, con la opción más conservadora y pedagógicamente coherente. Cambiarlas no altera la arquitectura.

1. **Rango y valor inicial:** `0..100` enteros, inicio en `50` las tres áreas (como el prototipo y la propuesta).
2. **Extremos:** mínimo `0` y máximo `100` terminan la partida; zona de riesgo de aviso en `≤ 20` y `≥ 80`.
3. **Duración máxima:** 16 semanas (un semestre), una carta por semana; completar el semestre es el otro cierre posible.
4. **Mazo:** 24 cartas (la propuesta pedía 20 a 30) en tres tramos de 8 por intensidad; cada partida usa como máximo 16 sin repetir.
5. **Elección de carta:** azar sin repetición dentro del tramo de la semana, con peso doble para cartas que tocan un área en riesgo.
6. **Pistas antes de elegir:** solo dirección (`sube` / `baja` / `igual`), nunca magnitud; la magnitud se revela después. No se pueden desactivar.
7. **Sin cronómetro por carta:** se mide el tiempo de decisión, pero no se limita ni se muestra durante la partida.
8. **Duración medida** como semanas jugadas y tiempo activo en el juego sin pausas; además, tiempo medio por decisión.
9. **"Área descuidada"** = área que baja con la opción elegida (área cedida); la más frecuente se muestra al final y se acumula en el historial.
10. **Decisiones de reequilibrio** como indicador de anticipación, mostrado sin calificar.
11. **Autorreportes de ánimo y estrés** (1–5) al inicio y al final de cada partida, por coherencia con la familia de productos, aunque el contexto de BalanceAct no los menciona.
12. **Elección por botón, teclado o arrastre:** como en la propuesta, la carta también se puede arrastrar con el mouse hacia una opción; el arrastre es siempre opcional y con `reduceMotion` la carta no se anima.
13. **Ilustraciones:** una por carta como máximo, reutilizables entre cartas; las dibuja el diseñador.
14. **Textos de cartas y retroalimentación** redactados por el equipo de programación y pendientes de revisión del docente o cliente.
15. **Sin ajustes propios** del juego; solo los ajustes clásicos.
