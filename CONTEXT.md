# APURATE! — Contexto del Proyecto

## Qué es
PWA móvil que calcula a qué hora empezar a alistarte para llegar a tiempo a una
reunión, midiendo tráfico en vivo/previsto con Google Distance Matrix, y te avisa
cuándo pedir el taxi/Uber.
Hermana de [[taxi-vs-auto]] — reusa su design system (paleta, tipografía, patrones
de tarjeta/origen/geocoding) pero es un proyecto y repo independiente.
Uso personal de Amaro.

## Repo y deploy
- Carpeta local: `G:\Mi unidad\apurate`
- GitHub: https://github.com/amaronovoaflores/apurate
- GitHub Pages: **https://amaronovoaflores.github.io/apurate/** (activo, deploy desde `main`/root)
- Tecnología: HTML/CSS/JS puro, single file (`index.html`), sin backend — mismo
  approach que taxi-vs-auto.
- Google Maps API key propia (no la de taxi-vs-auto), creada dentro del
  proyecto de Google Cloud **"apurate"**, con **Maps JavaScript API + Places
  API + Distance Matrix API** habilitadas y restringida por referrer a
  `https://amaronovoaflores.github.io/*`. Verificado con tráfico real en vivo.

## Lógica central (planificación hacia atrás)
```
Hora de la reunión (+ toggle Hoy/Mañana, visible y editable)
  − colchón de llegada (Perfil, default 10 min)
  = Hora de llegada
  − tiempo de viaje (Distance Matrix, 2 pasadas: 1) duración base para estimar
    hora de salida, 2) reconsulta con drivingOptions.departureTime en esa hora
    estimada para traer tráfico previsto real — trafficModel va en minúscula:
    'bestguess'/'pessimistic'/'optimistic')
  = Hora de salir de casa (== hora en que te recoge el taxi, si modo Taxi)
  − espera de taxi (Perfil, default 7 min) · SOLO si modo == Taxi
  = Hora de pedir el taxi/Uber (fin del checklist)
  − suma de tareas del checklist (Perfil: ducha, vestirse, maquillaje...)
  = HORA DE EMPEZAR A ALISTARTE ⏰ (dato principal que muestra la app)
```
En modo **Auto** se salta el paso de espera de taxi — el timeline pasa de 5 a
4 pasos y `prepStartTime` se calcula directo desde `departureTime`.

Si `prepStartTime` queda en el pasado (no alcanza el tiempo), se muestra una
tarjeta de advertencia roja arriba de todo con el mensaje según qué tan grave
es (aún se puede recortar checklist / ya debiste pedir el taxi / ya debiste
salir) — el plan se sigue mostrando igual, no se bloquea el cálculo.

Un `setInterval` de 1s recalcula la fase actual (tranquilo / alístate / pide
tu taxi / sal ya / en camino / en la reunión) y dispara `Notification` +
vibración en cada transición, mientras la pestaña siga abierta.

## Estructura de la pantalla principal (tras la reorganización del 10 ago)
1. Header: ícono + título + saludo con nombre + botón 👤 Perfil + reloj en vivo
2. Card "Tu reunión": hora + toggle Hoy/Mañana + dirección
3. Card "Sales desde": GPS/manual
4. Card "Vas en": toggle Auto/Taxi
5. Botón Calcular
6. Resultados, en este orden: tarjeta de advertencia (si no alcanza el
   tiempo) → tarjeta "¿Pides tu taxi/Uber?" con 2 números grandes (hora de
   pedido y hora en que te recoge, solo modo Taxi) → banner de estado en
   vivo → timeline completo → botones Pedir Uber/Cabify (copian la
   dirección de destino al portapapeles como respaldo, aunque normalmente
   Uber la trae precargada del deep link)

**Todo lo demás vive en el modal de Perfil** (botón 👤): nombre, colchón de
llegada, espera de taxi, modelo de tráfico, y el checklist completo de
tareas (editable, con "restaurar" a los defaults). Se configura una vez —
la primera vez que se abre la app, el modal aparece automático (onboarding)
y no se puede cerrar sin guardar. Después no vuelve a interrumpir, solo se
reabre tocando el ícono.

## Decisiones de diseño explícitas (para no repreguntar)
- **Sin integración a Google Calendar** — entrada de hora/dirección manual,
  cada vez.
- **Sin auto-reserva de Uber**: técnicamente no existe forma de agendar un
  viaje futuro vía deep link externo — Uber solo permite "pedir ahora". La
  app en cambio avisa (notificación + vibración) en el momento exacto para
  que el usuario lo pida él mismo con un toque. Tampoco se puede abrir la
  app de Uber sin un toque del usuario (restricción de seguridad de los
  navegadores, no evitable).
- **Un solo campo "espera de taxi"** (no uno separado para Uber vs taxi de
  calle) — el modo es genérico Auto/Taxi, no distingue plataforma.

## Reglas de diseño
- Mobile-first — el 100% del uso es en celular
- Sin frameworks externos — HTML/CSS/JS vanilla puro
- Mismo patrón de geocoding (Google Places con fallback a Nominatim) y de
  deep links a Uber/Cabify que taxi-vs-auto, ambos con timeout de 4s para no
  colgar la UI si Google no responde

