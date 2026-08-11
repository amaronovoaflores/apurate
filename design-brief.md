# Brief de diseño — APURATE!

**Para:** mejora visual (UI/UX) de la PWA ya construida y en producción.
**No se pide:** cambiar funcionalidad, flujo, ni arquitectura técnica — solo cómo se ve.
**App en vivo:** https://amaronovoaflores.github.io/apurate/
**Repo:** https://github.com/amaronovoaflores/apurate (archivo único: `index.html`)

---

## 1. Qué es la app

APURATE! calcula a qué hora una persona debe empezar a alistarse para llegar
a tiempo a una reunión, contando hacia atrás desde la hora de la reunión:
tiempo de viaje con tráfico real (Google Maps) + espera de un taxi/Uber (si
aplica) + una lista personal de tareas de arreglo (ducharse, vestirse,
maquillarse, etc.). Uso 100% en celular, en el momento de apuro real —
tiene que leerse rápido y de un vistazo, no es una app para "explorar".

Es hermana de otra PWA personal del mismo usuario (`taxi-vs-auto`, compara
costo de taxi vs. auto propio) y comparte su lenguaje visual base, pero es
un proyecto independiente — no hace falta mantenerlos visualmente idénticos,
solo coherentes en espíritu (dark UI, mobile-first, sin adornos).

---

## 2. Restricción técnica dura (leer antes de tocar nada)

**Es un solo archivo `index.html`** — HTML + CSS + JS inline, sin build
step, sin framework, sin bundler. El JavaScript manipula el DOM por
`document.getElementById(...)` en decenas de sitios. Cualquier rediseño
tiene que:

- **Mantener exactamente los mismos `id`** de los elementos (ver lista
  completa en la sección 5) — si se renombra un id sin actualizar el JS que
  lo referencia, la función asociada se rompe en silencio.
- **Mantener el árbol de elementos que el JS crea/lee dinámicamente**:
  `#checklist` (el JS le inyecta filas con `innerHTML`), `#timeline` (ídem),
  `#toast`, y las clases de estado que el JS alterna con `classList`
  (`.active`, `.show`, `.ready`, `.confirmed`, y las 6 clases de fase del
  banner: `st-wait / st-prep / st-uber / st-go / st-road / st-done`).
- **Puede cambiarse libremente**: toda la paleta de colores, tipografía,
  espaciados, bordes, sombras, animaciones, iconografía (hoy son emojis),
  orden visual dentro de una misma sección, y el CSS en general — mientras
  no se borren los ids/clases que el JS usa.
- Puede agregarse CSS nuevo, clases nuevas, wrappers nuevos — el límite es
  no *quitar* ni *renombrar* lo que ya existe sin coordinarlo.
- Sigue debiendo funcionar **sin conexión a un backend** (todo el estado
  vive en `localStorage`, dos claves: `apurate_v1` y `apurate_profile_v1`)
  y **sin build tools** — el archivo final tiene que seguir siendo HTML
  plano que un navegador abre directo.
- Fuentes/librerías externas actuales: Google Fonts (Inter) y Google Maps
  JS API. No agregar más dependencias externas sin avisar (cada una es un
  punto de fallo/latencia en una app que se usa apurado).

**Si el rediseño necesita romper algún id o restructurar el DOM de forma
que afecte al JS, que lo señale explícitamente en vez de hacerlo callado**
— se coordina el ajuste del JS en paralelo, no es un bloqueo, solo no puede
pasar desapercibido.

---

## 3. Sistema de diseño actual (punto de partida)

```css
--bg:      #0B0B14   /* fondo general */
--bg2:     #17171F   /* tarjetas */
--bg3:     #1F1F2B   /* inputs, chips internos */
--border:  #2E2E40
--text:    #F5F5FF
--text2:   #B4B4D4
--text3:   #8C8CB0
--yellow:  #F5C842   /* acento secundario / CTA de confirmación */
--green:   #4ADE80   /* éxito / confirmado / en camino */
--blue:    #38BDF8   /* estado neutro / "todavía tranquilo" */
--red:     #F87171   /* marca principal — urgencia (APURATE! usa rojo, no amarillo, a diferencia de taxi-vs-auto) */
```

