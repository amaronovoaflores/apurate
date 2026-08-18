# APURATE! — Contexto del Proyecto

## Qué es
PWA móvil que calcula a qué hora empezar a alistarte para llegar a tiempo a una
reunión, midiendo tráfico en vivo/previsto con Google Distance Matrix (o
transporte público para Bus), y te avisa cuándo pedir el taxi.
Hermana de [[taxi-vs-auto]] — reusa su design system pero es un proyecto y repo
independiente. Uso familiar (Amaro y su hija).

## Repo y deploy
- Carpeta local: `G:\Mi unidad\apurate`
- GitHub: https://github.com/amaronovoaflores/apurate
- GitHub Pages: **https://amaronovoaflores.github.io/apurate/**
- Tecnología: HTML/CSS/JS puro, single file (`index.html`), sin backend.
- Google Maps API key propia, proyecto GCP **"apurate"**, con **Maps
  JavaScript + Places + Distance Matrix** habilitadas, restringida por
  referrer a `https://amaronovoaflores.github.io/*`.
- **⚠️ El CDN de GitHub Pages a veces tarda mucho más de lo normal en
  propagar** (una vez tardó +25 min en vez de los ~1 min habituales). Si
  después de un push la web sigue sirviendo la versión vieja tras varios
  minutos, confirmar primero que el repo sí tiene el cambio
  (`fetch('https://raw.githubusercontent.com/amaronovoaflores/apurate/main/index.html',{cache:'no-store'})`
  — bypassea Pages por completo) y que el Action `pages-build-deployment`
  terminó bien en github.com/amaronovoaflores/apurate/actions. Si el repo
  está bien pero Pages sigue atascado, un commit vacío
  (`git commit --allow-empty -m "..." && git push`) para forzar un redeploy
  nuevo lo destrabó la última vez.

## Lógica central (planificación hacia atrás)
```
Hora de la reunión (+ toggle Hoy/Mañana)
  − colchón de llegada (Perfil, default 10 min)
  = Hora de llegada
  − tiempo de viaje:
      Taxi/Auto → Distance Matrix DRIVING, 2 pasadas (base + refinar con
        drivingOptions.departureTime/trafficModel — valores en minúscula:
        'bestguess'/'pessimistic'/'optimistic')
      Bus → Distance Matrix TRANSIT, 1 sola pasada (ya incluye caminata +
        espera + viaje según horario de Google, no acepta drivingOptions)
  = Hora de salir de casa / al paradero
  − espera de movilidad (Perfil, default 7 min) · aplica a Taxi Y Bus
    (Auto no la necesita, el auto ya está esperándote)
  = Hora de pedir el taxi (solo Taxi tiene este paso explícito con
    confirm-card; en Bus el margen se resta igual pero sin una tarjeta de
    "pedir", porque no hay nada que pedir)
  − suma de tareas del checklist (Perfil)
  = HORA DE EMPEZAR A ALISTARTE ⏰
```
Modos: **Taxi** (5 pasos, confirm-card, botones Uber/Cabify) · **Auto** (4
pasos, sin espera) · **Bus** (4 pasos, con espera, ícono/badge propios
"según horario de buses de Google Maps").

Si `prepStartTime` queda en el pasado (no alcanza el tiempo), tarjeta de
advertencia roja arriba de todo — el plan se sigue mostrando igual.

## Alarmas / notificaciones — cómo funcionan y sus límites
`tick()` corre cada 1s (mientras la pestaña esté abierta) y dispara
`Notification` + vibración en cada cambio de fase (empieza a alistarte →
pide tu taxi [solo Taxi] → sal ya), no solo al inicio.

**Limitaciones reales, no evitables sin backend propio:**
- Solo suena **con la app abierta** (pantalla prendida). No es una alarma
  nativa — si se cierra la app o se bloquea el celular, el `setInterval` dejar
  de correr y no hay forma de que avise. Arreglar esto de verdad requeriría
  Push API + VAPID + un servidor que dispare el push a la hora exacta —
  fuera de alcance del approach actual (estático, sin backend).
- **iPhone**: Safari solo permite notificaciones si la PWA está instalada en
  pantalla de inicio y se abre desde ese ícono, no desde una pestaña normal
  de Safari.
- El permiso de notificación se pide en **todo cálculo exitoso** (`calcular()`,
  línea con el comentario "en TODO cálculo exitoso") — antes solo se pedía
  al tocar "Sí, avísame" de la tarjeta de Taxi, así que en Auto/Bus o con la
  agenda automática nunca se pedía y las alarmas se quedaban en silencio sin
  avisar del error. Ya corregido (10 ago 2026, noche).

