# Innovatech Chile — Frontend

Aplicación web Flask para el sistema de gestión de usuarios de Innovatech Chile.
Se comunica con el backend API para mostrar y gestionar usuarios.
Desplegada en AWS EC2 con Docker y CI/CD automatizado via GitHub Actions.

## Stack tecnológico

- **Runtime:** Python 3.11
- **Framework:** Flask 2.3 + Gunicorn
- **Servidor web:** Gunicorn (producción)
- **Contenedor:** Docker (multi-stage build)
- **CI/CD:** GitHub Actions + Docker Hub
- **Infraestructura:** AWS EC2 en VPC

## Estructura del repositorio

```
.
├── app.py                  # Punto de entrada — aplicación Flask
├── requirements.txt        # Dependencias Python
├── templates/              # Plantillas HTML
│   ├── base.html           # Plantilla base
│   ├── index.html          # Lista de usuarios
│   ├── crear_usuario.html  # Formulario de creación
│   ├── editar_usuario.html # Formulario de edición
│   ├── 404.html            # Página de error 404
│   └── 500.html            # Página de error 500
├── Dockerfile              # Multi-stage build (builder + runner)
├── docker-compose.yml      # Stack del servicio frontend
├── .env.example            # Variables de entorno de referencia
├── .gitignore              # Excluye .env y llaves
└── .github/
    └── workflows/
        └── deploy.yml      # Pipeline CI/CD
```

## Cómo ejecutar localmente

```bash
git clone https://github.com/hyyjuan/Front_Eval2
cd Front_Eval2
cp .env.example .env
# Editar .env con la IP del backend
docker compose up -d
docker compose ps
# Abrir http://localhost:5000
```

## Pipeline CI/CD

El pipeline se activa automáticamente al hacer push a la rama `deploy`.

**Flujo:** push a deploy → build imagen Docker → push a Docker Hub → deploy en EC2 vía SSH

## Variables de entorno

| Variable | Descripción | Default |
|----------|-------------|---------|
| PORT | Puerto del servidor Flask | 5000 |
| DEBUG | Modo debug | False |
| SECRET_KEY | Clave secreta para sesiones | — |
| BACKEND_URL | URL del backend API | http://localhost:3000 |

## GitHub Secrets requeridos

- `DOCKERHUB_USERNAME` — usuario de Docker Hub
- `DOCKERHUB_TOKEN` — token de acceso Docker Hub
- `EC2_HOST` — IP pública de ec2-front
- `EC2_USER` — ec2-user
- `EC2_SSH_KEY` — contenido del archivo .pem

## Decisiones técnicas

**Multi-stage build:** la etapa builder instala las dependencias Python en una carpeta local, la etapa runner copia solo lo necesario. Esto evita incluir pip y herramientas de build en la imagen final.

**Gunicorn sobre Flask dev server:** Gunicorn es un servidor WSGI de producción más estable y eficiente que el servidor de desarrollo de Flask, soporta múltiples workers concurrentes.

**Usuario no root:** el contenedor corre con un usuario sin privilegios aplicando el principio de mínimo privilegio.

**Docker Hub sobre ECR:** AWS Academy genera credenciales temporales que rotan cada pocas horas, lo que hace inviable ECR en pipelines automáticos. Docker Hub usa credenciales permanentes almacenadas como GitHub Secrets.
