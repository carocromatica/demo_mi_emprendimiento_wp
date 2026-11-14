# Convertir el HTML de Brote en un theme de WordPress

Instrucciones para pasar el sitio de `proyecto/html/` a un tema clásico en PHP llamado **brotheme-shop**. Parte de la plantilla `tema-base/` y se escribe en Antigravity; no se edita desde el panel de WordPress. Sin Elementor ni temas de pago.

## Reglas del trabajo

- El diseño ya está hecho: no lo rediseñes. Cambia solo lo necesario para que WordPress genere el HTML.
- Convierte de a una página y prueba en WordPress antes de seguir (hitos).
- Un error en `functions.php` rompe todo el sitio: cambia poco y prueba después de cada cambio.
- Nombre del tema: `brotheme-shop`. Carpeta: `brotheme-shop/`. Prefijo de funciones: `brotheme_`. Text domain: `brotheme-shop`.

## Paso 1: crear la carpeta del tema

1. Copia `tema-base/` y renómbrala `brotheme-shop/`.
2. Edita la cabecera de `style.css`:

```css
/*
Theme Name: Brotheme Shop
Description: Tema propio de Brote, tienda de jardinería.
Author: (tu nombre)
Version: 0.1
Text Domain: brotheme-shop
*/
```

3. Cambia en `functions.php` los prefijos `mi_tema_` por `brotheme_`.

## Paso 2: pasar los archivos del HTML

| HTML (`proyecto/html/`) | Tema (`brotheme-shop/`) |
|---|---|
| `css/styles.css` | `assets/css/styles.css` (o se pega bajo la cabecera de `style.css`) |
| `js/*.js` | `assets/js/` |
| `img/` | `assets/img/` (en el tema solo las que usa el diseño; el contenido va a la biblioteca de medios) |

Después reemplaza cada ruta fija por la de WordPress:

```php
<img src="<?php echo esc_url( get_template_directory_uri() ); ?>/assets/img/marca/logo-oscuro.png" alt="Brote">
```

Nunca dejes rutas como `img/...`, `/img/...` ni `http://localhost/...`: se rompen en otro servidor.

## Paso 3: cargar estilos y scripts en `functions.php`

```php
function brotheme_assets() {
	$ver = wp_get_theme()->get( 'Version' );
	wp_enqueue_style( 'brotheme-estilos', get_template_directory_uri() . '/assets/css/styles.css', array(), $ver );
	wp_enqueue_script( 'brotheme-js', get_template_directory_uri() . '/assets/js/buscador.js', array(), $ver, true );
}
add_action( 'wp_enqueue_scripts', 'brotheme_assets' );
```

Si el CSS está en `style.css`, ya lo carga `get_stylesheet_uri()`. Agrega un `wp_enqueue_script` por cada JS que use la página, y solo en las páginas que lo necesitan (por ejemplo `is_page( 'tienda' )`).

## Paso 4: separar cabecera y pie

1. De `index.html` toma todo lo que va desde `<!DOCTYPE html>` hasta el cierre del menú y llévalo a `header.php`.
2. Reemplaza el `<title>` por nada: ya lo da `add_theme_support( 'title-tag' )`.
3. Antes de `</head>` pon `<?php wp_head(); ?>`, y en `<body>` usa `<body <?php body_class(); ?>>`.
4. Lo que va desde el pie hasta `</html>` va a `footer.php`, con `<?php wp_footer(); ?>` justo antes de `</body>`.
5. Cada plantilla empieza con `<?php get_header(); ?>` y termina con `<?php get_footer(); ?>`.

## Paso 5: menú y enlaces

- El menú se registra en `functions.php` (`register_nav_menus`) y en `header.php` se imprime con `wp_nav_menu( array( 'theme_location' => 'principal' ) )`. Pásale las clases del HTML con `menu_class` y `container`.
- Los enlaces a otras páginas dejan de terminar en `.html`:

| En el HTML | En WordPress |
|---|---|
| `index.html` | `<?php echo esc_url( home_url( '/' ) ); ?>` |
| `contacto.html` | `<?php echo esc_url( get_permalink( get_page_by_path( 'contacto' ) ) ); ?>` o el menú |
| `tienda.html?cat=sustratos` | `<?php echo esc_url( home_url( '/tienda/?cat=sustratos' ) ); ?>` |

## Paso 6: una plantilla por página

| Página del HTML | Archivo del tema | Cómo se llena |
|---|---|---|
| `index.html` | `front-page.php` | Secciones fijas del diseño; las entradas del blog con un `WP_Query` |
| `blog.html` | `home.php` (o `index.php`) | El Loop con las entradas |
| `articulo.html` | `single.php` | `the_title()`, `the_content()`, `the_post_thumbnail()`, fecha y categoría |
| Categoría del blog | `archive.php` | El Loop de la categoría |
| `nosotros.html`, `preguntas-frecuentes.html`, `terminos.html`, `privacidad.html` | `page.php` | `the_title()` y `the_content()`; el texto se escribe en la página de WordPress |
| `contacto.html` | `page-contacto.php` | Cabecera y tarjetas del diseño; el formulario con el shortcode de Contact Form 7 (ver `plugins.md`) |
| `tienda.html`, `producto.html`, `carrito.html` | `page-tienda.php`, `page-producto.php`, `page-carrito.php` | Prototipo con sus `js/` tal como está. La tienda real (WooCommerce) va al plan de continuidad |
| `404.html` | `404.php` | Contenido fijo del diseño |

`guias.html` es una página de apoyo: déjala como `page-guias.php` solo si el sitio la necesita.

### El Loop (blog y archivo)

```php
<?php if ( have_posts() ) : while ( have_posts() ) : the_post(); ?>
	<article <?php post_class(); ?>>
		<a href="<?php the_permalink(); ?>">
			<?php the_post_thumbnail( 'medium_large' ); ?>
			<h2><?php the_title(); ?></h2>
		</a>
		<p><?php echo esc_html( get_the_excerpt() ); ?></p>
	</article>
<?php endwhile; the_posts_pagination(); else : ?>
	<p>No hay artículos todavía.</p>
<?php endif; ?>
```

Copia las clases del HTML original en cada etiqueta para que el CSS siga funcionando.

## Paso 7: seguridad al imprimir datos

- Textos: `esc_html()`. Atributos: `esc_attr()`. Enlaces e imágenes: `esc_url()`.
- Todo archivo PHP parte con `if ( ! defined( 'ABSPATH' ) ) { exit; }` cuando no es una plantilla.

## Paso 8: probar y entregar

1. Comprime la carpeta `brotheme-shop/` en un `.zip` (la carpeta, no su contenido suelto).
2. En WordPress: Apariencia, Temas, Añadir nuevo, Subir tema.
3. Crea las páginas con los nombres (slugs) de la tabla y define la portada en Ajustes, Lectura.
4. Revisa en celular y escritorio, sin errores de PHP y sin enlaces rotos.
5. Corre Lighthouse y registra el resultado en el README.
6. Haz un commit por hito.

## Revisión final

- [ ] Ninguna ruta fija a `img/` ni `.html`.
- [ ] `wp_head()`, `wp_footer()` y `body_class()` presentes.
- [ ] Cada página del diseño tiene su plantilla.
- [ ] El menú se edita desde WordPress.
- [ ] El `.zip` se instala en un WordPress limpio sin errores.