## Agenda semanal (Perfil → Agenda semanal)
Toggle activar/desactivar + 7 días (Lun-Dom), cada uno con hora + destino +
modo. **No es por fecha, es por día de la semana — se repite cada semana
para siempre**, sin botón de "repetir" aparte (eso ya es el comportamiento
por defecto). Con la agenda activada, `initAgendaAutoRun()` corre en el
`Init` de la página: si hoy tiene un día activo, precarga esos datos, espera
el GPS si hace falta, y llama `calcular()` sola — sin que nadie toque nada.
Datos en `localStorage` clave `apurate_agenda_v1`, separado de
`apurate_profile_v1`.

## Estructura de la pantalla principal
1. Header: ícono + título + saludo + botón 👤 Perfil + reloj en vivo
2. Card "Tu reunión": hora + toggle Hoy/Mañana + dirección
3. Card "Sales desde": GPS/manual
4. Card "Vas en": toggle **Taxi/Uber · Auto · Bus** (3 vías)
5. Botón Calcular (spinner mientras calcula)
6. Resultados: barra sticky `#hero-times` (3 horas clave, siempre visible al
   scrollear) → advertencia si no alcanza el tiempo → confirm-card "¿Pides
   tu taxi?" (solo Taxi) → banner de estado en vivo → timeline → botones
   Uber/Cabify (solo Taxi, copian dirección al portapapeles como respaldo)

**Todo lo demás vive en Perfil** (botón 👤): nombre, colchón de llegada,
espera de movilidad, modelo de tráfico (con pista explicando qué es), el
checklist, y la agenda semanal. Se configura una vez — onboarding
obligatorio la primera vez, después se reabre tocando el ícono.

## Decisiones de diseño explícitas (para no repreguntar)
- Sin integración a Google Calendar — la agenda semanal cubre el caso de uso
  recurrente sin necesitar esa integración.
- Sin auto-reserva de Uber — no existe esa función vía deep link externo, y
  tampoco se puede abrir la app de Uber sin un toque del usuario (restricción
  del navegador).
- Un solo campo de espera ("espera de movilidad") compartido entre Taxi y
  Bus, no uno por plataforma.
- Español **tú** (Lima), no voseo — importante recordarlo si se vuelve a usar
  un handoff de diseño externo (uno anterior traía voseo argentino y se
  ignoró esa parte).

## Reglas de diseño ("hotline", desde el rediseño del 10 ago 2026)
- Mobile-first, sin frameworks, un solo archivo.
- Paleta: `--bg:#0D0B14`, `--red:#FF3B6B` (marca/urgencia), `--yellow:#FFCE45`,
  `--green:#00E5A0`, `--blue:#4CC9F0`. Toggle Hoy/Mañana = rojo activo, resto
  de toggles = celeste activo (intencional).
- Set de íconos SVG inline (`ICON_PATHS`/`iconSvg()`) — cero dependencias
  externas de íconos. Estáticos se pintan con `data-icon="nombre,tam,color"` +
  `paintStaticIcons()`; dinámicos (checklist, timeline, toast) se generan
  inline en JS.
- Mismo patrón de geocoding (Google Places con fallback a Nominatim) con
  timeout de 4s para no colgar la UI si Google no responde.
- Íconos de instalación PWA (`icon-192.png`, `icon-512.png`,
  `apple-touch-icon.png`) a juego con la paleta — generados con PowerShell +
  System.Drawing.

## Troubleshooting de la API key (por si se repite en otro proyecto)
1. "Habilitar la API" (Library) y "restringir la key a esa API" (Credentials)
   son dos pasos independientes — falta uno y sigue fallando con
   `ApiNotActivatedMapError` aunque el otro esté bien.
2. La key puede crearse sin querer en un proyecto de GCP distinto al
   esperado — activar APIs ahí no le llega a una key de otro proyecto. Crear
   la key *dentro* del proyecto correcto es más simple que perseguir dónde
   quedó la vieja.
3. `trafficModel` va en minúscula (`'bestguess'`), no camelCase — Google lo
   rechaza si no.
4. Cambiar restricciones de API keys es una acción de seguridad de cuenta
   que Claude tiene bloqueada por política — Amaro tiene que hacerlo él mismo.

## ⚠️ Pendiente / por confirmar con uso real
- Confirmar con la hija si usa iPhone o Android (afecta si necesita instalar
  la PWA en pantalla de inicio para que las notificaciones funcionen).
- Validar en campo que el fix de permiso de notificaciones (10 ago noche)
  resolvió el caso real donde la alarma no sonó el primer día que se probó
  la agenda semanal.
- Ajustar defaults del checklist/tiempos si en el uso real se sienten cortos
  o largos.

## Cómo trabajar
- Editar `index.html` en `G:\Mi unidad\apurate`
- `cd "G:\Mi unidad\apurate" && git add -A && git commit -m "..." && git push`
- Servidor local de prueba: configuración `apurate` en
  `../taxi-vs-auto/.claude/launch.json` (puerto 8843) — la key está
  restringida al dominio real, así que en localhost siempre cae al fallback
  (Nominatim + distancia estimada), es normal.
