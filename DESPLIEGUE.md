# Despliegue Automático con GitHub Actions

Para esta práctica hemos creado un workflow en GitHub Actions que automatiza la construcción de la imagen Docker y su publicación en Docker Hub cada vez que se hace push a la rama `master`.

## Tabla de Contenidos

- [Introducción](#introducción)
- [Archivo del Workflow](#archivo-del-workflow)
- [Configuración de Secretos en GitHub](#configuración-de-secretos-en-github)
- [Cómo Funciona el Workflow](#cómo-funciona-el-workflow)
- [Uso de la Imagen desde Docker Hub](#uso-de-la-imagen-desde-docker-hub)
- [Docker Compose Actualizado](#docker-compose-actualizado)

---

## Introducción

En este proyecto, utilizamos Github Actions para:

1. **Construir** la imagen Docker usando el `Dockerfile` existente
2. **Autenticarse** en Docker Hub con credenciales (usando secretos de Docker en el repositorio)
3. **Publicar** la imagen en Docker Hub automáticamente

### Ventajas de este enfoque

- **Automatización**: No necesitamos construir la imagen manualmente
- **Consistencia**: La imagen siempre se construye en el mismo entorno
- **Disponibilidad**: La imagen estará disponible en Docker Hub para cualquier servidor
- **Seguridad**: Las credenciales se gestionan como secretos cifrados

---

A continuación, se detallan paso por paso los cambios agregados en esta práctica:

## 1. Archivo del Workflow

**Ubicación**: `.github/workflows/docker-publish.yml`

https://github.com/nolocardeno/2526_DAW_u2_springboot/blob/aa169b0a97b3b9e0836c5a36d611b259d167323d/.github/workflows/docker-publish.yml#L1-L26


```yaml
name: Build and Push Docker Image

on:
  push:
    branches:
      - master

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Log in to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build Docker image
        run: docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/springboot-crud-app:latest .

      - name: Push Docker image
        run: docker push ${{ secrets.DOCKERHUB_USERNAME }}/springboot-crud-app:latest
```

### Explicación del archivo

| Sección | Descripción |
|---------|-------------|
| `name` | Nombre descriptivo del workflow |
| `on.push.branches` | Se ejecuta cuando hay un push a la rama `master` |
| `runs-on` | Usa una máquina virtual Ubuntu en los servidores de GitHub |
| `actions/checkout@v4` | Descarga el código del repositorio |
| `docker/login-action@v3` | Inicia sesión en Docker Hub con las credenciales |
| `docker build` | Construye la imagen usando el Dockerfile |
| `docker push` | Sube la imagen a Docker Hub |

---

## Configuración de Secretos en GitHub

Para que el workflow funcione, necesitamos configurar dos secretos en tu repositorio de GitHub:

| Nombre del Secreto | Valor |
|--------------------|-------|
| `DOCKERHUB_USERNAME` | Mi nombre de usuario de Docker Hub |
| `DOCKER_PASSWORD` | El token de acceso o contraseña |

---


## Uso de la Imagen desde Docker Hub

Una vez que el workflow se ejecuta correctamente, la imagen está disponible públicamente en Docker Hub.

### Descargar y ejecutar la imagen

```bash
# Descargar la imagen
docker pull nolorubio23/springboot-crud-app:latest

# Ejecutar el contenedor
docker run -d -p 8080:8080 nolorubio23/springboot-crud-app:latest

# Ejecutar con volumen para persistencia
docker run -d -p 8080:8080 -v $(pwd)/data:/app/data nolorubio23/springboot-crud-app:latest
```

### Verificar la imagen en Docker Hub

Puedes ver la imagen publicada en:
```
https://hub.docker.com/r/nolorubio23/springboot-crud-app
```

---

## 🐳 Docker Compose Actualizado

El archivo `docker-compose.yml` ha sido actualizado para usar la imagen de Docker Hub en lugar de construirla localmente:

```yaml
version: '3.8'

services:
  springboot-crud-app:
    container_name: springboot-users-crud
    
    # Usar imagen de DockerHub (ya no se construye localmente)
    image: nolorubio23/springboot-crud-app:latest
    
    ports:
      - "8080:8080"
    
    environment:
      - APP_DATA_DIR=/app/data
      - APP_DATA_FILE=/app/data/users.json
    
    volumes:
      - ./data:/app/data
    
    restart: unless-stopped
    
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### Cambios realizados

| Antes | Después |
|-------|---------|
| `build: context: .` | ❌ Eliminado |
| `image: springboot-crud-app:latest` | `image: nolorubio23/springboot-crud-app:latest` |

### Comandos para usar Docker Compose

```bash
# Descargar la imagen y levantar el contenedor
docker-compose up -d

# Ver logs
docker-compose logs -f

# Detener
docker-compose down

# Actualizar a la última versión de la imagen
docker-compose pull
docker-compose up -d
```

---

## 📋 Resumen

| Componente | Descripción |
|------------|-------------|
| **Workflow** | `.github/workflows/docker-publish.yml` |
| **Trigger** | Push a la rama `master` |
| **Imagen** | `nolorubio23/springboot-crud-app:latest` |
| **Secretos** | `DOCKERHUB_USERNAME`, `DOCKER_PASSWORD` |

### Flujo completo

1. Haces cambios en el código
2. Ejecutas `git push` a `master`
3. GitHub Actions construye y sube la imagen automáticamente
4. Cualquier servidor puede descargar la imagen con `docker pull`
5. Ejecutas `docker-compose up -d` para desplegar

---

**¡Despliegue automatizado completado! 🎉**