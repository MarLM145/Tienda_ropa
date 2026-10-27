# növa · tienda de ropa en línea (proyecto escolar)

Tienda hecha con **Django (Python)**, base de datos SQLite y Bootstrap 5 (diseño responsivo).
Los pagos y las facturas están **simulados**: es un proyecto escolar, no una tienda real.

> **Estado de la verificación:** el código fue escrito sin poder ejecutar Django en el entorno de desarrollo
> (no había acceso a PyPI). Se verificó la sintaxis de todos los archivos Python y se probó por separado la lógica de
> la pasarela simulada y la generación de PDF/XML. El flujo completo (migraciones, vistas, plantillas) **hay que
> probarlo localmente** siguiendo la lista de pruebas de abajo y corregir lo que aparezca.

## Puesta en marcha

Requiere Python 3.10 o superior.

```bash
python -m venv venv
source venv/bin/activate          # En Windows: venv\Scripts\activate
pip install -r requirements.txt

python manage.py makemigrations accounts shop orders
python manage.py migrate
python manage.py seed_demo         # 11 productos con foto y precio, FAQ, políticas y redes sociales
python manage.py createsuperuser   # tu usuario administrador (correo + contraseña)
python manage.py runserver
```

- Tienda: http://127.0.0.1:8000/
- Panel de administración: http://127.0.0.1:8000/admin/
- Cliente de demostración: `cliente@demo.com` / `Demo12345!`

> Importante: ejecuta `makemigrations` con los tres nombres de app **antes** del primer `migrate`
> (el proyecto usa un modelo de usuario personalizado, así que no se puede agregar después).

## Productos de demostración

`seed_demo` carga **11 productos con foto, precio, tallas y stock** (de $199 a $1,199 MXN), organizados en las categorías
Mujer, Hombre y Calzado:

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

Algunas tallas quedan con poco stock o agotadas a propósito, para que veas las alertas de reabastecimiento en el admin y la
etiqueta "Agotado" en la tienda.

- Es seguro volver a ejecutarlo: actualiza precio, descripción y foto sin pisar el stock que ya hayas movido, y elimina los
  productos y categorías de demostración de versiones anteriores.
- Las fotos están en `shop/seed_images/` (600 x 800 px, fondo blanco). Son fotos de terceros que traen marcas visibles: están
  bien para un proyecto escolar, pero no para una tienda real. Para cambiarlas, súbelas desde el admin (Productos → Imágenes).
- Si ya habías ejecutado `makemigrations` antes de esta versión, vuelve a correr `python manage.py makemigrations` y
  `python manage.py migrate` (solo cambian los valores por defecto de las redes sociales).

## Redes sociales

El pie de página tiene una sección **Redes sociales** con Instagram (`nova_lmmi`) y TikTok (`@user7580026901613`);
los mismos íconos aparecen arriba en la barra superior. Se editan desde el admin en *Configuración general*.

## Correo de confirmación de compra

Al comprar, el cliente recibe un correo con el folio, los artículos, el desglose (subtotal, envío, IVA y total), la dirección de
entrega y el enlace a su pedido. Hay tres variantes: compra con tarjeta o billetera (confirmada), pedido con OXXO (con la
referencia y la fecha de vencimiento) y pago OXXO recibido. Si el correo falla, la compra no se rompe: el error queda en el log.

- **Sin configurar nada**, el correo se imprime en la terminal donde corre `runserver` (sirve para ver cómo queda).
- **Para enviarlo de verdad con Gmail:** activa la verificación en dos pasos en tu cuenta de Google, crea una *contraseña de
  aplicación* (16 caracteres) y define estas variables **antes** de arrancar el servidor:

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

  Nunca escribas la clave dentro del código ni la compartas. El remitente será tu cuenta de Gmail (límite aproximado de 500
  correos al día). Para que los enlaces del correo apunten a tu sitio publicado, define también
  `SITE_URL=https://tu-dominio` (por defecto es `http://127.0.0.1:8000`).
- **Cómo probarlo:** crea una cuenta con tu correo real, compra con la tarjeta `4242 4242 4242 4242` y revisa tu bandeja
  (y spam). Repite con OXXO y luego usa el botón de demostración (o la acción del admin) para recibir el aviso de pago.

## Tarjetas de prueba (pasarela simulada)

| Número | Resultado |
|---|---|
| 4242 4242 4242 4242 (Visa) | Aprobada |
| 5555 5555 5555 4444 (Mastercard) | Aprobada |
| 3782 822463 10005 (Amex, CVC de 4 dígitos) | Aprobada |
| 4000 0000 0000 0002 | Rechazada |

Usa cualquier fecha futura y CVC de 3 dígitos. Para OXXO, en modo desarrollo el detalle del pedido muestra un botón
"Demo: simular que ya pagué en OXXO" (solo con `DEBUG=1`); en el admin hay una acción "Marcar como pagado".

## Dónde está cada requerimiento

| Requerimiento | Implementación |
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
| RNF-02 Seguridad | Contraseñas con PBKDF2; HTTPS al desplegar con `DJANGO_DEBUG=0`; nunca se guarda el número de tarjeta, solo token, marca y últimos 4 |
| RNF-03 Responsivo | Bootstrap 5, probado en el diseño para móvil y escritorio |

## Qué es simulado (dilo en tu documentación)

- **Pasarela de pago:** imita el flujo de Stripe/Mercado Pago. En una integración real la tokenización la hace
  Stripe.js en el navegador y el servidor nunca ve la tarjeta (así se cumple PCI-DSS). Apple Pay y Google Pay
  también se autorizan de forma simulada.
- **Facturación:** el UUID se genera localmente y los documentos dicen "sin validez fiscal". Para timbrar de verdad
  hace falta un PAC y los certificados de sello digital (CSD) del emisor. Los datos del emisor son los de prueba
  públicos del SAT.

## Decisiones de diseño

- El IVA (16 %) se calcula sobre productos + envío; los precios del catálogo no lo incluyen.
- El envío cuesta $99 y es gratis desde $999 (ajustable en `config/settings.py`).
- El inventario se descuenta al crear el pedido y se regresa si se cancela o vence la ficha OXXO
  (`python manage.py expire_oxxo`, programable cada hora).
- El umbral de reabastecimiento se define por producto y se compara contra el stock de **cada variante**
  (así avisa cuando se acaba, por ejemplo, la talla M).

## Despliegue con HTTPS (RNF-01 / RNF-02)

La guía completa, paso a paso, está en **`DESPLIEGUE.md`** (PythonAnywhere, plan gratuito, con HTTPS en
`https://USUARIO.pythonanywhere.com`, correos con Gmail y cómo darlo de baja). Las variables de entorno que usa el proyecto:

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

## Lista de pruebas sugerida (en este orden)

1. Registro, cierre de sesión, login y recuperación de contraseña (el enlace aparece en la terminal).
2. Agregar productos al carrito sin sesión, ir al checkout, iniciar sesión y confirmar que el carrito sigue ahí.
3. Compra con tarjeta aprobada y rechazada; comprobante PDF.
4. Compra con OXXO: ficha, botón de demo, y cambio de estatus desde el admin.
5. Solicitar factura desde el checkout y desde el historial; descargar PDF y XML.
6. Bajar el stock de una variante al umbral y ver la alerta en el admin; stock 0 → "Agotado".
7. Reseña: solo tras comprar, y aprobación en el admin.
8. Revisar todo desde el celular (o las herramientas de desarrollo del navegador).
