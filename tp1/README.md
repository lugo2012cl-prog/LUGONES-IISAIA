# TP 1 — Caza patos, candado de un libro

Un juego de tiro al blanco tipo feria donde tenés que voltear 3 patos en 5 intentos para desbloquear la lectura de "El Príncipe" de Maquiavelo. Parece un juego de puntería — apuntás con una mira telescópica, los patos se mueven a distinta velocidad, hay una pelotita que vuela hasta donde clickeaste — pero el resultado de cada disparo no depende en absoluto de tu puntería: un algoritmo sortea de antemano cuál de los 5 intentos va a ganar, sin que el jugador lo sepa nunca.

## Cómo se ejecuta

Doble click en `index.html`. Un solo archivo, sin dependencias.

## Qué me propuse construir

Una bad UI que finge ser un juego de habilidad y no lo es. No es hostil por dificultad (como un captcha imposible de leer) sino por **engaño de agencia**: le das al jugador todas las señales de que su puntería importa —mira que sigue al mouse, patos con velocidades distintas, animación de disparo hacia el punto exacto donde apuntó— y ninguna de esas señales tiene relación real con si acierta o no. La frustración no es "esto es difícil", es "creí que dependía de mí y no dependía de nada que yo hiciera". Salió en cuatro prompts, en una sola conversación de Gemini.

## Decisiones que tomé yo

**El resultado nunca depende de la colisión real.** Es la decisión central. Le pedí explícitamente al modelo que NO implementara detección de colisión de verdad para decidir acierto/error — solo para decidir *qué pato* desaparece visualmente cuando ya se sabe que hubo acierto. La puntería es enteramente cosmética.

**Un intento garantizado, sorteado y nunca mostrado.** En vez de hacer el juego imposible o completamente al azar, un `luckyAttempt` entre 1 y 5 asegura que siempre haya una salida — pero el jugador no tiene forma de saber cuál es el intento bueno, así que igual juega los otros como si importaran.

**Intentos anidados en disparos, no una sola bolsa de 5 tiros.** Son 5 intentos, cada uno con sus propios 5 disparos (no "5 disparos en total"). Esto multiplica la sensación de progreso falso: cada intento fallido se siente como una partida nueva, no como quedarte sin balas.

**Estética de feria.** El fondo de rayas rojo/blanco, la tipografía tipo cartel y los botones dorados hacen que el engaño se sienta "divertido" antes de descubrir que está amañado — el contraste entre lo lúdico y lo tramposo es parte del chiste.

**El libro como candado, el juego como paso intermedio.** Mismo principio que remarca la cátedra en su propio ejemplo ("el captcha no es la página"): un juego de caza de patos suelto no frustra a nadie, porque nadie llega ahí queriendo leer nada. Envuelto como la única puerta hacia un libro que sí querías leer, algo mas real.

**Los indicadores de intento cambian de propósito a mitad de camino** Ver más abajo — fue un hallazgo, no un plan.

**Decisiones mías (diseño y concepto del TP):**
- La idea central del engaño: un juego de caza de patos que aparenta depender de la puntería pero en realidad tiene un resultado sorteado de antemano.
- El mecanismo concreto: 5 intentos de 5 disparos cada uno, un `luckyAttempt` garantizado y nunca revelado, sin colisión real.
- La estética de feria (rayas rojo/blanco, tipografía de cartel, dorados).
- Que el libro funcione como candado y el juego como paso intermedio, no como el contenido final en sí.
- El texto exacto de los Prompts 1, 2 y 3 — los escribí yo directamente en Gemini.
- Rechazar el placeholder de texto para el libro y pedir el PDF real; rechazar la carpeta con archivos sueltos y pedir un solo archivo; rechazar que yo (o el docente) tuviera que pegar el base64 a mano sin entender por qué, hasta confirmar que era un límite técnico real y no una excusa (probé de todo).

