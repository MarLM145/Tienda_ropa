# növa — tienda de ropa en línea

Növa es una tienda de ropa desarrollada con **Django**, **SQLite** y **Bootstrap 5**. Incluye catálogo, carrito,
cuentas de usuario y un flujo de compra de demostración. Los pagos y las facturas no son reales ni tienen validez fiscal.

## Instalación y ejecución

Se necesita Python 3.10 o posterior. Desde la carpeta del proyecto, crea un entorno virtual e instala las dependencias:

```bash
python -m venv venv
source venv/bin/activate          # En Windows: venv\Scripts\activate
pip install -r requirements.txt
```

Después, prepara la base de datos y los datos de ejemplo:

```bash
python manage.py makemigrations accounts shop orders
python manage.py migrate
python manage.py seed_demo
python manage.py createsuperuser
```

Inicia el servidor local:

```bash
python manage.py runserver
```

- Tienda: http://127.0.0.1:8000/
- Panel de administración: http://127.0.0.1:8000/admin/
- Cuenta de prueba: `cliente@demo.com` / `Demo12345!`

La primera vez, ejecuta `makemigrations` con las tres aplicaciones antes de `migrate`. El proyecto usa un modelo de usuario
personalizado, que debe quedar incluido desde la primera migración.

## Productos de demostración

El comando `seed_demo` agrega 11 productos con imágenes, tallas y existencias. Los precios van de $199 a $1,199 MXN:

| Producto | Categoría | Precio |
|---|---|---|
| Pantuflas de felpa tejidas | Calzado / Pantuflas | $249 |
| Sandalias de espuma antiderrapantes | Calzado / Sandalias | $199 |
| Chamarra acolchada con capucha | Hombre / Chamarras | $1,199 |
| Pantalón de vestir tiro alto | Mujer / Pantalones | $649 |
| Set de 3 camisetas sin mangas | Hombre / Ropa interior | $449 |
| Jersey de béisbol con estampado | Hombre / Playeras | $499 |
| Set de 2 tops cuello cuadrado | Mujer / Blusas y tops | $329 |
| Paquete de 6 bóxers de algodón | Hombre / Ropa interior | $599 |
| Suéter polo de rayas azul | Mujer / Suéteres | $549 |
| Suéter polo de rayas rosa | Mujer / Suéteres | $549 |
| Playera básica negra | Hombre / Playeras | $219 |

Algunas tallas tienen poco stock o están agotadas para mostrar las alertas de inventario y la etiqueta «Agotado».

- Puedes volver a ejecutar `seed_demo`: actualiza los datos de los productos sin sobrescribir el stock y quita las categorías
  y productos de demostraciones anteriores.
- Las imágenes están en `shop/seed_images/`. Para reemplazarlas, súbelas desde el panel de administración, en **Productos →
  Imágenes**. Algunas imágenes de muestra tienen marcas visibles; conviene cambiarlas antes de usar fotos propias en una tienda.
- Si ya habías creado las migraciones, ejecuta `python manage.py makemigrations` y `python manage.py migrate` para aplicar
  los cambios recientes a la configuración de redes sociales.

## Redes sociales

Los enlaces a Instagram (`nova_lmmi`) y TikTok (`@user7580026901613`) aparecen en el encabezado y en el pie de página.
Puedes editarlos en **Configuración general** desde el panel de administración.

## Correo de confirmación de compra

El correo de confirmación incluye el folio, los artículos, el desglose del pedido, la dirección de entrega y un enlace al
detalle. Hay avisos para compras confirmadas con tarjeta o billetera, pedidos OXXO pendientes y pagos OXXO recibidos. Si falla
el envío del correo, el error queda registrado en el log.

- Por defecto, el correo se muestra en la terminal donde se ejecuta `runserver`.
- Para enviar correos con Gmail, activa la verificación en dos pasos, crea una *contraseña de aplicación* y define estas
  variables antes de iniciar el servidor:

  ```bash
  # Linux / Mac
  export EMAIL_HOST=smtp.gmail.com
  export EMAIL_HOST_USER=tucorreo@gmail.com
  export EMAIL_HOST_PASSWORD=la-clave-de-16-caracteres
  python manage.py runserver

  # Windows (PowerShell)
  $env:EMAIL_HOST="smtp.gmail.com"; $env:EMAIL_HOST_USER="tucorreo@gmail.com"; $env:EMAIL_HOST_PASSWORD="la-clave-de-16-caracteres"
  python manage.py runserver
  ```

  No guardes la contraseña de aplicación en el código ni la compartas. Para que los enlaces de los correos apunten al sitio
  publicado, configura también `SITE_URL=https://tu-dominio`. En local, el valor predeterminado es
  `http://127.0.0.1:8000`.
- Para comprobar el envío, registra una cuenta con un correo accesible y realiza una compra de prueba. Revisa también la
  carpeta de correo no deseado.

## Tarjetas de prueba (pasarela simulada)

| Número | Resultado |
|---|---|
| 4242 4242 4242 4242 (Visa) | Aprobada |
| 5555 5555 5555 4444 (Mastercard) | Aprobada |
| 3782 822463 10005 (Amex, CVC de 4 dígitos) | Aprobada |
| 4000 0000 0000 0002 | Rechazada |

Usa una fecha futura y un CVC de tres dígitos. En modo desarrollo, el detalle de un pedido OXXO incluye el botón
«Demo: simular que ya pagué en OXXO» (`DEBUG=1`). También puedes marcar el pedido como pagado desde el panel de
administración.

