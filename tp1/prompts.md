# Prompts para Canvas — versión consolidada en 4 pasos

Los 4 prompts se pegan en orden, en una misma conversación de Canvas (sin resetear entre uno y otro).

---

## Prompt 1 — El juego base

Construí un juego de tiro al blanco tipo "caza patos" en una galería de feria.

Estructura:
- `<header>` con el título "Tiro al Blanco — 3 patos para ganar" y una fila de
  5 `<div class="intento-indicador">` (uno por intento disponible, se van
  marcando como "usado" a medida que se consumen).
- `<main>` con:
  - un `<section id="galeria">` de 700x450px, fondo de feria (ver estilo),
    donde se mueven 4 `<div class="pato">` con emoji 🦆, cada uno rebotando
    en direcciones aleatorias y a su propia velocidad, distinta entre sí
    (sorteada al azar entre 1 y 4 px por frame).
  - superpuesta a la galería, una `<div id="mira">` que sigue al cursor del
    mouse dentro del área (un círculo con cruz tipo mira telescópica de
    rifle), visible solo mientras el mouse está sobre la galería.
  - debajo, un contador "Disparos en este intento: X/5 — Patos volteados: Y/3".
- `<footer>` con un botón "Reiniciar todo".

Estilo:
- Estética de feria/parque de diversiones: fondo con rayas diagonales rojo
  y blanco (como una carpa de circo), bordes redondeados y gruesos color
  dorado, tipografía tipo cartel de feria (bold, mayúsculas en títulos).
  Paleta: fondo #fff5f0, acento #d62828 (rojo), acento secundario #fcbf49
  (dorado), texto #1d1d1d.
- La mira: círculo de 40px con una cruz fina en el centro, borde blanco con
  sombra oscura para que se vea sobre cualquier fondo.
- Los indicadores de intento: 5 círculos pequeños en fila, dorados cuando
  están disponibles, grises y tachados cuando ya se usaron.

Comportamiento:
- Estado: `shots` (número, inicial 0, tope 5), `ducksHit` (número, inicial
  0, tope 3), `ducks` (array de {x, y, velocidadX, velocidadY, vivo}, cada
  uno con velocidad propia).
- Los patos se mueven con requestAnimationFrame, rebotando en los bordes
  del área, solo mientras `vivo` es true.
- La mira sigue al mouse dentro de la galería con un listener de
  `mousemove`.
- Al hacer click dentro de la galería: se dispara una pelotita animada
  desde abajo del área hacia la posición donde estaba la mira en ese
  instante. `shots` se incrementa en 1. Si la pelotita coincide con la
  posición de algún pato vivo en ese instante (radio de colisión simple),
  ese pato pasa a `vivo: false` (desaparece con una animación corta) y
  `ducksHit` se incrementa.
- Mientras la pelotita está en vuelo (desde el click hasta que termina su
  animación, ~300ms), los clicks nuevos no hacen nada — hay que esperar a
  que termine cada disparo antes del siguiente.

Constraints:
- Un solo archivo HTML, con el CSS en un `<style>` y el JS en un `<script>`.
- Vanilla JS, sin frameworks ni dependencias externas.
- Los patos, la pelotita y la mira son elementos del DOM posicionados con
  CSS. No usar `<canvas>`.

---

## Prompt 2 — El mecanismo amañado

Los 5 círculos "intento-indicador" que ya existen en el header hoy marcan
disparos dentro de la ronda actual — quiero que dejen de hacer eso y
pasen a representar los 5 INTENTOS (rondas) en su lugar. El contador de
texto de abajo ("Disparos en este intento: X/5") ya cubre el conteo de
disparos dentro de la ronda, así que no hace falta un indicador nuevo
para eso.

Agregale al juego dos estados: `attempt` (número, inicial 1, tope 5 — el
intento/ronda actual) y `luckyAttempt` (número entero entre 1 y 5,
sorteado una sola vez al cargar la página y de nuevo en cada "Reiniciar
todo" — el intento garantizado a tener éxito, nunca mostrado en pantalla
ni revelado al jugador de ninguna forma).