**Decisiones de la IA (diagnóstico técnico e implementación de los ajustes):**
- Ajustes puntuales de los prompts (bajar la velocidad de los patos, agregar la tolerancia de distancia, corregir el bug del orden en el listener del candado, cambiar el visor de PDF por `pdf.js`, pasar de URL a base64) a partir de problemas que yo reportaba o que la IA encontraba revisando el código antes de aceptarlo.
- Diagnosticar la causa técnica de cada falla (el bug de los indicadores, el error de CORS en `file://`, el estiramiento del canvas por `align-items` de flexbox, el límite de salida de los modelos de lenguaje para generar el base64).
- Decidir cuándo un ajuste era lo bastante simple y mecánico como para hacerlo directo en el código (el cambio a base64, mover esa constante al final del archivo, el fix del estiramiento) en vez de redactar otro prompt.
- Cuando pedi el texto completo del libro, el primer intento de la IA fue extraerlo del PDF con pdftotext y reconstruir los capítulos con un script — una solución innecesariamente complicada para el problema, que termino descartando. Está documentada como error en la sección siguiente.
- Armar y consolidar los prompts en la secuencia final de `prompts.md`, y detectar que pedirle a Gemini que generara el base64 completo (Prompt 5, después descartado) no era viable por el límite de salida del modelo.

## Qué salió mal y cómo lo corregí

El primer prompt pedía "una fila de 5 indicadores, uno por intento disponible, que se van marcando como usado a medida que se consumen" — pero en ese momento el juego todavía no tenía el concepto de "intento" como ronda (eso lo agregaba recién el segundo prompt). El modelo resolvió la ambigüedad de la única forma que tenía sentido con lo que existía en ese momento: interpretó "intento" como "disparo", y los 5 círculos terminaron marcando los disparos dentro de la ronda actual, no las rondas en sí.

Si hubiera mandado el segundo prompt tal como lo tenía planeado —"marcá el indicador de intento correspondiente como usado" al perder una ronda— habría chocado en silencio con lo que el primer prompt ya había construido: dos conceptos distintos peleando por los mismos cinco círculos.

Lo agarré releyendo el código generado por el primer prompt antes de mandar el segundo, no después. La corrección fue agregar, al principio del segundo prompt, una instrucción explícita: los indicadores dejan de contar disparos (eso ya lo cubre el contador de texto) y pasan a contar intentos/rondas completas, marcándose recién cuando una ronda se agota sin ganar o cuando el juego termina.

Es exactamente el tipo de bug silencioso que una revisión cruzada entre prompt y código detecta antes de que se complique con la siguiente iteración — y no lo vi al escribir el prompt inicial, lo vi releyendo el resultado.

**El paso "libro" mostraba un texto de ejemplo, no el libro real, y me compliqué de más resolviéndolo.** El prompt que arma el flujo de tres pasos pedía a propósito un placeholder ("dejá un texto largo de ejemplo, lo reemplazo yo después") para no obligar a Gemini a escribir de memoria un libro entero. El problema es que mi primer instinto para resolverlo fue extraer el texto completo del PDF con `pdftotext`, reconstruir los 26 capítulos por script (detectando títulos en mayúsculas, uniendo párrafos partidos por saltos de línea del PDF) e insertarlo todo como HTML estático — una solución frágil y sobretrabajada para un problema simple. La corrección más directa era pedirle a Gemini que, en vez de mostrar texto, pusiera un `<iframe>` apuntando al archivo real (`src="el-principe.pdf"`) ubicado junto al `index.html`: el navegador lo renderiza solo, sin parsear ni reformatear nada. Descarté el script de extracción y pedí ese cambio con un prompt nuevo.

**El PDF cargado por URL fallaba al abrir el HTML con doble click, aunque el archivo estuviera al lado.** Después de pasar el libro a un lector con `pdf.js` (para poder fijar el zoom yo mismo en vez de depender del visor nativo), el paso "libro" tiraba "Error al cargar el libro" incluso con `el-principe.pdf` bien ubicado junto al `index.html`. La causa no era la ruta del archivo: `pdf.js` necesita *pedir* el PDF por código (un `fetch` interno) para leer sus bytes, y los navegadores bloquean por seguridad que una página abierta como `file://` haga ese tipo de pedido a otro archivo local — es la política de CORS para archivos locales, no un bug del código ni algo que un prompt distinto a Gemini Canvas pudiera arreglar. La solución fue sacar el `fetch` de la ecuación por completo: en vez de pedirle a `pdf.js` una URL, le paso los bytes del PDF ya decodificados desde una constante en base64 embebida en el propio script (`pdfjsLib.getDocument({ data: ... })`). Sin URL de por medio, no hay pedido de red y el bloqueo de CORS no aplica.

