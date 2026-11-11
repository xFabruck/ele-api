# Ele API

Backend de **Ele**, desarrollado con Django y Django REST Framework. Lo consume la app móvil (React Native + Expo).

## Stack

- Python 3.12 o superior
- Django + Django REST Framework
- SimpleJWT (autenticación con tokens)
- PostgreSQL
- drf-spectacular (documentación OpenAPI)

## Requisitos previos

Instala en tu computador:

- [Python 3.12+](https://www.python.org/downloads/) (en Windows, marca **"Add python.exe to PATH"** al instalar)
- [PostgreSQL 16+](https://www.postgresql.org/download/) con una contraseña para el usuario `postgres`
- [Git](https://git-scm.com/)

Verifica:

```bash
python --version
psql --version
git --version
```

## Puesta en marcha local

### 1. Clonar el repositorio

```bash
git clone https://github.com/[ORGANIZACION]/ele-api.git
cd ele-api
```

### 2. Crear y activar el entorno virtual

Windows (PowerShell):

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

Si PowerShell bloquea el script, ejecuta una sola vez:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

macOS / Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Cuando esté activo verás `(.venv)` al inicio de la línea de comandos.

### 3. Instalar dependencias

```bash
pip install -r requirements.txt
```

### 4. Crear la base de datos

```bash
psql -U postgres
```

Dentro de `psql`:

```sql
CREATE DATABASE ele;
\q
```

### 5. Configurar las variables de entorno

Copia el archivo de ejemplo:

```bash
cp .env.example .env          # macOS / Linux
copy .env.example .env        # Windows (cmd)
Copy-Item .env.example .env   # Windows (PowerShell)
```

Abre `.env` y completa los valores:

```
SECRET_KEY=
DEBUG=True
DATABASE_URL=postgres://postgres:TU_CONTRASEÑA@localhost:5432/ele
```

Para generar la `SECRET_KEY`:

```bash
python -c "from django.core.management.utils import get_random_secret_key as g; print(g())"
```

Pega el resultado en `SECRET_KEY=`.

Notas:
- El archivo `.env` es personal. **Nunca se sube a Git.**
- Si tu contraseña de PostgreSQL tiene caracteres especiales (`@`, `#`, `/`), codifícalos en la URL o usa una contraseña sencilla para desarrollo.

### 6. Aplicar migraciones

```bash
python manage.py migrate
```

### 7. Crear un superusuario

```bash
python manage.py createsuperuser
```

### 8. Iniciar el servidor

```bash
python manage.py runserver
```

Abre:

- Panel de administración: http://localhost:8000/admin
- Documentación de la API: http://localhost:8000/api/docs/ *(disponible cuando se configure Swagger)*

## Estructura del proyecto

```
ele-api/
├── config/            # configuración del proyecto (settings, urls)
├── users/             # usuarios y roles (cliente, repartidor, admin)
├── manage.py
├── requirements.txt
├── .env.example       # plantilla de variables de entorno
└── .gitignore
```

Cada dominio nuevo (pedidos, repartidores, tiendas) será una app propia dentro del proyecto.

## Roles de usuario

| Rol | Descripción |
|---|---|
| `cliente` | Realiza pedidos. Es el rol por defecto al registrarse. |
| `repartidor` | Entrega pedidos. Lo asigna un administrador. |
| `admin` | Gestiona la plataforma. |

## Comandos útiles

```bash
python manage.py makemigrations     # generar migraciones tras cambiar modelos
python manage.py migrate            # aplicar migraciones
python manage.py check              # validar la configuración
python manage.py test               # ejecutar pruebas
pip freeze > requirements.txt       # actualizar dependencias (tras instalar algo nuevo)
```

## Flujo de trabajo con Git

- Ramas permanentes: `main` (estable) y `develop` (integración).
- Cada tarea tiene su propia rama, creada desde `develop`: `feat/...`, `fix/...`, `chore/...`.
- Nadie hace push directo a `main` ni a `develop`: todo entra por **Pull Request** con al menos una revisión.
- Mensajes de commit con formato `tipo: descripción` (`feat:`, `fix:`, `chore:`, `docs:`, `refactor:`).

```bash
git switch develop
git pull
git switch -c feat/nombre-de-la-tarea
```

## Problemas frecuentes

**`SECRET_KEY` vacía o `None`**
El archivo `.env` no existe, no está junto a `manage.py` o no tiene la variable. En Windows verifica que no se haya guardado como `.env.txt`.

**`SyntaxError` al importar `environ`**
Hay instalado el paquete equivocado. Corrige con:
```bash
pip uninstall -y environ django-environ
pip install django-environ
```

**`password authentication failed` o `database "ele" does not exist`**
Revisa la contraseña en `DATABASE_URL` y que hayas creado la base `ele` (paso 4).

**`connection refused`**
El servicio de PostgreSQL no está corriendo. En Windows, revísalo en *Servicios*.

**`python` no se reconoce**
Cierra y abre la terminal. Si persiste, usa `py` en lugar de `python` o agrega Python al `Path`.

**El comando `Activate.ps1` está bloqueado**
Ejecuta `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`.

## Seguridad

- No subas `.env`, claves ni contraseñas al repositorio.
- Cada entorno (desarrollo, pruebas, producción) usa su propia `SECRET_KEY` y su propia base de datos.
- Si una clave se filtra, **se revoca y se genera otra**; borrarla del historial de Git no basta.