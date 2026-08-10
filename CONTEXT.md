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

## ⚠️ Pendiente antes de usar en producción
### 1. Autorizar el dominio en Google Cloud Console (pendiente — Amaro debe hacerlo)
Reusa la misma API key hardcodeada de taxi-vs-auto
(`AIzaSyBjOBgEngBzLZh1yiF1bkq4C-KtdiILHCI`). Si esa key está restringida por
HTTP referrer solo al path de taxi-vs-auto, hay que ampliar la restricción a
`https://amaronovoaflores.github.io/*` (o crear una key nueva) para que
funcione en el dominio/path de apurate. Cambiar restricciones de API keys es
una acción de seguridad de cuenta que Claude tiene bloqueada por política —
Amaro debe hacerlo él mismo en Cloud Console → Credentials. Sin esto, la app
sigue funcionando (cae a Nominatim + estimado por línea recta) pero sin
tráfico real.
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
