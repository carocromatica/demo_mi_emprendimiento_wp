# plugins: instrucciones para el agente

Eres un agente de código (Antigravity). Tu tarea es dejar el tema `brotheme-shop` **compatible con los plugins del proyecto**. Un plugin no cambia el diseño por sí solo: el tema debe darle los ganchos que necesita y estilizar su resultado. Este archivo se usa después de `html-to-theme.md` (hito 5) o sobre un tema ya creado.

## Objetivo

Que el tema funcione sin errores con **Yoast SEO, Contact Form 7, Akismet, Newsletter y All-in-One WP Migration**, conservando el diseño de Brote.

## Entradas

- `brotheme-shop/`: el tema a ajustar.
- `proyecto/html/css/styles.css` y `proyecto/html/contacto.html`: clases y estructura del formulario de contacto y de la captación de correos del diseño.
- `proyecto/decisiones.md`: plugins elegidos.

## Salida

Cambios solo dentro de `brotheme-shop/` y un bloque "Plugins" en el README del repo con la configuración que debe hacer la estudiante desde el panel de WordPress (lo que no se puede dejar en código).

## Reglas (no las rompas)

1. **No instales ni descargues plugins.** Solo prepara el tema. La estudiante los instala desde el panel.
2. **No toques archivos de plugins** ni `wp-content/plugins/`. Todo ajuste va en el tema.
3. **No cambies el diseño.** Si un plugin imprime HTML propio, estilízalo con CSS para que calce; no reescribas la página.
4. **Nada de rutas ni URL fijas del sitio.** Usa `home_url()`, `get_permalink()` y `get_template_directory_uri()`.
5. **Escapa todo lo que imprimas:** `esc_html()`, `esc_attr()`, `esc_url()`.
6. **Cambios pequeños en `functions.php`** y revisión de sintaxis (`php -l`) después de cada edición.
7. **Prefijo `brotheme_`** en funciones y handles.
8. **Un commit por hito**, con mensaje en español.

## Procedimiento

### Hito 1: requisitos generales del tema

Verifica, y corrige si falta, cada punto. Si uno no se cumple, muchos plugins fallan sin avisar.

| Requisito | Dónde | Cómo verificar |
|---|---|---|
| `<?php wp_head(); ?>` antes de `</head>` | `header.php` | `grep -n "wp_head" brotheme-shop/header.php` |
| `<?php wp_footer(); ?>` antes de `</body>` | `footer.php` | `grep -n "wp_footer" brotheme-shop/footer.php` |
| `<body <?php body_class(); ?>>` | `header.php` | `grep -n "body_class" brotheme-shop/header.php` |
| `add_theme_support( 'title-tag' )` | `functions.php` | `grep -n "title-tag" brotheme-shop/functions.php` |
| Estilos y scripts con `wp_enqueue_style` y `wp_enqueue_script` | `functions.php` | Sin `<link>` ni `<script>` pegados en las plantillas |
| Sin `<title>` ni `<meta name="description">` fijos | `header.php` | `grep -n "<title\|name=\"description\"" brotheme-shop/header.php` no devuelve nada |

**Criterio:** los seis requisitos se cumplen.
**Commit:** `Plugins: requisitos generales del tema`.

### Hito 2: Yoast SEO

1. Confirma que no hay `<title>` ni `<meta name="description">` fijos (hito 1).
2. Si el diseño tiene migas de pan, imprímelas con Yoast en las plantillas donde aparezcan (`single.php`, `archive.php`, `page.php`, páginas de tienda):

```php
<?php
if ( function_exists( 'yoast_breadcrumb' ) ) {
	yoast_breadcrumb( '<nav class="migas" aria-label="Migas de pan">', '</nav>' );
}
?>
```

Usa la clase que ya tenga el HTML original para las migas. La función solo imprime si la estudiante activa "Migas de pan" en Yoast, y no rompe nada si el plugin no está instalado.

3. Verifica que cada plantilla tenga un solo `<h1>` (el título de la página o entrada con `the_title()`).
4. Verifica que toda imagen impresa por PHP tenga `alt` (`the_post_thumbnail()` lo maneja; en imágenes del tema escribe un `alt` descriptivo).

**Criterio:** una sola etiqueta `<h1>` por plantilla; sin `<title>` fijo; migas con `function_exists`.
**Commit:** `Plugins: compatibilidad con Yoast SEO`.

### Hito 3: Contact Form 7

1. En `page-contacto.php`, en el lugar del formulario del HTML original, imprime:

```php
<?php echo do_shortcode( '[contact-form-7 title="Contacto"]' ); ?>
```

Usa `title` y no `id`: el id cambia entre instalaciones.

2. Crea el archivo `proyecto/context/contacto-cf7.txt` con el contenido del formulario para que la estudiante lo pegue en CF7. Usa las mismas clases del HTML original:

```html
<label class="campo">Nombre
  [text* tu-nombre class:input]</label>
<label class="campo">Correo
  [email* tu-correo class:input]</label>
<label class="campo">Asunto
  [select* tu-asunto class:input "Consulta" "Pedido" "Otro"]</label>
<label class="campo">Mensaje
  [textarea* tu-mensaje class:input]</label>
[submit class:boton "Enviar"]
```

Ajusta los nombres de clase (`campo`, `input`, `boton`) a las que realmente use `contacto.html`; léelo antes de escribir.

3. Agrega a `assets/css/styles.css` estilos para lo que CF7 imprime, con las variables y colores ya existentes: `.wpcf7-form`, `.wpcf7-not-valid-tip`, `.wpcf7-response-output`, `.wpcf7-spinner`.

**Criterio:** la plantilla usa el shortcode por título; existe el archivo del formulario con clases reales del diseño; los estilos de `.wpcf7-*` están en el CSS.
**Commit:** `Plugins: Contact Form 7 en Contacto`.