**Entregar una carpeta con PDF (y de paso una carpeta de imágenes, en un intento intermedio) no tenía sentido para un TP que se abre con doble click.** Mientras perseguía el problema anterior, terminé con una carpeta `tp1/` con `index.html` + `el-principe.pdf` (y en un paso intermedio, previo a la solución de base64, hasta 98 imágenes de las páginas del libro convertidas con `pdftoppm`, como alternativa para controlar el tamaño sin depender de ningún visor). Ese armado funcionaba, pero era innecesariamente pesado y frágil: cualquiera que abriera el TP tenía que mantener todos los archivos juntos en la carpeta exacta, cuando el punto de partida era justamente poder abrir un solo archivo. La solución de base64 del punto anterior resuelve las dos cosas a la vez: al no depender de ningún archivo externo, todo el TP —juego, prompts y libro— queda en un único `index.html` que se abre con doble click, sin carpetas ni archivos sueltos que puedan separarse. Descarté la carpeta de imágenes y el PDF suelto una vez confirmado que el enfoque de base64 andaba.

**El texto de las páginas se veía estirado horizontalmente al cambiar el visor por `pdf.js` con canvas.** El contenedor del libro usa `display: flex` sin fijar `align-items`, y por defecto un contenedor flex estira a sus hijos en el eje transversal para llenar el espacio disponible — eso deformaba el `<canvas>` donde se dibuja cada página, achatándola. Lo corregí fijando `align-items: flex-start` en el contenedor y declarando el canvas con `width: auto; height: auto` explícitos, para que respete siempre las dimensiones reales que le da `pdf.js` en vez de las que el layout quisiera imponerle.

**Nota sobre el proceso:** estos tres últimos cambios (base64 en vez de fetch, un solo archivo en vez de carpeta, y el fix del estiramiento) los resolví yo directamente en el código, no a través de un prompt a Gemini Canvas — fue un pedido explícito así, dado lo específico y técnico del diagnóstico. El prompt equivalente para que Gemini Canvas genere la misma estructura (lectura del PDF desde una constante en base64 en vez de por URL, con esa constante al final del script) está documentado en `prompts.md` para dejar el proceso completo, aunque en este caso el código real que se usó en la entrega salió de mi intervención directa y no de la respuesta de Gemini Canvas.

**El PDF se veía chico dentro del iframe.** Ya con el iframe apuntando al archivo real, el visor de PDF nativo del navegador (no algo que controle el código del juego) renderizaba la página pequeña dentro del contenedor. Pedí dos cambios a la vez con el mismo prompt: agrandar el ancho máximo de `#contenedor-libro` (de 1000px a 1400px / 95vw) y agregar el parámetro estándar de PDF `#zoom=125` al final del `src` del iframe. Al probarlo, lo que realmente resolvió el problema fue el ancho del contenedor — el visor solo tiene tanto espacio horizontal para renderizar la página, y con 1000px la achicaba para que entrara. El parámetro de zoom quedó en el código pero no fue el factor determinante; lo dejo igual porque no molesta, pero la causa real era más simple de lo que pensé al principio.

**Los patos se movían demasiado rápido.** El primer prompt pedía "distinta velocidad" sin fijar un rango, y Gemini Canvas eligió uno (1 a 4 px por frame) que en la práctica resultaba frenético — casi imposible seguir un pato con la mira. No era un error de interpretación, era una ambigüedad mía: "distinta velocidad" no dice nada sobre la magnitud. Lo corregí con un prompt de ajuste puntual pidiendo explícitamente un rango más lento, conservando la variación relativa entre patos.

**El disparo acertaba sin importar dónde apuntara.** Al revisar el código del segundo prompt encontré que `verificarImpacto` decidía el acierto solo con la probabilidad (intento sorteado o 25% al azar) y, si tocaba acertar, volteaba directamente al pato vivo más cercano — sin comparar esa distancia contra ningún umbral. Resultado: aunque clickeara en una esquina vacía de la galería, si el algoritmo decidía que ese disparo "acertaba", igual caía un pato. Rompía la sensación de apuntar, que es justamente lo que hace creíble el engaño (el jugador tiene que sentir que su puntería importa un poco, aunque en el fondo no importe nada). Lo corregí agregando una distancia máxima entre el click y el pato más cercano: si no hay ningún pato dentro de ese radio, el disparo no voltea nada, así el resultado sigue estando amañado pero ya no se siente arbitrario.