- Tipografía: **Inter** (400/500/600/700/800/900), sin serif en ningún lado.
- Radios: 14px tarjetas, 9px controles chicos.
- Un solo layout: columna centrada, `max-width: 420px` — pensado para verse
  igual de bien en escritorio (poco uso) que en celular (uso real).
- Sin modo claro — solo dark. (Abierto a agregar `prefers-color-scheme` si
  se quiere, no existe hoy.)
- Botón primario: gradiente rojo, sombra suave, texto blanco, 900 weight.
- Tarjetas: fondo `--bg2`, borde 1px `--border`, radio 14px, padding
  ~14-16px — patrón repetido en todas las secciones.
- Toggles de dos opciones (Hoy/Mañana, Auto/Taxi, GPS/Escribir): pill con
  fondo `--bg3`, la opción activa se resalta con color + borde.
- Iconografía: **emojis nativos**, no hay set de íconos SVG. Es deliberado
  (cero dependencias, funciona en cualquier dispositivo) pero es la parte
  más "genérica" visualmente — probablemente el mayor espacio de mejora.

---

## 4. Inventario de pantallas/estados a rediseñar

### 4.1 Pantalla principal (siempre visible)
1. Header: ícono app + título "APURATE!" + saludo con nombre (dinámico,
   ej. "Hola, Amaro 👋") + botón redondo de perfil (👤) + reloj en vivo
   (hh:mm, actualiza cada segundo).
2. Card "Tu reunión": input de hora + **toggle Hoy/Mañana** al costado +
   input de dirección (con un "dot" de estado: neutro/cargando/ok/error).
3. Card "Sales desde": toggle GPS / Escribir dirección manual + bloque de
   estado GPS (nombre del lugar detectado, coordenadas, botón reintentar
   si falla).
4. Card "Vas en": toggle Taxi/Uber vs. Mi auto.
5. Botón "Calcular →" (grande, ancho completo, estado disabled mientras
   calcula, texto cambia a "Calculando...").
6. Mensaje de error inline (rojo, aparece solo si algo falta).

### 4.2 Modal de Perfil (overlay desde abajo, tipo bottom sheet)
Se abre solo (obligatorio, sin botón cerrar) la primera vez que se usa la
app — "onboarding". Después se reabre a demanda tocando el ícono 👤.
Contiene: nombre, colchón de llegada (min), espera de taxi (min), modelo
de tráfico (select), y **el checklist completo** de tareas de arreglo
(cada fila: checkbox + nombre editable + minutos + botón borrar; más botón
"agregar tarea" y total en minutos). Botón Guardar grande al final.

### 4.3 Resultados (aparecen debajo del botón Calcular tras calcular)
En este orden — **el orden es intencional, no cambiarlo sin avisar**:

a. **Tarjeta de advertencia** (solo si no alcanza el tiempo) — fondo/borde
   rojo, ⚠️, explica cuánto tiempo falta.
b. **Tarjeta "¿Pides tu taxi/Uber?"** (solo modo Taxi) — el elemento más
   importante de decisión: dos números GRANDES lado a lado (hora de pedido
   / hora en que te recoge), texto de contexto, botón "Sí, avísame" que al
   tocarlo cambia la tarjeta a estado "confirmado" (borde verde).
c. **Banner de estado en vivo** — cambia de color/texto/animación cada
   segundo según la fase (6 fases, ver `st-*` arriba): número grande = la
   próxima hora clave, texto chico debajo = cuenta regresiva. En las fases
   urgentes (pedir taxi / salir ya) pulsa visualmente.