### Hito 4: Akismet y comentarios

1. En `single.php`, después del contenido, imprime los comentarios:

```php
<?php
if ( comments_open() || get_comments_number() ) {
	comments_template();
}
?>
```

2. Si el tema no tiene `comments.php`, créalo con `wp_list_comments()` para la lista y `comment_form()` para el formulario, con las clases del diseño. Si ya existe, déjalo.
3. Confirma que `functions.php` tiene `add_theme_support( 'html5', array( 'comment-form', 'comment-list' ) )`.
4. Agrega estilos para la lista y el formulario de comentarios en el CSS.

Akismet no necesita código en el tema: filtra los comentarios y los envíos de CF7 una vez que la estudiante lo activa y conecta su clave.

**Criterio:** `single.php` llama `comments_template()`; existe `comments.php` con `wp_list_comments()` y `comment_form()`.
**Commit:** `Plugins: comentarios y Akismet`.

### Hito 5: Newsletter

1. En `front-page.php` y `single.php`, donde el diseño tiene el bloque de captación de correos, reemplaza el formulario fijo por:

```php
<?php echo do_shortcode( '[newsletter_form]' ); ?>
```

Conserva el título, el texto y el contenedor del bloque original; reemplaza solo el `<form>`.

2. Agrega a `assets/css/styles.css` estilos para el formulario del plugin (`.tnp-subscription`, `.tnp-field`, `.tnp-submit`) para que calce con el diseño.
3. Revisa los nombres de clase reales en el HTML que genera el plugin; si no coinciden con los de arriba, ajusta los selectores. Si no puedes verificarlo sin WordPress, déjalo anotado en el resumen final.

**Criterio:** el bloque de captación usa el shortcode en las dos plantillas; hay estilos para el formulario del plugin.
**Commit:** `Plugins: formulario de Newsletter`.

### Hito 6: All-in-One WP Migration (respaldo)

El tema debe sobrevivir a una exportación e importación en otro servidor. Verifica en todo `brotheme-shop/`:

| Revisión | Comando | Resultado esperado |
|---|---|---|
| Sin URL fijas del sitio | `grep -rn "http://\|https://" brotheme-shop --include=*.php` | Solo URL externas legítimas (fuentes, redes), nunca el dominio del sitio |
| Sin rutas absolutas del servidor | `grep -rn "/home/\|/var/www\|C:\\\\" brotheme-shop` | Sin resultados |
| Sin `localhost` | `grep -rn "localhost" brotheme-shop` | Sin resultados |
| Sin imágenes con ruta relativa | `grep -rn 'src="img/\|src="/img/' brotheme-shop` | Sin resultados |

Corrige cada hallazgo con `home_url()` o `get_template_directory_uri()`.

**Criterio:** las cuatro revisiones salen limpias.
**Commit:** `Plugins: tema sin URL fijas para migrar`.

### Hito 7: README y verificación

1. Revisa la sintaxis de todos los PHP (`php -l`), si está disponible.
2. Agrega al README del repo un bloque "Plugins" con lo que la estudiante hace desde el panel:
   - **Yoast SEO:** activar migas de pan; completar palabra clave y descripción de cada página y entrada.
   - **Contact Form 7:** crear el formulario con título `Contacto` y pegar el contenido de `contacto-cf7.txt`; poner el correo del emprendimiento en "Para".
   - **Akismet:** activar y conectar una clave gratuita (uso personal).
   - **Newsletter:** crear el formulario de suscripción, activar la casilla de consentimiento y la confirmación por correo; poner el enlace de la guía y el código del 10 % en el mensaje de "gracias".
   - **All-in-One WP Migration:** exportar a archivo (`.wpress`) al cerrar cada etapa; para reinstalar, instalar WordPress limpio y el mismo plugin e importar el archivo.
3. Agrega la nota del correo en InfinityFree (abajo).

**Criterio:** los PHP pasan la revisión y el README tiene el bloque "Plugins".
**Commit:** `Plugins: documentación en el README`.

## Correo en InfinityFree

El hosting limita el correo que sale del servidor. No puedes arreglarlo desde el tema. Deja esta nota en el README:

1. Si CF7 o Newsletter no envían, revisar la carpeta de spam.
2. Instalar WP Mail SMTP y configurarlo con una cuenta de correo propia.
3. Como plan B, dejar el correo del emprendimiento visible en la página de Contacto.

## Plugins opcionales (solo si la estudiante los eligió en `decisiones.md`)

| Plugin | Qué hacer en el tema |
|---|---|
| WP Mail SMTP | Nada en el tema; se configura desde el panel |
| Joinchat (WhatsApp) | Verificar `wp_footer()` |
| WP Super Cache | Verificar que estilos y scripts estén bien encolados |
| Smush o EWWW Image Optimizer | Nada en el tema; las imágenes de contenido van a la biblioteca de medios |
| Relevanssi | Agregar `searchform.php` (o `get_search_form()`) y `search.php` con el diseño del buscador |
| CookieYes | Verificar `wp_footer()` |

## Cuándo detenerte y preguntar

Detente y pregunta a la estudiante, sin adivinar, si:

- el HTML original no tiene un formulario de contacto o un bloque de captación de correos que reemplazar;
- el diseño del formulario necesita campos que CF7 no puede generar;
- tienes que cambiar el diseño para que un plugin funcione;
- necesitas un plugin que no está en `decisiones.md`.

## Al terminar

Entrega un resumen corto con: los archivos cambiados, lo que dejaste listo para configurar desde el panel, lo que no pudiste verificar sin WordPress instalado (por ejemplo las clases reales que imprime Newsletter) y cualquier duda abierta.