Cambiá la lógica de acierto en `verificarImpacto`: ya no debe depender de
si la pelotita coincide con la posición real de un pato.
- Si `attempt` es igual a `luckyAttempt`: cada disparo es acierto
  automático — se incrementa `ducksHit` siempre, y el pato vivo más
  cercano al punto del disparo desaparece (aunque el resultado ya estaba
  decidido antes de calcular esa cercanía).
- Si `attempt` es distinto de `luckyAttempt`: cada disparo acierta con
  una probabilidad fija del 25%, sorteada de nuevo en cada click, sin
  relación con la posición real de la pelotita ni de los patos.

Agregá además una distancia máxima de tolerancia (`DISTANCIA_MAX_TOLERANCIA
= 90`, en píxeles) entre el punto del click y el pato vivo más cercano:
solo si esa distancia es menor o igual a la tolerancia, el pato
desaparece — aunque el algoritmo ya haya decidido que ese disparo
"acierta". Si el click está lejos de todos los patos, no debe voltear
ninguno, aunque el intento sea el `luckyAttempt`. Esto hace que el
jugador sienta que su puntería importa, sin que en el fondo determine
nada.

Bajá también el rango de velocidad de los patos: en vez de 1 a 4 px por
frame, usá un rango de 0.3 a 1.2 px por frame — más lento en general,
pero conservando que cada pato tenga una velocidad distinta entre sí.

Cambiá `verificarFinJuego`:
- Cuando `ducksHit` llega a 3: victoria total (`finalizarJuego(true)`),
  marcando además como "usado" el intento-indicador correspondiente al
  `attempt` actual.
- Cuando `shots` llega a 5 y `ducksHit` es menor a 3:
  - Si `attempt` es menor a 5: marcá como "usado" el intento-indicador de
    ese `attempt`, incrementá `attempt`, y reseteá `shots` a 0,
    `ducksHit` a 0, y los patos a posiciones y velocidades nuevas (todos
    `vivo: true`) — NO llames a `finalizarJuego`, el juego sigue.
    Mostrá brevemente (1 segundo) el mensaje "Intento fallido — te
    quedan X intentos" en el overlay, y después ocultalo automáticamente
    para que se pueda seguir jugando.
  - Si `attempt` ya es 5 (era el último): ahí sí `finalizarJuego(false)`
    como hoy, marcando también ese último indicador como usado.

`reiniciarJuego` debe además: resetear `attempt` a 1, sortear un
`luckyAttempt` nuevo entre 1 y 5, y quitar la clase "usado" de los 5
intento-indicadores.

---

## Prompt 3 — Portada y libro (candado)

Envolvé el juego en una pantalla que sea sobre otra cosa: acceso a un libro.

La página arranca mostrando un `<section id="portada">` con el título "El
Príncipe — Nicolás Maquiavelo" y un `<button id="candado">🔒 Bloqueado —
cazá 3 patos para leer</button>`. El juego de tiro al blanco no se ve
todavía.

Agregá un estado `paso` con tres valores: "portada", "juego", "libro".
- Arranca en "portada": se ve el candado, nada más.
- Al hacer click en el botón de candado: primero cambiá `paso` a "juego"
  y recién después reiniciá el juego (posiciones de patos, `attempt`,
  `luckyAttempt`, contadores) — en ese orden, porque reiniciar el juego
  es lo que arranca la animación de los patos, y esa animación solo debe
  correr cuando `paso` ya vale "juego". Se oculta la portada y aparece la
  galería de tiro completa (indicadores de intentos, mira, patos, contador).
- Cuando el juego termina en victoria (`ducksHit` llega a 3): esperá 1
  segundo mostrando "¡Ganaste!", después `paso` pasa a "libro" — se oculta
  el juego y aparece un `<article>` con el texto de "El Príncipe" (dejá un
  placeholder de texto largo de ejemplo, lo reemplazo yo después) en
  formato legible con `<h1>`/`<p>`, más un botón "Volver a intentar" que
  vuelve todo a `paso: "portada"` y reinicia el juego como si fuera la
  primera vez.
- Si el juego termina en derrota (se acabaron los 5 intentos): quedate en
  `paso: "juego"` con el mensaje de derrota y el botón "Reiniciar todo" ya
  existente, sin pasar a "libro".

El juego no cambia por dentro: misma galería, mismos estados, misma
lógica de intentos y del `luckyAttempt`. Solo deja de ser lo primero que
se ve y pasa a ser un paso intermedio entre la portada y el libro.

---

## Prompt 4 — El libro real (PDF embebido, sin depender de red)

Reemplazá por completo el `<article id="libro-articulo">` con el texto
placeholder del paso "libro" — ya no hace falta, ni el texto de ejemplo.

Agregá en el `<head>`, antes del `<style>`, esta librería (no reemplaza
nada existente, solo se suma):

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.16.105/pdf.min.js"></script>
```