## Rediseño visual "hotline" (10 ago 2026, noche)
Se aplicó un refresco visual completo via handoff de diseño (paquete
`Refrescador visual para PWA.zip`, ver `design-brief.md` para las
restricciones de arquitectura que se le dieron al diseñador). Cambios:
- **Paleta nueva**: `--bg:#0D0B14`, `--red:#FF3B6B` (marca/urgencia,
  reemplaza al rojo anterior), `--yellow:#FFCE45`, `--green:#00E5A0`,
  `--blue:#4CC9F0` (toggles de selección Hoy/Mañana=rojo,
  GPS/Escribir y Taxi/Auto=celeste — sí, el toggle de día usa rojo y los
  otros dos celeste, es intencional del handoff)
- **Set de íconos SVG inline** (`ICON_PATHS`/`iconSvg()` en el JS)
  reemplazando todos los emojis — sin dependencias externas, se pintan al
  cargar via `data-icon="nombre,tamaño,color"` + `paintStaticIcons()`, o
  generados inline en JS para contenido dinámico (checklist, timeline)
- **Nueva barra sticky `#hero-times`**: 3 horas clave siempre visibles al
  scrollear los resultados (Reunión / Alístate desde / Pide el taxi o
  Sales de casa según modo) — puramente visual, lee del mismo `plan` que
  ya se calculaba, no tocó lógica
- Timeline con puntos conectados por línea vertical, checklist con
  checkbox custom (checkmark SVG, tachado si está desactivada), modal de
  perfil rediseñado tipo bottom sheet, botón Calcular con spinner animado,
  confirm-card ya no se oculta al confirmar (queda visible con check verde)
- **Decisiones tomadas al aplicar el handoff** (el diseño de referencia
  traía cosas que no se llevaron literal):
  - Se mantuvo español **tú** (Lima) — el prototipo de referencia usaba
    voseo argentino ("Salís", "Pedí"), se ignoró esa parte y solo se tomó
    lo visual
  - El `<select>` de modelo de tráfico mantiene sus `value` reales
    (`bestguess`/`pessimistic`/`optimistic`, los que acepta la API de
    Google) en vez de las etiquetas inventadas del demo
    (`mejor`/`promedio`/`pesimista`) que habrían roto el cálculo
  - El selector de fases del prototipo (6 chips para previsualizar
    manualmente cada fase) **no se llevó** — es una ayuda del demo, en la
    app real el banner cambia de fase solo por tiempo real
- Todos los ids del DOM y funciones JS se preservaron intactos — cero
  cambios de lógica/cálculo, solo visual. Verificado en vivo con Distance
  Matrix real funcionando (`con tráfico en vivo`) sobre el nuevo diseño.
- Pendiente opcional: los íconos de instalación PWA (`icon-192.png`,
  `icon-512.png`, `apple-touch-icon.png`) siguen con la paleta vieja
  (rojo/amarillo + reloj) — no se tocaron porque el handoff no los incluía,
  se podrían regenerar a juego con el ícono de rayo nuevo si se quiere
  consistencia total.

## Troubleshooting de la API key (por si se repite en otro proyecto)
Costó varias vueltas — dejar constancia:
1. "Habilitar la API" (Library) y "restringir la key a esa API" (Credentials)
   son dos pasos independientes en Cloud Console — falta uno y sigue
   fallando con `ApiNotActivatedMapError` aunque el otro esté bien.
2. La key se había creado sin querer en un proyecto de GCP distinto a
   "apurate" — activar las APIs en "apurate" no le llegaba a esa key. La
   solución fue crear la key *dentro* del proyecto correcto en vez de
   perseguir en cuál había quedado la vieja.
3. Bug real de paso: `trafficModel` se mandaba como `'bestGuess'` (camelCase)
   pero `google.maps.TrafficModel` solo acepta minúsculas (`'bestguess'`) —
   quedaba tapado por el fallback silencioso hasta que la key por fin
   autenticó y se destapó el error. Ya corregido.
4. Cambiar restricciones de API keys es una acción de seguridad de cuenta
   que Claude tiene bloqueada por política, incluso con permiso explícito —
   Amaro tiene que hacerlo él mismo en Cloud Console.

## ⚠️ Pendiente
- Probar en campo con reunión real (celular, GPS real, tráfico real en el
  momento) — todo lo verificado hasta ahora fue con datos de prueba en
  navegador de escritorio.
- Ajustar defaults del checklist/tiempos si en el uso real se sienten
  cortos o largos.

## Cómo trabajar
- Editar `index.html` en `G:\Mi unidad\apurate`
- `cd "G:\Mi unidad\apurate" && git add -A && git commit -m "..." && git push`
  — GitHub Pages se actualiza solo en ~1 min
- Servidor local de prueba: configuración `apurate` en
  `../taxi-vs-auto/.claude/launch.json` (puerto 8843, sirve este folder vía
  `python -m http.server --directory`) — ojo, la API key está restringida al
  dominio real, así que en localhost siempre cae al fallback (Nominatim +
  distancia estimada), es normal y no indica un bug.
