# Hacer que el tema brotheme-shop sea compatible con los plugins

Instrucciones para que el tema propio funcione con los plugins del proyecto. Un plugin no cambia el diseño por sí solo: el tema debe dejarle los "ganchos" que necesita y darle estilo al resultado.

## Lo que todo plugin necesita del tema

Si falta alguno de estos, muchos plugins fallan sin avisar:

| Requisito | Dónde |
|---|---|
| `<?php wp_head(); ?>` antes de `</head>` | `header.php` |
| `<?php wp_footer(); ?>` antes de `</body>` | `footer.php` |
| `<body <?php body_class(); ?>>` | `header.php` |
| `add_theme_support( 'title-tag' )` | `functions.php` |
| Estilos y scripts con `wp_enqueue_style` y `wp_enqueue_script` | `functions.php` |
| Sin `<title>` fijo ni `<meta name="description">` fijo | `header.php` |

## 1. Yoast SEO (SEO, obligatorio)

- Yoast escribe el título, la descripción y las etiquetas sociales con `wp_head()`. Borra del HTML cualquier `<title>` y `<meta name="description">` fijo.
- Migas de pan (opcional): activa "Migas de pan" en Yoast y pon en la plantilla:

```php
<?php
if ( function_exists( 'yoast_breadcrumb' ) ) {
	yoast_breadcrumb( '<nav class="migas" aria-label="Migas de pan">', '</nav>' );
}
?>
```

- Cada página y entrada usa un solo `<h1>` (el `the_title()`).
- Las imágenes llevan `alt`; en entradas se imprime con `the_post_thumbnail()`.
- Completa en cada entrada la palabra clave y la descripción desde el cuadro de Yoast.

## 2. Contact Form 7 (contacto, obligatorio)

- Crea el formulario en Contactos, Formularios, con el título **Contacto** y los campos del diseño: nombre, correo, asunto y mensaje.
- En `page-contacto.php`, en el lugar del formulario del HTML:

```php
<?php echo do_shortcode( '[contact-form-7 title="Contacto"]' ); ?>
```

Usa el `title` y no el `id`: el id cambia entre instalaciones.

- Para conservar el diseño, escribe el formulario en CF7 con las mismas clases del HTML:

```html
<label class="campo">Nombre
  [text* tu-nombre class:input]</label>
<label class="campo">Asunto
  [select* tu-asunto class:input "Consulta" "Pedido" "Otro"]</label>
<label class="campo">Mensaje
  [textarea* tu-mensaje class:input]</label>
[submit class:boton "Enviar"]
```

- Dale estilo a lo que CF7 agrega: `.wpcf7-form`, `.wpcf7-not-valid-tip`, `.wpcf7-response-output`, `.wpcf7-spinner`.
- En la pestaña Correo pon el correo del emprendimiento en "Para". Prueba el envío (en InfinityFree el correo puede demorar o fallar; ver "Correo" más abajo).

## 3. Akismet (antispam)

- Instálalo, actívalo y conecta una clave gratuita (uso personal).
- Protege los comentarios y los envíos de Contact Form 7.
- Para que los comentarios del blog funcionen, `single.php` debe llamar:

```php
<?php if ( comments_open() || get_comments_number() ) { comments_template(); } ?>
```

- Si usas un `comments.php` propio, imprime los comentarios con `wp_list_comments()` y el formulario con `comment_form()`.
- `add_theme_support( 'html5', array( 'comment-form', 'comment-list' ) )` ya viene en `functions.php`.

## 4. Newsletter (suscripción por correo)

- Instálalo y crea el formulario de suscripción desde el plugin.
- Pega el formulario donde el diseño tiene la captación (Inicio y Artículo):

```php
<?php echo do_shortcode( '[newsletter_form]' ); ?>
```

- Si el diseño ya tiene su propio formulario, copia sus clases al formulario del plugin y estiliza `.tnp-subscription` y sus campos (`.tnp-field`, `.tnp-submit`).
- Activa la casilla de consentimiento y la confirmación por correo.
- Muestra el enlace de la guía gratuita y el código del 10 % en el mensaje de "gracias" de la misma página. Así no dependes de que el correo llegue.

## 5. All-in-One WP Migration (respaldo y migración)

- El tema debe quedar en el respaldo y reinstalarse en otro servidor sin romperse. Para eso:
  - ninguna URL fija (`http://...` del sitio), solo `home_url()` y `get_template_directory_uri()`;
  - ninguna ruta absoluta del servidor.
- Para respaldar: Exportar, Exportar a, Archivo. Se descarga un `.wpress`.
- Para reinstalar: instala WordPress limpio, instala el mismo plugin, Importar y sube el `.wpress`.
- La versión gratis limita el tamaño de importación según el hosting. Comprime las imágenes para que el respaldo no crezca de más.
- Guarda un respaldo al cerrar cada etapa y documenta estos pasos en el README del sitio.

## Correo en InfinityFree

El hosting limita el correo que sale del servidor. Si CF7 o Newsletter no envían:

1. Revisa la carpeta de spam.
2. Instala WP Mail SMTP y configúralo con una cuenta de correo propia.
3. Como plan B, deja el correo del emprendimiento visible en la página de Contacto.

## Plugins opcionales

| Plugin | Qué necesita el tema |
|---|---|
| WP Mail SMTP | Nada; configúralo desde el panel |
| Joinchat (WhatsApp) | `wp_footer()` presente |
| WP Super Cache | Estilos y scripts bien encolados; vacía el caché al cambiar el tema |
| Smush o EWWW Image Optimizer | Imágenes subidas a la biblioteca de medios, no pegadas en el tema |
| Relevanssi | Un `searchform.php` o `get_search_form()` y un `search.php` |
| CookieYes | `wp_footer()` presente |

## Probar la compatibilidad

1. Activa los plugins uno por uno y revisa el sitio tras cada uno.
2. Si aparece una pantalla blanca, renombra la carpeta del plugin por FTP para desactivarlo.
3. Revisa que no haya errores en la consola del navegador (F12).
4. Comprueba: título y descripción por página (Yoast), envío del formulario (CF7), comentario de prueba (Akismet), suscripción de prueba (Newsletter) y un respaldo exportado (All-in-One).
5. Corre Lighthouse con los plugins activos.