**El libro bloqueado abría una pantalla en blanco.** Esto no fue un problema del prompt de Gemini Canvas sino de mi propio entorno de prueba: para que pudieras jugar el código generado sin descargar nada, lo publiqué como una Claude Artifact y probé incrustar el PDF entero codificado en base64 dentro de un `<iframe>`. La Artifact corre dentro de un sandbox con política de contenido restrictiva, y ese sandbox anidado no renderizaba el visor de PDF nativo del navegador dentro del iframe — quedaba en blanco. La solución fue reemplazar el iframe por un lector propio: cargo la librería `pdf.js` desde un CDN y dibujo cada página del PDF sobre un `<canvas>`, con botones para pasar de página. Esto es una particularidad de cómo probé el juego en este chat, no algo que vayas a necesitar replicar en el repo real — ahí el PDF va como archivo aparte (`tp1/el-principe.pdf`) y un iframe normal con `src` relativo alcanza, sin sandbox de por medio.

**Los patos no se movían la primera vez que entrabas al juego.** El prompt que envuelve todo en un flujo de tres pasos (portada → juego → libro) generó el botón del candado con este handler:

```js
elCandado.addEventListener('click', () => {
  reiniciarJuego();
  setPaso("juego");
});
```

`reiniciarJuego()` solo arranca el `requestAnimationFrame` si la variable de estado `paso` ya vale `"juego"` en ese momento — pero se llama *antes* de `setPaso("juego")`, así que todavía vale `"portada"` y la animación nunca se dispara. Los patos se creaban pero quedaban congelados en su posición inicial en esa primera partida (el bug no se notaba si perdías y usabas "Reiniciar todo" dentro del juego, porque ahí `paso` ya valía `"juego"` cuando se llamaba a la función — el problema era únicamente en la entrada desde la portada). Lo agarré releyendo el código antes de aceptarlo, igual que el bug de los indicadores del Prompt 2, y lo mandé de vuelta a Gemini Canvas para que invirtiera el orden de esas dos líneas en vez de corregirlo yo por fuera del proceso de prompting.

**Por qué el base64 del PDF queda como placeholder y no se le pide a Gemini Canvas que lo genere.** El PDF real, codificado en base64, son 821.968 caracteres. Cualquier modelo de lenguaje (Gemini incluido, en cualquiera de sus versiones) tiene un límite máximo de texto que puede generar en una sola respuesta — muy por debajo de esa cifra. No es una limitación de cómo está redactado el prompt: es un límite de salida del modelo, así que no hay forma de escribir el pedido de otra manera para esquivarlo. Por eso el Prompt 4 le pide a Gemini Canvas que deje la constante `EL_PRINCIPE_BASE64` con un valor placeholder corto (`"PEGAR_AQUI_EL_PDF_EN_BASE64"`) en vez de pedirle el contenido real — ese reemplazo final es un paso mecánico de pegar texto una sola vez, no algo que la IA pueda ni deba intentar generar.

El contenido real para ese reemplazo está en `el-principe-base64.txt` (incluido junto a esta entrega): abrir ese archivo, copiar todo su contenido (es una sola línea de 821.968 caracteres, sin saltos), y pegarlo reemplazando `"PEGAR_AQUI_EL_PDF_EN_BASE64"` en el HTML que devuelva Gemini Canvas — las comillas de la constante quedan igual, solo cambia lo que está entre ellas.

Si más adelante hace falta pedirle a Gemini Canvas otro ajuste sobre el archivo ya completo (con el base64 real ya insertado), hay que aclarárselo explícitamente en el prompt: "no reescribas la constante `EL_PRINCIPE_BASE64`, dejala tal cual está". Sin esa aclaración, el modelo puede intentar reproducir el archivo completo en su respuesta —incluyendo esa constante— y la respuesta se corta a la mitad por el mismo límite de salida. Pedirle que la deje intacta evita que la tenga que volver a escribir.

## Prompts

El registro completo está en [prompts.md](prompts.md). El más importante es el segundo: es donde el juego deja de ser un juego real y pasa a ser la ilusión de uno, y donde se corrigió la ambigüedad de los indicadores antes de que se propagara.
