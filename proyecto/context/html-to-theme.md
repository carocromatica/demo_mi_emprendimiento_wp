# html-to-theme: instrucciones para el agente

Tu tarea es convertir el sitio HTML estático de **Brote** en un **tema clásico de WordPress en PHP** llamado `brotheme-shop`. Lee este archivo completo antes de escribir código y síguelo en orden.

## Objetivo

Producir la carpeta `brotheme-shop/` que se pueda comprimir en un `.zip`, subir a WordPress (Apariencia, Temas, Añadir nuevo) y activar sin errores, con el mismo diseño que `proyecto/html/`.

## Entradas

- `proyecto/html/`: sitio fuente. Contiene `*.html`, `css/styles.css`, `js/*.js` e `img/`. Es la fuente de verdad del diseño.
- `tema-base/`: tema en blanco. Es el punto de partida (`header.php`, `footer.php`, `front-page.php`, `index.php`, `page.php`, `single.php`, `archive.php`, `functions.php`, `style.css`).
- `proyecto/context/plugins.md`: qué debe cumplir el tema para ser compatible con los plugins. Léelo antes del paso 5.
- `DESIGN.md` y `PRODUCT.md` del proyecto, si existen: colores, tipografías y tono.

## Salida

Una carpeta `brotheme-shop/` en la raíz del repo, con esta estructura:

```text
brotheme-shop/
├── style.css          (cabecera del tema)
├── functions.php
├── header.php
├── footer.php
├── front-page.php
├── home.php
├── single.php
├── archive.php
├── page.php
├── page-contacto.php
├── page-tienda.php
├── page-producto.php
├── page-carrito.php
├── 404.php
└── assets/
    ├── css/styles.css
    ├── js/
    └── img/
```

## Reglas (no las rompas)

1. **No rediseñes.** Conserva el HTML, las clases y el CSS tal como están. Cambia solo lo que WordPress debe generar.
2. **No uses Elementor, constructores visuales ni temas de pago.** Solo PHP, HTML, CSS y JS propios.
3. **No instales librerías ni frameworks nuevos.** Sin Composer ni npm.
4. **No toques `proyecto/html/`.** Es la fuente: léela, no la modifiques.
5. **No inventes contenido.** El texto de las páginas va en WordPress (`the_content()`); las plantillas solo traen la estructura.
6. **Nada de rutas fijas.** Prohibido `img/...`, `/img/...`, `http://...` del sitio o enlaces que terminen en `.html`. Usa siempre `get_template_directory_uri()`, `home_url()` o `get_permalink()`.
7. **Escapa todo lo que imprimas:** `esc_html()`, `esc_attr()`, `esc_url()`.
8. **Un archivo PHP que no es plantilla** empieza con `if ( ! defined( 'ABSPATH' ) ) { exit; }`.
9. **Cambios pequeños en `functions.php`.** Un error ahí rompe todo el sitio. Después de cada edición, revisa la sintaxis con `php -l <archivo>` si PHP está instalado.
10. **Prefijo `brotheme_`** en funciones, handles y hooks. Text domain: `brotheme-shop`.
11. **Un commit por hito**, con mensaje en español, como se indica abajo.

## Procedimiento

Trabaja por hitos. Al terminar cada uno, verifica sus criterios y haz commit. Si un criterio falla, corrígelo antes de pasar al siguiente.

### Hito 1: esqueleto del tema

1. Copia `tema-base/` a `brotheme-shop/`.
2. Reemplaza la cabecera de `style.css` por:

```css
/*
Theme Name: Brotheme Shop
Description: Tema propio de Brote, tienda de jardinería.
Author: (nombre de la estudiante)
Version: 0.1
Text Domain: brotheme-shop
*/
```

3. En `functions.php`, renombra el prefijo `mi_tema_` a `brotheme_` y mantén `title-tag`, `post-thumbnails`, `html5` y el menú `principal`.

**Criterio:** la carpeta existe, la cabecera es correcta y no quedan referencias a `mi_tema` ni `mi-tema`.
**Commit:** `Tema: esqueleto brotheme-shop`.

### Hito 2: assets

1. Copia `proyecto/html/css/styles.css` a `brotheme-shop/assets/css/styles.css`.
2. Copia los archivos de `proyecto/html/js/` a `brotheme-shop/assets/js/`.
3. Copia a `brotheme-shop/assets/img/` solo las imágenes del diseño que el CSS o las plantillas usan (logos, íconos, fondos). No copies fotos de contenido: esas van a la biblioteca de medios.
4. En el CSS, reescribe cualquier `url(...)` para que apunte a `../img/...`.
5. En `functions.php`, carga todo con `wp_enqueue_style` y `wp_enqueue_script`, con versión (`wp_get_theme()->get( 'Version' )`). Carga cada JS solo en la página que lo usa (`is_page( 'tienda' )`, etc.).

**Criterio:** ninguna ruta del CSS apunta fuera de `assets/`; los scripts se cargan con `wp_enqueue_script`, no con `<script>` pegado.
**Commit:** `Tema: estilos, scripts e imágenes`.

### Hito 3: cabecera, pie y menú