## Funciones principales

| Función | Implementación |
|---|---|
| REQ-01 Registro antes de comprar | Carrito y wishlist anónimos (por sesión); `/pedidos/checkout/` exige login y al entrar se fusionan con los del usuario |
| REQ-02 Login y recuperación | Login con correo (`accounts/forms.py`); recuperación con enlace por correo (en desarrollo el correo sale en la terminal) |
| REQ-03 Perfil | `/cuenta/perfil/`: pedidos con estatus, direcciones (con predeterminada), datos fiscales, descarga de comprobante y facturas |
| REQ-04 Catálogo y ficha | `shop/views.py`, categorías con subcategorías, variantes talla/color, inventario visible |
| REQ-05 Wishlist | `WishlistItem`; botón "Mover al carrito" |
| REQ-06 Stock y umbral | `Product.reorder_level`; `Variant.check_stock_alert()` crea `StockAlert` (visible en el admin); stock 0 muestra "Agotado" y deshabilita la compra |
| REQ-07 Reseñas | Solo compradores; quedan pendientes hasta que el admin las aprueba |
| REQ-08 Carrito | Persistente por usuario; desglose de subtotal, envío e IVA |
| REQ-09 Pagos | `orders/gateway.py` (tarjeta con token, Apple Pay, Google Pay, OXXO con vencimiento de 48 h) |
| REQ-10 Facturación | `orders/invoicing.py`: PDF + XML con estructura CFDI 4.0 **sin timbrado** |
| REQ-11/12 FAQ y políticas | Editables en el admin (`FAQ`, `StaticPage`) |
| REQ-13 Redes sociales | Íconos en encabezado y pie (se configuran en el admin); botones para compartir en la ficha |
| REQ-14/15/16 Admin | Django admin: productos con variantes e imágenes, pausar/activar, pedidos con estatus y código de rastreo, configuración general |
| RNF-02 Seguridad | Contraseñas con PBKDF2; HTTPS al desplegar con `DJANGO_DEBUG=0`; se conserva el token, la marca y los últimos cuatro dígitos, no el número completo de la tarjeta |
| RNF-03 Diseño adaptable | Bootstrap 5 para móvil y escritorio |

## Alcance de la demostración

- **Pagos:** el flujo de tarjeta, Apple Pay, Google Pay y OXXO es simulado; no se realiza ningún cargo.
- **Facturas:** el sistema genera archivos PDF y XML de muestra, sin timbrado ni validez fiscal. Para emitir CFDI reales se
  necesita un PAC y los certificados de sello digital del emisor. Los datos incluidos corresponden a datos de prueba públicos
  del SAT.

## Reglas de la tienda

- El IVA (16 %) se calcula sobre productos + envío; los precios del catálogo no lo incluyen.
- El envío cuesta $99 y es gratis desde $999 (ajustable en `config/settings.py`).
- El inventario se descuenta al crear el pedido y se repone si se cancela o vence una ficha OXXO. Para revisar pedidos vencidos,
  ejecuta `python manage.py expire_oxxo`; el comando se puede programar para correr cada hora.
- El umbral de reabastecimiento se configura por producto y se compara con el stock de cada variante.

## Despliegue con HTTPS (RNF-01 / RNF-02)

Los pasos para publicar el proyecto en PythonAnywhere están en [`DESPLIEGUE.md`](DESPLIEGUE.md). Estas son las variables de
entorno disponibles:

| Variable | Para qué sirve |
|---|---|
| `DJANGO_DEBUG=0` | Modo producción (cookies seguras, sin mensajes de depuración) |
| `DJANGO_SECRET_KEY` | Clave secreta larga y aleatoria |
| `DJANGO_ALLOWED_HOSTS` | Tu dominio, por ejemplo `usuario.pythonanywhere.com` |
| `DJANGO_CSRF_ORIGINS` | `https://` + tu dominio |
| `SITE_URL` | `https://` + tu dominio (enlaces de correos y de compartir) |
| `DEMO_MODE=1` | Deja visible el botón "simular pago en OXXO" aunque `DJANGO_DEBUG=0` |
| `EMAIL_HOST`, `EMAIL_HOST_USER`, `EMAIL_HOST_PASSWORD` | Envío de correos por SMTP (Gmail) |
| `DJANGO_SSL_REDIRECT=1` | Solo si tu hosting no redirige de http a https por su cuenta |

## Comprobaciones recomendadas

1. Crear una cuenta, cerrar sesión, volver a entrar y probar la recuperación de contraseña.
2. Añadir productos al carrito sin iniciar sesión; luego entrar al checkout y comprobar que el carrito se conserve.
3. Probar una compra aprobada y otra rechazada, y descargar el comprobante PDF.
4. Completar el flujo OXXO y revisar el cambio de estado desde el panel de administración.
5. Solicitar una factura de muestra y descargar los archivos PDF y XML.
6. Cambiar el stock de una variante para comprobar la alerta de reabastecimiento y el estado «Agotado».
7. Enviar una reseña después de una compra y aprobarla desde el panel de administración.
8. Revisar las páginas principales en una pantalla móvil.y XML.
6. Bajar el stock de una variante al umbral y ver la alerta en el admin; stock 0 → "Agotado".
7. Reseña: solo tras comprar, y aprobación en el admin.
8. Revisar todo desde el celular (o las herramientas de desarrollo del navegador).
