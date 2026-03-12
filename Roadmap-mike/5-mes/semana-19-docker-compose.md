# 🐳 Semana 19 — Contenerización con Docker & Docker Compose

> **Mes 5 · DevOps, Docker & CI/CD** | Stack: `Docker` · `Docker Compose` · `Dockerfile` · `Multi-stage builds`

---

## 🎯 Objetivo del Proyecto

Contenerizar completamente una aplicación Full Stack usando **Docker**. Crear `Dockerfile`s optimizados para Node.js y React, y orquestar todos los servicios (API, Frontend, Base de Datos) con un único archivo **`docker-compose.yml`**.

---

## 🛠️ Stack Tecnológico

| Herramienta | Versión | Rol |
|---|---|---|
| Docker Engine | 24+ | Runtime de contenedores |
| Docker Compose | v2 | Orquestación multi-contenedor en dev local |
| Docker Hub | — | Registry de imágenes |
| Alpine Linux | — | Imagen base minimalista para producción |

---

## 📐 Arquitectura de Contenedores

```
docker-compose.yml
├── 🐳 api         (Node.js + Express)  → Puerto 3000
├── 🌐 frontend    (React + Nginx)      → Puerto 80
├── 🐘 postgres    (PostgreSQL 15)      → Puerto 5432
└── 🍃 mongo       (MongoDB 7)          → Puerto 27017
     └── 🗂 volumes/
         ├── postgres_data:/var/lib/postgresql/data
         └── mongo_data:/data/db
```

---

## 📋 Requisitos del Proyecto

### 1. Dockerfile para Node.js (Multi-stage Build)

```dockerfile
# backend/Dockerfile

# ─── ETAPA 1: Build ───────────────────────────────
FROM node:20-alpine AS builder
WORKDIR /app

# Copiar solo archivos de dependencias primero (optimiza caché de Docker)
COPY package*.json ./
RUN npm ci

# Copiar el resto del código y compilar TypeScript
COPY . .
RUN npm run build


# ─── ETAPA 2: Producción ──────────────────────────
FROM node:20-alpine AS production
WORKDIR /app

ENV NODE_ENV=production

# Copiar solo las dependencias de producción
COPY package*.json ./
RUN npm ci --production && npm cache clean --force

# Copiar solo el output compilado de la etapa anterior
COPY --from=builder /app/dist ./dist

# Usuario no root por seguridad
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

EXPOSE 3000
CMD ["node", "dist/app.js"]
```

### 2. Dockerfile para React (con Nginx)

```dockerfile
# frontend/Dockerfile

# ─── ETAPA 1: Build React ─────────────────────────
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# ─── ETAPA 2: Servir con Nginx ────────────────────
FROM nginx:alpine AS production
# Copiar el build de React al directorio de Nginx
COPY --from=builder /app/dist /usr/share/nginx/html
# Configuración de Nginx para SPA (React Router)
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

```nginx
# frontend/nginx.conf
server {
    listen 80;
    root /usr/share/nginx/html;
    index index.html;

    # Para React Router: redirigir todas las rutas al index.html
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache de assets estáticos
    location /assets/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### 3. Docker Compose completo

```yaml
# docker-compose.yml
version: '3.9'

services:
  # ── Base de Datos PostgreSQL ──────────────────────
  postgres:
    image: postgres:15-alpine
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${POSTGRES_DB:-roadmap_db}
      POSTGRES_USER: ${POSTGRES_USER:-admin}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-supersecret}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-admin}"]
      interval: 10s
      retries: 5

  # ── API Node.js ───────────────────────────────────
  api:
    build:
      context: ./backend
      target: production
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}
      JWT_SECRET: ${JWT_SECRET}
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - app-network

  # ── Frontend React ────────────────────────────────
  frontend:
    build:
      context: ./frontend
      target: production
    restart: unless-stopped
    ports:
      - "80:80"
    depends_on:
      - api
    networks:
      - app-network

volumes:
  postgres_data:
    driver: local

networks:
  app-network:
    driver: bridge
```

### 4. Archivo `.env` para Compose

```env
# .env (nunca subir a GitHub)
POSTGRES_DB=roadmap_db
POSTGRES_USER=admin
POSTGRES_PASSWORD=supersecret_password
JWT_SECRET=tu_secreto_jwt_muy_largo_aqui
```

### 5. Comandos esenciales de Docker

```bash
# Construir y levantar todos los servicios
docker compose up --build -d

# Ver logs en tiempo real
docker compose logs -f api

# Ejecutar comandos dentro de un contenedor
docker compose exec api sh
docker compose exec postgres psql -U admin -d roadmap_db

# Ver estado de los contenedores
docker compose ps

# Detener y limpiar
docker compose down
docker compose down -v  # También elimina los volúmenes

# Ver imágenes locales
docker images

# Subir imagen a Docker Hub
docker tag mi-api:latest usuario/mi-api:v1.0
docker push usuario/mi-api:v1.0
```

---

## 🧠 Conceptos Clave Aprendidos

| Concepto | Descripción |
|---|---|
| **Multi-stage Build** | Separar la etapa de compilación de la de producción. Resultado: imagen ~3x más pequeña |
| **Volúmenes** | Datos persistentes fuera del ciclo de vida del contenedor |
| **Networks** | Comunicación interna entre contenedores por nombre de servicio (ej: `postgres:5432`) |
| **Healthcheck** | Docker espera a que el servicio esté listo antes de iniciar los dependientes |
| **`.env` + `${VAR}`** | Variables de entorno en Compose para separar config de código |
| **`depends_on + condition`** | Garantiza el orden de inicio de servicios |

---

## ✅ Entregables

- [ ] Dockerfile multi-stage para el backend (imagen < 200MB).
- [ ] Dockerfile multi-stage para el frontend con Nginx.
- [ ] `docker-compose.yml` que levanta toda la app con `docker compose up`.
- [ ] Variables de entorno en `.env` (nunca en el `compose` directamente).
- [ ] App accesible en `http://localhost` tras `docker compose up --build`.

---

*← [Semana 18](../5-mes/semana-18-linux-bash.md) | [Volver al Roadmap](../../README.md) | [Semana 20 →](../5-mes/semana-20-microservices.md)*