Nuevo HTML para el paso "libro" (el contenedor exterior mantiene el id
`contenedor-libro`):

```html
<div id="contenedor-libro" class="oculto">
  <div class="libro-header-bar">
    <div class="paginacion-controls">
      <button id="btn-prev" class="btn-action">‹ Anterior</button>
      <span id="page-num-info">Página 1 / 98</span>
      <button id="btn-next" class="btn-action">Siguiente ›</button>
    </div>
    <button class="btn-action" id="btn-volver">Volver a intentar</button>
  </div>
  <div class="pagina-container">
    <canvas id="pdf-canvas"></canvas>
  </div>
</div>
```

CSS para el paso "libro" (reemplaza las reglas del placeholder de texto;
las reglas de ancho ya existentes de `#contenedor-libro` quedan igual):

```css
.libro-header-bar {
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
  flex-wrap: wrap;
  gap: 15px;
}

.paginacion-controls {
  display: flex;
  align-items: center;
  gap: 15px;
  background: #fff;
  padding: 8px 16px;
  border: 4px solid var(--text-color);
  border-radius: 8px;
  box-shadow: 4px 4px 0px var(--text-color);
  font-size: 1.1rem;
}

.pagina-container {
  width: 100%;
  max-height: 85vh;
  overflow: auto;
  border: 4px solid var(--text-color);
  border-radius: 8px;
  box-shadow: 6px 6px 0px var(--text-color);
  background-color: #525659;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 20px;
}

#pdf-canvas {
  display: block;
  width: auto;
  height: auto;
  max-width: 100%;
  background: #fff;
  box-shadow: 0 4px 10px rgba(0,0,0,0.3);
}
```

IMPORTANTE sobre `align-items: flex-start` en `.pagina-container`: sin
esa propiedad, el `display: flex` por defecto estira verticalmente al
`<canvas>` hijo para llenar el contenedor, lo que deforma la página del
PDF. No la omitas.

Lógica JS del libro (se agrega dentro del mismo `<script>` que ya tiene
el juego, después de las funciones del juego):

