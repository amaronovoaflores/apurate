# APURATE! — Contexto del Proyecto

## Qué es
PWA móvil que calcula a qué hora empezar a alistarte para llegar a tiempo a una
reunión, midiendo tráfico en vivo/previsto con Google Distance Matrix, y te avisa
cuándo pedir el Uber/Cabify.
Hermana de [[taxi-vs-auto]] — reusa su design system (paleta, tipografía, patrones
de tarjeta/origen/geocoding) pero es un proyecto y repo independiente.
Uso personal de Amaro.

## Lógica central (planificación hacia atrás)
```
Hora de la reunión
  − colchón de llegada (input, default 10 min)
  = Hora de llegada
  − tiempo de viaje (Distance Matrix, 2 pasadas: 1) duración base para estimar
    hora de salida, 2) reconsulta con drivingOptions.departureTime en esa hora
    estimada para traer tráfico previsto real)
  = Hora de salir de casa
  − espera del Uber pedido (input, default 5 min)
  = Hora de pedir el Uber (fin del checklist)
  − suma de tareas del checklist (ducha, vestirse, maquillaje... editable)
  = HORA DE EMPEZAR A ALISTARTE ⏰ (dato principal que muestra la app)
```
Un `setInterval` de 1s recalcula la fase actual (tranquilo / alístate / pide tu
Uber / sal ya / en camino / en la reunión) y dispara `Notification` + vibración
en cada transición, mientras la pestaña siga abierta.

## Repo y deploy
- Carpeta local: `G:\Mi unidad\apurate`
- GitHub: https://github.com/amaronovoaflores/apurate
- GitHub Pages: https://amaronovoaflores.github.io/apurate/ (activo, deploy desde `main`/root)
- Tecnología: HTML/CSS/JS puro, single file (`index.html`), sin backend — mismo
  approach que taxi-vs-auto.

## Estado actual (10 ago 2026)
MVP construido, verificado en navegador y **publicado en GitHub Pages**.
Formulario de reunión (solo hora + dirección), origen GPS/manual, selector
**Auto / Taxi**, checklist editable de tareas, **Perfil** (nombre + colchón
de llegada + espera de taxi 7min + modelo de tráfico) con onboarding la
primera vez, cálculo con fallback a Nominatim/haversine, timeline de
resultados (5 pasos en Taxi, 4 en Auto — sin espera de taxi), banner de
estado con la hora clave en grande y countdown chico debajo, tarjeta de
confirmación "¿te aviso para pedir tu taxi?" que arma la notificación
justo a tiempo (Uber no soporta reserva por deep link externo, así que no
se promete auto-reserva), botones Uber/Cabify que copian la dirección al
portapapeles antes de abrir la app. Falta uso real en campo (celular, GPS
real, reunión real).

## ✅ Resuelto (10 ago 2026, tarde)
### API key propia, en el proyecto correcto, con las 3 APIs habilitadas
apurate ya no reusa la key de taxi-vs-auto — tiene su propia key
(`AIzaSyADGCtGld68ZHWeAonBALzNRZL79Lhafk8`) creada dentro del proyecto de
Google Cloud **"apurate"**, con **Maps JavaScript API + Places API +
Distance Matrix API** habilitadas y restringida por referrer a
`https://amaronovoaflores.github.io/*`. Verificado en vivo: Distance Matrix
responde `status=OK` con datos reales y el badge de la app muestra
"🟢 Con tráfico previsto".

Troubleshooting real que costó varias vueltas, por si se repite en otro
proyecto: (1) "habilitar la API" en Library y "restringir la key a esa API"
en Credentials son dos pasos independientes — falta uno y sigue fallando
con `ApiNotActivatedMapError` aunque el otro esté bien; (2) la key nueva se
había creado sin querer en un proyecto de GCP distinto a "apurate", así que
activar las APIs en "apurate" no le llegaba a esa key — la solución fue
crear la key *dentro* del proyecto correcto en vez de perseguir en cuál
había quedado la vieja; (3) de paso se encontró y arregló un bug real en
[apurate/index.html](index.html): `trafficModel` se mandaba como `'bestGuess'`
(camelCase) pero `google.maps.TrafficModel` solo acepta minúsculas
(`'bestguess'`) — quedaba tapado por el fallback silencioso hasta que la key
por fin autenticó y se destapó el error.

### 2. Probar en campo con reunión real
El cálculo se verificó con datos de prueba (no un caso real con tráfico en
vivo) — confirmar que las horas resultantes se sienten correctas en el uso
diario y ajustar los defaults del checklist si hace falta.

## Reglas de diseño (heredadas de taxi-vs-auto)
- Mobile-first — el 100% del uso es en celular
- Paleta: `--bg:#0B0B14`, acentos `--yellow`, `--green`, `--blue`, `--red`,
  usando `--red` como color principal de marca (urgencia) en vez de `--yellow`
- Sin frameworks externos — HTML/CSS/JS vanilla puro
- Mismo patrón de geocoding (Google Places con fallback a Nominatim) y de
  deep links a Uber/Cabify que taxi-vs-auto, ambos con timeout de 4s para no
  colgar la UI si Google no responde (bug encontrado y corregido durante el
  build: antes el callback de Places/DistanceMatrix podía no dispararse nunca
  si la key estaba bloqueada, dejando "Calculando..." pegado para siempre)

## Cómo trabajar
- Editar `index.html` en `G:\Mi unidad\apurate`
- Servidor local de prueba: configuración `apurate` en
  `../taxi-vs-auto/.claude/launch.json` (puerto 8843, sirve este folder vía
  `python -m http.server --directory`)
