# 🧾 DEPLOY COMPLETO – Proyecto Django + Docker + CI/CD (con cambios personalizados)

## 🧩 1️⃣ Preparación del entorno

Se utilizó como base el proyecto open source:
👉 [amerkurev/django-docker-template](https://github.com/amerkurev/django-docker-template)

Este proyecto fue clonado y adaptado para el desarrollo de un pipeline de **Integración Continua (CI)** y **Despliegue Continuo (CD)** con **Django + Docker + GitHub Actions**.

### 📂 Estructura final del proyecto:
```
manage.py
website/
  ├── settings.py
  ├── urls.py     ← modificado
  └── ...
polls/
Dockerfile
docker-compose.yml
.github/workflows/ci.yml
requirements.txt
```

---

## ⚙️ 2️⃣ Configuración del CI con GitHub Actions

Archivo creado: `.github/workflows/ci.yml`

```yaml
name: build

on:
  push:
    branches:
      - master
    tags:
      - '*'
    paths-ignore:
      - '*.md'
  pull_request:
    paths-ignore:
      - '*.md'

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        id: buildx
        uses: docker/setup-buildx-action@v2

      - name: Test and submit coverage report
        run: |
          docker build -t django-docker-template:master .
          docker run --rm -v $(pwd)/website/:/usr/src/website django-docker-template:master ./pytest.sh
          pip install coveralls
          cd website && coveralls --service=github
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

✅ Ejecuta automáticamente la construcción de la imagen Docker y los tests con pytest.

---

## 🧱 3️⃣ Configuración Docker y Traefik

### `Dockerfile`
Basado en Python 3.10, instala dependencias y ejecuta Gunicorn para Django.

### `docker-compose.yml`
Incluye servicios:
- Django (app principal)
- PostgreSQL (base de datos)
- Traefik (proxy inverso)
- Nginx (archivos estáticos)

Configuración relevante:
```yaml
django:
  build: .
  image: django-docker
  env_file: .env
  restart: unless-stopped
  volumes:
    - "staticfiles-data:/var/www/static"
    - "media-data:/var/www/media"
  depends_on:
    - postgres
  labels:
    - "traefik.enable=true"
    - "traefik.http.routers.development.rule=Host(`127.0.0.1`) || Host(`localhost`)"
    - "traefik.http.routers.development.entrypoints=web"
    - "traefik.http.routers.development.priority=1"
```

---

## 💡 4️⃣ Modificación de `urls.py`

Archivo: `website/urls.py`

```python
from django.contrib import admin
from django.urls import path, include
from django.http import HttpResponse

def home(request):
    return HttpResponse("<h1>🚀 Django + Docker funcionando correctamente</h1><p>CI/CD activo y en ejecución ✅</p>")

urlpatterns = [
    path('', home),  # Página principal agregada
    path("admin/", admin.site.urls),
    path("polls/", include("polls.urls")),
]
```

✅ Permite visualizar una página principal funcional en `http://localhost`

---

## 🧪 5️⃣ Ejecución de pruebas

Se corrieron los tests dentro del contenedor:
```bash
docker run --rm django-docker-template:master ./pytest.sh
```

**Resultado:**
```
polls/tests.py .......... [100%]
================== 10 passed in 0.19s ==================
```

---

## 🧱 6️⃣ Levantar entorno con Docker Compose

```bash
docker compose -f docker-compose.debug.yml up
```

**Logs esperados:**
```
django-1 | [INFO] Starting gunicorn 22.0.0
django-1 | [INFO] Listening at: http://0.0.0.0:8000
traefik  | Configuration loaded from flags.
```

Verificación:
```bash
docker ps
```
```
CONTAINER ID   IMAGE               PORTS
dd7bd449b2a5   django-docker       8000/tcp
aa8715605091   nginx:1.23-alpine   80/tcp
3e5ca197ba2f   postgres:15         5432/tcp
8e085c45f96f   traefik:v2.9        0.0.0.0:80->80/tcp
```

✅ Todos los servicios corriendo correctamente.

---

## 🌍 7️⃣ Validación en navegador

URL: `http://localhost`

Salida esperada:
> 🚀 Django + Docker funcionando correctamente  
> CI/CD activo y en ejecución ✅

---

## 📦 8️⃣ Simulación de despliegue continuo

Simulación en entorno externo:
```bash
git clone https://github.com/<tu_usuario>/<tu_repo>.git
cd <tu_repo>
docker compose up -d
```
Acceder a `http://localhost` → Sistema funcionando.

---

## 📘 9️⃣ Archivos entregables

| Archivo | Descripción |
|----------|--------------|
| `.github/workflows/ci.yml` | Flujo CI ejecutando tests en GitHub |
| `Dockerfile` | Imagen base Django |
| `docker-compose.yml` | Contenedores y servicios |
| `website/urls.py` | Vista raíz personalizada |
| `deploy-notes.md` | Pasos del despliegue simulado |
| `Flujo_CI_CD_para_Django.pdf` | Documento técnico final |
| `evidencias/django_funcionando.png` | Captura del navegador con Django funcionando |

---

## 🧠 10️⃣ Conclusión

El pipeline **CI/CD con Django, Docker y GitHub Actions** fue implementado correctamente e incluye:

- Integración Continua (CI) con tests automáticos
- Construcción de imágenes Docker
- Despliegue simulado con Docker Compose y Traefik
- Página de validación funcional

🎯 **Resultado final:** Django + Docker funcionando correctamente con CI/CD automatizado ✅
