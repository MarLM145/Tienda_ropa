# Publicar növa con HTTPS (PythonAnywhere, gratis y temporal)

Guía para subir el proyecto unos días (por ejemplo, para entregarlo al profe) y darlo de baja después.
Usa una cuenta gratuita ("Beginner") de https://www.pythonanywhere.com. El nombre de usuario que elijas será parte de la
dirección: `https://USUARIO.pythonanywhere.com`. En esta guía, **reemplaza `USUARIO` por el tuyo** en todos los pasos.

Esto sirve para el requisito RNF-02 (conexión cifrada con HTTPS): la dirección `.pythonanywhere.com` ya tiene certificado.

---

## 1. Subir el proyecto

1. En PythonAnywhere abre la pestaña **Files** y usa **Upload a file** para subir `tienda-ropa.zip`
   (queda en `/home/USUARIO`).
2. Abre la pestaña **Consoles** → **Bash** y escribe estos comandos, uno por uno:

```bash
unzip -o tienda-ropa.zip
cd ~/tienda
mkvirtualenv --python=/usr/bin/python3.10 nova-env
pip install -r requirements.txt
python manage.py makemigrations accounts shop orders
python manage.py migrate
python manage.py seed_demo
python manage.py createsuperuser
python manage.py collectstatic --noinput
python -c "import secrets; print(secrets.token_urlsafe(50))"
```

- El último comando imprime una clave larga: **cópiala**, la usarás como `DJANGO_SECRET_KEY` en el paso 3.
- Si `python3.10` no existe en tu cuenta, usa otra versión disponible igual o mayor a 3.10 (por ejemplo `python3.12`) y
  elige **esa misma versión** en el paso 2.
- `createsuperuser` te pide correo, nombre, apellidos, teléfono y contraseña: es tu usuario administrador.

## 2. Crear la aplicación web

1. Pestaña **Web** → **Add a new web app** → **Next** → elige **Manual configuration** (NO "Django") → elige la misma
   versión de Python del paso anterior.
2. En la sección **Virtualenv** escribe `nova-env` y confirma.
3. En **Code**, pon en *Source code* y en *Working directory*: `/home/USUARIO/tienda`
4. En **Static files**, agrega dos filas:

| URL | Directory |
|---|---|
| `/static/` | `/home/USUARIO/tienda/staticfiles` |
| `/media/` | `/home/USUARIO/tienda/media` |

(La segunda es la que hace que se vean las fotos de los productos.)

## 3. Configurar el archivo WSGI

En la sección **Code** de la pestaña Web hay un enlace al archivo WSGI (algo como `/var/www/USUARIO_pythonanywhere_com_wsgi.py`).
Ábrelo, **borra todo su contenido** y pega esto, cambiando `USUARIO` y la clave:

```python
import os
import sys

path = "/home/USUARIO/tienda"
if path not in sys.path:
    sys.path.insert(0, path)

os.environ["DJANGO_SETTINGS_MODULE"] = "config.settings"
os.environ["DJANGO_DEBUG"] = "0"
os.environ["DJANGO_SECRET_KEY"] = "PEGA-AQUI-LA-CLAVE-LARGA-DEL-PASO-1"
os.environ["DJANGO_ALLOWED_HOSTS"] = "USUARIO.pythonanywhere.com"
os.environ["DJANGO_CSRF_ORIGINS"] = "https://USUARIO.pythonanywhere.com"
os.environ["SITE_URL"] = "https://USUARIO.pythonanywhere.com"
os.environ["DEMO_MODE"] = "1"   # deja visible el botón "simular pago en OXXO" para la demostración

# Correos con Gmail (opcional): quita el # de las 3 líneas y pon tus datos.
# Usa una "contraseña de aplicación" de Google, NO tu contraseña normal.
# os.environ["EMAIL_HOST"] = "smtp.gmail.com"
# os.environ["EMAIL_HOST_USER"] = "tucorreo@gmail.com"
# os.environ["EMAIL_HOST_PASSWORD"] = "la-clave-de-16-letras"

from django.core.wsgi import get_wsgi_application
application = get_wsgi_application()
```

Guarda el archivo (**Save**).

## 4. Activar HTTPS y encender

1. Pestaña **Web** → sección **Security** → activa el interruptor **Force HTTPS** (así, si alguien entra con `http://`, lo manda a
   `https://`). Si no ves la opción, igual puedes entrar escribiendo `https://` al inicio de la dirección.
2. Pulsa el botón verde **Reload** (arriba).
3. Abre `https://USUARIO.pythonanywhere.com`. Debe aparecer el candado en el navegador.

## 5. Qué probar antes de entregar

- Registro e inicio de sesión (con un error a propósito: las contraseñas no deben borrarse).
- Compra con la tarjeta `4242 4242 4242 4242` (cualquier fecha futura, CVC `123`) y revisa que llegue el correo
  (si activaste Gmail).
- Compra con OXXO y el botón de demostración.
- Entrar a `/admin/` con tu usuario administrador.
- Abrir la página desde el celular.

## Si algo falla

Abre el **Error log** (enlace en la pestaña Web) y busca la última línea del error:

| Síntoma | Causa y solución |
|---|---|
| `DisallowedHost` o página 400 | `DJANGO_ALLOWED_HOSTS` no coincide con tu dirección exacta. |
| "CSRF verification failed" al enviar un formulario | `DJANGO_CSRF_ORIGINS` debe ser `https://USUARIO.pythonanywhere.com` (con `https://`). |
| La página se ve sin estilos | Falta `collectstatic` o la fila `/static/` de la tabla del paso 2. |
| No se ven las fotos de los productos | Falta la fila `/media/` o ejecutar `seed_demo`. |
| `no such table` | Falta `python manage.py migrate` (en la consola Bash, dentro de `~/tienda`, con el virtualenv activo). |
| No llegan los correos | Revisa el Error log: casi siempre es una contraseña de aplicación mal copiada. Mira también el spam. |
| Cambiaste algo y no se nota | Pulsa **Reload** en la pestaña Web. |

Para ejecutar comandos de nuevo en una consola nueva: `cd ~/tienda` y `workon nova-env`.

## Actualizar el proyecto después

Sube otra vez el zip nuevo (Files → Upload), y en la consola Bash: `unzip -o tienda-ropa.zip`, luego
`cd ~/tienda && python manage.py collectstatic --noinput` y **Reload**. Tu base de datos no se toca (el zip no la incluye).

## Darlo de baja al terminar

1. Pestaña **Web** → baja hasta el final → **Delete** la aplicación web (el sitio deja de existir).
2. Si quieres borrar todo, en la pestaña **Account** elimina la cuenta.
3. Si usaste Gmail: entra a https://myaccount.google.com/apppasswords y **elimina** la contraseña de aplicación que creaste.

Mientras esté publicado, cualquier persona con el enlace puede registrarse y comprar (todo simulado). Evita compartirlo fuera del
profe y tus compañeros, y no uses contraseñas que uses en otros sitios.