1. De `index.html`, mueve a `header.php` todo desde `<!DOCTYPE html>` hasta el final del menú, y a `footer.php` todo desde el pie hasta `</html>`.
2. En `header.php`: elimina el `<title>` y cualquier `<meta name="description">` fijo; agrega `<?php wp_head(); ?>` antes de `</head>` y usa `<body <?php body_class(); ?>>`.
3. En `footer.php`: agrega `<?php wp_footer(); ?>` justo antes de `</body>`.
4. Reemplaza el menú escrito a mano por `wp_nav_menu()` con `theme_location => 'principal'`, pasando las clases del HTML original con `menu_class` y `container`.
5. Reemplaza los logos y rutas por `get_template_directory_uri()`.

**Criterio:** `wp_head()`, `wp_footer()` y `body_class()` están presentes; no hay `<title>` fijo; el menú sale de WordPress.
**Commit:** `Tema: cabecera, pie y menú`.

### Hito 4: portada y blog

Conserva en cada plantilla el HTML y las clases del archivo fuente.

| Fuente | Plantilla | Qué reemplazar por PHP |
|---|---|---|
| `index.html` | `front-page.php` | Secciones fijas tal cual; las tarjetas del blog con un `WP_Query` de las 3 últimas entradas |
| `blog.html` | `home.php` | El Loop con `the_title()`, `the_permalink()`, `get_the_excerpt()`, `the_post_thumbnail()` |
| `articulo.html` | `single.php` | `the_title()`, `the_content()`, `the_post_thumbnail()`, categoría, fecha y `comments_template()` |
| Categoría del blog | `archive.php` | El Loop de la categoría |

Cada plantilla empieza con `get_header()` y termina con `get_footer()`. Si no hay entradas, muestra un mensaje breve en lugar de dejar el espacio vacío.

**Criterio:** las tres plantillas usan el Loop; los enlaces de entradas usan `the_permalink()`.
**Commit:** `Tema: portada, blog y artículo`.

### Hito 5: páginas

| Fuente | Plantilla | Notas |
|---|---|---|
| `nosotros.html`, `preguntas-frecuentes.html`, `terminos.html`, `privacidad.html` | `page.php` | `the_title()` y `the_content()`; si el diseño de una página es distinto, crea `page-{slug}.php` |
| `contacto.html` | `page-contacto.php` | Deja el lugar del formulario con `<?php echo do_shortcode( '[contact-form-7 title="Contacto"]' ); ?>` |
| `tienda.html`, `producto.html`, `carrito.html` | `page-tienda.php`, `page-producto.php`, `page-carrito.php` | Prototipo: conserva el HTML y carga su JS desde `assets/js/`. No agregues WooCommerce |
| `404.html` | `404.php` | Contenido fijo del diseño |

Sigue lo que pide `proyecto/context/plugins.md` para el formulario y los comentarios.

**Criterio:** cada página del diseño tiene plantilla; no queda ningún enlace que termine en `.html`.
**Commit:** `Tema: páginas y plantillas`.

### Hito 6: enlaces

1. Busca en todo `brotheme-shop/` cualquier `.html"` y cualquier `img/` o `css/` sin prefijo de WordPress.
2. Reemplaza:

| Antes | Después |
|---|---|
| `index.html` | `<?php echo esc_url( home_url( '/' ) ); ?>` |
| `contacto.html` | `<?php echo esc_url( home_url( '/contacto/' ) ); ?>` |
| `tienda.html?cat=sustratos` | `<?php echo esc_url( home_url( '/tienda/?cat=sustratos' ) ); ?>` |

3. Los JS que leen `location` o rutas relativas deben seguir funcionando con las URL nuevas; revísalos y ajusta solo lo necesario.

**Criterio:** `grep -rn '\.html"' brotheme-shop/` no devuelve resultados.
**Commit:** `Tema: enlaces de WordPress`.

### Hito 7: verificación y entrega

1. Revisa la sintaxis de todos los PHP (`php -l`), si está disponible.
2. Revisa que no haya `<?php` sin cerrar, ni llaves o etiquetas HTML desbalanceadas.
3. Genera `brotheme-shop.zip` con la carpeta `brotheme-shop/` dentro (la carpeta, no su contenido suelto).
4. Escribe en el README del repo, en pocas líneas: nombre del tema, cómo instalarlo y las páginas que hay que crear con su slug (`contacto`, `tienda`, `producto`, `carrito`, `nosotros`, `preguntas-frecuentes`, `terminos`, `privacidad`).

**Criterio:** todos los PHP pasan la revisión y el `.zip` tiene `style.css` en `brotheme-shop/style.css`.
**Commit:** `Tema: verificación y entrega`.

## Cuándo detenerte y preguntar

Detente y pregunta a la estudiante, sin adivinar, si:

- una página del HTML no encaja con ninguna plantilla de la tabla;
- un JS depende de datos que no existen en WordPress y no sabes cómo reemplazarlos;
- necesitas un plugin o librería que no está en `proyecto/context/plugins.md`;
- tienes que cambiar el diseño (colores, tipografías, estructura) para que funcione.

## Al terminar

Entrega un resumen corto con: los archivos creados, lo que quedó como prototipo y no es funcional (tienda, producto y carrito), lo que no pudiste verificar sin WordPress instalado, y cualquier duda abierta.
