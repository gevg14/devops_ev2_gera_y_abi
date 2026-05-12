# Innovatech Chile – DevOps Setup (EP2)

## Estructura del repositorio

```
/
├── back-Despachos_SpringBoot/
│   ├── Dockerfile                  ← Multi-stage build (Maven → JRE Alpine)
│   └── Springboot-API-REST-DESPACHO/
├── back-Ventas_SpringBoot/
│   ├── Dockerfile                  ← Multi-stage build (Maven → JRE Alpine)
│   └── Springboot-API-REST/
├── front_despacho/
│   ├── Dockerfile                  ← Multi-stage build (Node → Nginx Alpine)
│   └── nginx.conf                  ← SPA routing + proxy a backends
├── docker-compose.yml              ← Orquestación completa
├── .env.example                    ← Variables de entorno (template)
└── .github/
    └── workflows/
        └── cicd.yml                ← Pipeline CI/CD (rama: deploy)
```

## Secrets requeridos en GitHub

Ve a **Settings → Secrets and variables → Actions** y agrega:

| Secret | Descripción |
|--------|-------------|
| `DOCKERHUB_USERNAME` | Tu usuario de Docker Hub |
| `DOCKERHUB_TOKEN` | Token de acceso Docker Hub (no tu password) |
| `EC2_HOST` | IP pública o DNS de tu instancia EC2 |
| `EC2_USER` | Usuario SSH (ej: `ec2-user` o `ubuntu`) |
| `EC2_SSH_KEY` | Contenido completo de tu archivo `.pem` |
| `DB_DESPACHOS_NAME` | Nombre de la BD de despachos |
| `DB_DESPACHOS_USER` | Usuario de la BD de despachos |
| `DB_DESPACHOS_PASSWORD` | Password de la BD de despachos |
| `DB_DESPACHOS_ROOT_PASSWORD` | Password root MySQL despachos |
| `DB_VENTAS_NAME` | Nombre de la BD de ventas |
| `DB_VENTAS_USER` | Usuario de la BD de ventas |
| `DB_VENTAS_PASSWORD` | Password de la BD de ventas |
| `DB_VENTAS_ROOT_PASSWORD` | Password root MySQL ventas |

## Cómo generar el token de Docker Hub

1. Login en hub.docker.com → Account Settings → Security
2. New Access Token → nombre: `github-actions-innovatech`
3. Copiar el token y pegarlo como secret `DOCKERHUB_TOKEN`

## Cómo agregar la llave SSH de EC2

```bash
# El contenido del archivo .pem va completo, incluyendo las cabeceras:
# -----BEGIN RSA PRIVATE KEY-----
# ...
# -----END RSA PRIVATE KEY-----
cat tu-llave.pem   # copiar todo este output al secret EC2_SSH_KEY
```

## Activar el pipeline

```bash
# El pipeline se dispara con cualquier push a la rama deploy
git checkout -b deploy     # si no existe la rama
git add .
git commit -m "feat: agregar configuración DevOps EP2"
git push origin deploy
```

## Verificar despliegue en EC2

```bash
# Conectarse a EC2
ssh -i tu-llave.pem ec2-user@<IP_PUBLICA>

# Ver estado de los contenedores
docker compose ps

# Ver logs de un servicio
docker compose logs back-despachos
docker compose logs back-ventas
docker compose logs front-despacho

# Verificar redes
docker network ls
docker network inspect innovatech-privada
docker network inspect innovatech-publica
```

## Arquitectura de red

```
Internet
    │
    ▼ (puerto 80)
┌─────────────────┐
│  front-despacho │  ← red-publica + red-privada
│   (Nginx)       │
└────────┬────────┘
         │ red-privada (interna, sin acceso desde Internet)
    ┌────┴──────────────────┐
    ▼                       ▼
┌──────────────┐    ┌──────────────┐
│back-despachos│    │ back-ventas  │
│  :8081       │    │   :8080      │
└──────┬───────┘    └──────┬───────┘
       ▼                   ▼
┌──────────────┐    ┌──────────────┐
│ db-despachos │    │  db-ventas   │
│  (MySQL)     │    │  (MySQL)     │
│  vol. Docker │    │  vol. Docker │
└──────────────┘    └──────────────┘
```
