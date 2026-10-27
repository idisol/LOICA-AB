# Comparador de Planos (VialAG)

Página web que compara dos planos (PDF o DXF). A es la referencia y se ve completo en café (#61594b); B va encima: lo que coincide con A queda en ese café y lo que A no tiene se pinta en rojo. Al intercambiar A y B, el otro plano pasa a ser la referencia y la alineación se mantiene (también la hecha a mano). Las pestañas Plano A y Plano B muestran cada plano solo, sin hatch, en #877a67 y #c7bda8. El porcentaje de cambio aproximado cuenta lo distinto de ambos lados. Todo se procesa en el navegador de quien la usa: los planos no se suben a ningún servidor.

Es HTML, CSS y JavaScript sin compilar. No usa React ni necesita Node, base de datos ni servidor de aplicaciones.

## Archivos

```
index.html            estructura de la página, textos y dibujos de la loica (SVG)
css/estilos.css       colores, tipografía y diseño
js/comparador.js      toda la lógica: lectura de PDF/DXF, alineación, comparación y descargas
img/logo-vialag.png   logo del header
img/favicon-32.png, img/favicon-192.png, img/apple-touch-icon.png, favicon.ico
                      ícono de la pestaña del navegador (y del acceso directo en celular),
                      hechos desde img/icono-idisol.png (el original, 213 × 213 px)
audio/loica-canto.mp3 canto de loica (4 s) que suena al hacer clic sobre el pájaro
.nojekyll             archivo vacío para GitHub Pages: publica los archivos tal como están
vendor/               librerías externas incluidas (no se cargan desde internet)
  pdf.min.js, pdf.worker.min.js   pdf.js 3.11.174 (lee PDF)
  dxf-parser.js                   dxf-parser 1.1.2 (lee DXF)
  jspdf.umd.min.js                jsPDF 2.5.1 (genera el PDF de resultado)
  jszip.min.js                    JSZip 3.10.1 (solo en la página de Claude, que no acepta .dxf: ahí el DXF va dentro de un .zip; alojada en tu sitio se descarga el .dxf directo)
  licencias/                      licencias de esas librerías
```

## Probarlo en tu computador

Descomprime el ZIP y abre `index.html` con doble clic (Chrome, Edge, Firefox o Safari). Funciona sin servidor.

## Publicarlo en GitHub (GitHub Pages)

La carpeta funciona tal cual en GitHub Pages: todas las rutas son relativas y no hay nada que compilar.

1. Crea un repositorio (por ejemplo `comparador-planos`) y sube **el contenido de esta carpeta** a la raíz: `index.html`, `css/`, `js/`, `img/`, `audio/`, `vendor/`, `favicon.ico`, `.nojekyll` y este README. Desde la web de GitHub: *Add file → Upload files* y arrastra todo (incluidas las carpetas). Con git: `git add -A`, `git commit` y `git push`.
2. En el repositorio: *Settings → Pages → Build and deployment → Source: Deploy from a branch*, rama `main`, carpeta `/ (root)`, *Save*.
3. En uno o dos minutos queda en `https://<tu-usuario>.github.io/comparador-planos/`. Si en vez del contenido subes la carpeta entera, la dirección termina en `/comparador-planos/comparador-planos/`; funciona igual.
4. Para usar un dominio propio (por ejemplo `comparador.vialag.com`): en *Settings → Pages → Custom domain* escribe el dominio y en los DNS de vialag.com crea un registro CNAME de `comparador` hacia `<tu-usuario>.github.io`.

Para publicar una versión nueva, reemplaza los archivos por los del ZIP nuevo (mismos nombres) y vuelve a subirlos.

## Publicarlo en otro sitio propio

Sube la carpeta completa, tal como está, a cualquier hosting de archivos estáticos. Algunas opciones:

- **Netlify Drop** (app.netlify.com/drop): arrastra la carpeta y te entrega un link. Luego puedes asociar un dominio, por ejemplo `comparador.vialag.com`.
- **Hosting actual de vialag.com** (cPanel, FTP o similar): sube la carpeta a una ruta, por ejemplo `vialag.com/comparador/`.
- **Cloudflare Pages o Vercel**: sube la carpeta a un repositorio y publícalo como sitio estático.

Al estar en tu propio sitio, la página se ve sin el header de claude.ai. Recomendación: publícalo con HTTPS.

## Internet

Las librerías van incluidas en `vendor/`. Lo único que se pide a internet son las tipografías IBM Plex (Google Fonts). Sin conexión la página funciona igual y usa Arial.

## Seguridad

pdf.js 3.11.174 tiene una vulnerabilidad conocida (CVE-2024-4367) que permite ejecutar código al abrir un PDF preparado para eso. La página la evita con la mitigación oficial (`isEvalSupported: false` al abrir cada PDF, en `js/comparador.js`). Lo recomendable a mediano plazo es actualizar a pdf.js 4.2.67 o posterior, lo que exige adaptar la carga de la librería.

## Dónde cambiar cosas

- **Colores de marca:** al inicio de `css/estilos.css`, en `:root` (`--brand` panel izquierdo, `--brand-dark` header, `--diff` rojo, `--match` café de lo que coincide en la leyenda). Los colores del dibujo están en `js/comparador.js`: `COL_MATCH` (lo que coincide, #61594b) y `COL_PLAIN` (pestañas Plano A #877a67 y Plano B #c7bda8). La barra sobre el plano está en la regla `.toolbar` (#c7bda8).
- **Textos:** en `index.html` (títulos, ayudas, pie "Desarrollado por VialAG…"). El header y el pie de las descargas PDF y DXF están en `js/comparador.js` (busca `pdfHeader`, `dxfHeader`, `CREDIT` y `PRIVACY`); el logo de esas descargas es el dibujo vectorial de la viñeta de VialAG (constante `LOGO`).
- **Criterios de comparación:** al inicio de `js/comparador.js`:
  - `TOL = 1` px que una línea se puede mover sin contar como cambio.
  - `RES = 4500` tamaño de la imagen de comparación (lado mayor, en px). Más alto es más detallado y más lento.
  - `NO_MATCH = 20` bajo este % de coincidencia aparece la pantalla de error "No match".
  - `HATCH_MIN_MM = 4` en PDF, rellenos de este tamaño o más cuentan como hatch.
  - `hatch: true` (dentro de `const S = {`): estado inicial del interruptor "Hatchs" (encendido: se compara todo). Apagado, los hatch y rellenos que tienen líneas dibujadas en su borde no se comparan (sí se compara el contorno); los rellenos sin contorno, como un pilar dibujado solo en negro, sí se comparan.
    - Hatch dibujado con líneas sueltas (v24): muchos programas imprimen el hatch como una línea por trazo, sin relleno, y en DXF a veces viene "explotado" en líneas. `lineHatch` los reconoce como familia: al menos `HL_MIN` (10) líneas del mismo estilo y ángulo dibujadas seguidas, con un espaciado regular entre `HL_MIN_MM` (0,4) y `HL_MAX_MM` (6) mm del papel, y cuyos extremos tocan el borde en diagonal (eso deja fuera peldaños, barandas y celosías, que terminan en escuadra). Una familia de menos de 20 líneas debe tener un espaciado de hasta `HL_TIGHT_MM` (2,2 mm), para no confundirla con los peldaños de una gradería. En PDF se mide sobre el papel; en DXF se supone `DXF_PX_PER_MM` (4,5 px por mm) en la vista general.
    - En DXF, los hatch de patrón (tierra, hormigón) siempre se van con el interruptor, aunque no tengan contorno; solo los rellenos sólidos sin contorno se quedan.
    - Pilares (v24): un relleno sólido chico y compacto (cuadrado, círculo o rectángulo de hasta ~1:2,5, de 1,2 a 15 mm en el papel; `PILLAR_ROUND` y `PILLAR_MM`) se compara siempre, tenga o no contorno. Así un pilar queda igual en un PDF (donde suele venir sin contorno) y en un DXF (donde suele venir con él).
  - `annot: true` (también en `const S = {`): estado inicial del interruptor "Elementos de anotación" (encendido: se compara todo). Apagado, no se dibujan ni se comparan ejes y sus burbujas, cotas, textos, niveles (NPT, "NIVEL…", marcas de altura) ni líneas y cabezas de corte y elevación (tampoco van en el DXF descargado); la alineación automática los sigue usando.
    - En un DXF se reconocen por la capa (`annotLayer`: EJE, GRID, COTA, DIM, TEXTO, NIVEL, CORTE + LINEA/MARCA…, también dentro de una xref), por el bloque (`annotBlock`: nivel, eje, etiqueta…), por el tipo de entidad (TEXT, MTEXT, DIMENSION) o por un tipo de línea de grilla; además, las marcas de nivel dibujadas en otras capas se reconocen por su forma.
    - En un PDF se usan sus capas si las trae con esos nombres; si no, la página las reconoce por la forma alrededor de los textos (`annotFind`): burbujas y etiquetas con texto, marcas de nivel junto a un texto como "NPT" o "NIVEL", líneas que parten de una burbuja (ejes, cortes) hasta su otro extremo, y cotas (una línea bajo un número, con tics, puntos, flechas o líneas de extensión en sus extremos, y cuyo largo concuerda con el número a la escala de la mayoría de las cotas de la lámina).
    - También se reconocen (v24): marcas de corte (barra gruesa o rectángulo angosto con una letra al lado y su línea delgada, incluido el quiebre de un corte escalonado), etiquetas en rombo o romboide con texto, flechas de pendiente y líneas guía (leaders) que parten de un texto y terminan en una flecha, punto o tic, y las marcas de nivel en triángulo, rombo o círculo con su línea de nivel.
    - Límites en PDF: textos convertidos en líneas por el programa CAD (fuentes SHX) no se reconocen como texto; textos de menos de `MIN_TEXT` px (3) a la escala de la vista general, como los de un plano de ubicación chico, se dejan; las cotas angulares o fuera de escala se comparan.
  - `PDF_RES = 12000`: lado mayor, en píxeles, de la comparación dentro del PDF descargado; se redibuja desde los planos en cuadros de `PDF_TILE` px. Más alto = más nítido, pero más lento y pesado.
  - `LONE_EDGE = 0.75`: un relleno con menos de esta fracción de su borde sobre otras líneas cuenta como "sin contorno". Si el mismo relleno queda de un lado del umbral en A y del otro en B (pasa entre entregas de un mismo muro), se decide igual en ambos con el promedio, para que no aparezca un muro rojo que no cambió.
- **Loica:** los dos dibujos están dentro de `index.html` (`loica-ab` es la bienvenida, la loica que mira A y B, y `loica-error` el error). El cambio de mensajes del cartel lo maneja `js/comparador.js` (busca "loica"). Al hacer clic sobre el cuerpo o la cabeza del pájaro (grupo `loica-bird`, sin contar las letras A y B) suena `audio/loica-canto.mp3` una vez, a volumen moderado, y el pico se mueve mientras canta (clase `is-singing`); los clics durante el canto se ignoran. Para cambiar el sonido, reemplaza ese archivo con otro del mismo nombre; el volumen está en `song.volume` (busca `SONG`). Si la página no encuentra el archivo (por ejemplo, si se abre o se sube `index.html` solo, sin la carpeta `audio/`), usa una copia del mismo canto que va dentro de `js/comparador.js` (`SONG_DATA`, mono a 128 kbps); si cambias el sonido, cambia también esa copia o bórrala.
- **Loica roja del aviso de carga:** es un PNG dentro de `css/estilos.css` (regla `.busy div::before`). Mientras se ve el aviso (carga, comparación o descarga), la loica de bienvenida y la de error se ocultan (`.is-busy > .empty`, `.is-busy > .errorview`; la clase la pone `setBusy`), para que quede solo el botón rojo con la loica encima.
- **"VS" entre los nombres de los planos:** regla `.status .vs` en `css/estilos.css` (negrita, 1,6 veces el texto).
- **Alinear con dos puntos:** cada clic se ajusta a la esquina o cruce de líneas más cercano dentro de `SNAP_PX` (14 px de pantalla), para que A y B reciban exactamente el mismo punto. Con dos puntos por plano la escala siempre sale de ellos; "misma escala" solo redondea a 1 una escala casi igual, y viene desmarcada cuando A y B son de formatos distintos (un PDF y un DXF se dibujan a tamaños sin relación). Antes de v24 esa casilla imponía escala 1 y el PDF quedaba encima del DXF a otro tamaño.
- **Alineación automática entre PDF y DXF:** además de los ejes, prueba calzar las líneas más externas de la lámina (el marco) de uno con las del otro (`autoAlign`), y entre unas 40 propuestas elige las de mejor coincidencia de tinta (`quickScorer`).
- **Versión:** el texto chico junto al título del panel (`V01`, en #c7bda8) está en `index.html`, dentro de `<span class="ver">`; su estilo es la regla `.ptitle .ver` de `css/estilos.css`. Súbelo con cada versión (V01, V02, V03…). La numeración visible partió en V01 el 27-10-2026.
- **Ícono de la pestaña:** las líneas `<link rel="icon" …>` y `apple-touch-icon` de `index.html`. Para cambiarlo, reemplaza los archivos de `img/` y `favicon.ico` por otros con los mismos nombres.
- **Intercambiar A y B:** el botón invierte la alineación vigente en vez de alinear de nuevo (`$("swap").onclick` en `js/comparador.js`), así se conserva una alineación hecha con dos puntos.
- **Texto escondido:** `.secreto` en `index.html` (blanco sobre blanco, abajo a la derecha del visor, oculto en celular) y `SECRETO` en `js/comparador.js`, que también va en el pie del PDF. No quitarlo.
- **Líneas sobre hatch:** dentro de zonas rellenas, una línea solo cuenta como coincidente donde la otra versión tiene tinta exacta (sin la tolerancia `TOL`), para que no se vea engrosada.

## Licencias

El código propio es de VialAG. Las librerías de `vendor/` mantienen sus licencias (Apache 2.0 y MIT), cuyos textos están en `vendor/licencias/`. Ver `LICENCIAS-TERCEROS.txt`.