d. **Timeline** — lista vertical de 4-5 hitos (según modo) con ícono +
   etiqueta + subtítulo + hora, la fila "empieza a alistarte" resaltada en
   amarillo. Termina con un badge chico "🟢 con tráfico en vivo" o
   "🔵 estimado".
e. **Botones Pedir Uber / Pedir Cabify** (solo modo Taxi) — se ponen en
   estado "ready" (borde rojo pulsante) cuando es el momento.
f. Nota chica de disclaimer.

### 4.4 Toast
Notificación flotante chica abajo, aparece 2.5s (ej. "📋 Dirección
copiada"), es el único elemento que no vive dentro del flujo normal.

---

## 5. IDs que el JS depende de ellos (no renombrar sin coordinar)

```
app-sub, live-clock, hora-reunion, day-hoy, day-manana, destino, dest-dot,
otab-gps, otab-manual, gps-block, loc-name, loc-sub, loc-retry,
origen-manual, mtab-taxi, mtab-auto, calc-btn, err-msg, results,
warn-card, warn-sub, confirm-card, confirm-title, confirm-pedido,
confirm-llegada, confirm-sub, confirm-btn, status-banner, status-tag,
status-title, status-detail, timeline, action-row, btn-uber, btn-cabify,
profile-overlay, profile-title, profile-sub, prof-name, prof-buffer,
prof-espera, prof-traffic, checklist, cl-total-val, profile-close-btn,
toast
```

Clases que el JS alterna dinámicamente (mantener su significado, el
nombre literal de la clase puede cambiar si se actualiza el JS a la par):
`.active` (toggles), `.show` (resultados/modal), `.ready` (botones de
pedir taxi listos), `.confirmed` (confirm-card tras aceptar), `.st-wait
.st-prep .st-uber .st-go .st-road .st-done` (fases del banner), `.fdot
.ok/.loading/.error` (estado del input de dirección).

---

## 6. Qué NO tocar (fuera de alcance de este brief)

- Lógica de cálculo, orden de las restas, nombres de funciones JS.
- Los textos/copys en español — se puede pulir redacción si mejora
  claridad, pero no cambiar el tono (directo, urgente, sin formalismos).
- Integraciones externas (Google Maps, Nominatim, deep links de
  Uber/Cabify, Service Worker/manifest.json para PWA).
- El hecho de que sea un solo archivo sin build step.

## 7. Dónde sí hay espacio real para mejorar (pistas, no mandato)

- **Iconografía**: hoy son emojis crudos (🚕🧖🚪📍🗓️). Un set visual más
  cuidado (SVG inline, sin dependencias externas) le daría otro nivel sin
  romper la regla de "cero dependencias".
- **Jerarquía en los resultados**: son 5-6 bloques apilados, todos tarjetas
  con bordes — podría explorarse más contraste entre "lo urgente" (banner,
  confirm-card) y "lo informativo" (timeline) para que el ojo vaya directo
  a lo accionable.
- **Las 6 fases del banner de estado** hoy se diferencian solo por color de
  borde/fondo y una animación de pulso — hay espacio para micro-
  interacciones o transiciones más claras entre fases.
- **Checklist dentro del modal de Perfil**: es una lista de inputs bastante
  utilitaria, podría sentirse menos "formulario de configuración" y más
  ligera de editar.
- **Onboarding**: hoy es el mismo modal de Perfil con un título distinto —
  podría tener más carácter de bienvenida sin volverse largo (se usa una
  vez).
- **Estados vacíos/de carga**: el "Calculando..." del botón y el "dot" de
  estado del input de dirección son minimalistas al extremo — margen para
  algo de personalidad sin agregar peso.

---

## 8. Formato de entrega esperado

Como es un archivo único sin build, lo más directo es recibir el
`index.html` completo modificado (o un diff/parche claro), no assets
sueltos ni un design system separado. Si se proponen fuentes o íconos
nuevos, que sean auto-contenidos (inline SVG, o @font-face con archivo
incluido) — nada que dependa de un CDN nuevo sin avisar antes.