```js
pdfjsLib.GlobalWorkerOptions.workerSrc = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.16.105/pdf.worker.min.js';

const PDF_SCALE = 2.0;
let pdfDoc = null;
let paginaLibroActual = 1;
let pageRendering = false;
let pageNumPending = null;
const pdfCanvas = document.getElementById('pdf-canvas');
const ctxPdf = pdfCanvas.getContext('2d');
const pageNumInfo = document.getElementById('page-num-info');
const btnPrev = document.getElementById('btn-prev');
const btnNext = document.getElementById('btn-next');

function base64ToUint8Array(base64) {
  const raw = atob(base64);
  const bytes = new Uint8Array(raw.length);
  for (let i = 0; i < raw.length; i++) bytes[i] = raw.charCodeAt(i);
  return bytes;
}

function renderPaginaLibro(n) {
  pageRendering = true;
  pdfDoc.getPage(n).then(page => {
    const viewport = page.getViewport({ scale: PDF_SCALE });
    pdfCanvas.width = viewport.width;
    pdfCanvas.height = viewport.height;
    page.render({ canvasContext: ctxPdf, viewport }).promise.then(() => {
      pageRendering = false;
      if (pageNumPending !== null) {
        const siguiente = pageNumPending;
        pageNumPending = null;
        renderPaginaLibro(siguiente);
      }
    });
  });

  paginaLibroActual = n;
  pageNumInfo.textContent = `Página ${n} / ${pdfDoc.numPages}`;
  btnPrev.disabled = n <= 1;
  btnNext.disabled = n >= pdfDoc.numPages;
  document.querySelector('.pagina-container').scrollTop = 0;
}

function mostrarPaginaLibro(n) {
  if (pageRendering) {
    pageNumPending = n;
  } else {
    renderPaginaLibro(n);
  }
}

function cargarLibroSiHaceFalta() {
  if (pdfDoc) {
    mostrarPaginaLibro(1);
    return;
  }
  pageNumInfo.textContent = "Cargando libro...";
  pdfjsLib.getDocument({ data: base64ToUint8Array(EL_PRINCIPE_BASE64) }).promise.then(doc => {
    pdfDoc = doc;
    mostrarPaginaLibro(1);
  }).catch(err => {
    console.error("Error al cargar el libro:", err);
    pageNumInfo.textContent = "Error al cargar el libro.";
  });
}

btnPrev.addEventListener('click', () => {
  if (paginaLibroActual > 1) mostrarPaginaLibro(paginaLibroActual - 1);
});

btnNext.addEventListener('click', () => {
  if (pdfDoc && paginaLibroActual < pdfDoc.numPages) mostrarPaginaLibro(paginaLibroActual + 1);
});
```

En `setPaso`, la rama `else if (paso === "libro")` debe llamar a
`cargarLibroSiHaceFalta()` (no a ninguna función de renderizado de texto):

```js
} else if (paso === "libro") {
  elContenedorLibro.classList.remove('oculto');
  detenerAnimacion();
  cargarLibroSiHaceFalta();
}
```

Por qué se lee el PDF desde una constante en base64 y no por URL: usar
`pdfjsLib.getDocument(PDF_URL)` con una ruta de archivo hace que pdf.js
pida el PDF por red (fetch) — eso falla con "Error al cargar el libro"
cuando el HTML se abre con doble click (protocolo file://), porque los
navegadores bloquean por seguridad que una página local pida otro
archivo local así (política de CORS). Pasándole los bytes ya
decodificados con `{ data: ... }` no hay ningún pedido de red de por
medio, así que no hay bloqueo, y además todo el libro queda embebido en
un solo archivo HTML sin depender de ningún PDF suelto al lado.

Declarar la constante al final del todo, inmediatamente antes de cerrar
`</script>`, con este comentario arriba (así alguien que lea el código no
tiene que scrollear un bloque de datos larguísimo para ver el resto del
programa):

```js
// ---------------------------------------------------------------
// DATOS DEL LIBRO (PDF codificado en base64, generado a partir del
// archivo real de "El Príncipe" — no es parte de la lógica del juego,
// es solo el contenido embebido para que todo quede en un único archivo)
// ---------------------------------------------------------------
const EL_PRINCIPE_BASE64 = "PEGAR_AQUI_EL_PDF_EN_BASE64";
```

Como todavía no tengo el PDF codificado en base64, dejá esa constante con
ese valor placeholder — yo lo reemplazo por el contenido real después.
Este reemplazo es un paso mecánico de pegar texto, no algo que un modelo
de IA pueda generar de una sola vez: el PDF real en base64 son 821.968
caracteres, muy por encima del límite de salida de cualquier modelo de
lenguaje en una sola respuesta.

No cambies nada de la galería de tiro, los patos, el `luckyAttempt`, los
indicadores de intento, la portada, ni los estilos de los pasos 1 y 2.
Este prompt solo toca el paso "libro".
