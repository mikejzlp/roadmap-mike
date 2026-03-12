# ⚙️ Semana 21 — GitHub Actions & CI/CD Pipelines

> **Mes 5 · DevOps, Docker & CI/CD** | Stack: `GitHub Actions` · `CI/CD` · `Docker Hub` · `Jest` · `Deploy Automático`

---

## 🎯 Objetivo del Proyecto

Implementar un **pipeline de CI/CD completo** con GitHub Actions que automatice: ejecutar tests, construir imágenes Docker y desplegar la aplicación a producción en cada push a la rama `main`. Aplicar el principio de **"si no está en CI, no existe"**.

---

## 🛠️ Stack Tecnológico

| Herramienta | Rol |
|---|---|
| GitHub Actions | Plataforma de CI/CD integrada con GitHub |
| Docker Hub | Registry de imágenes Docker |
| Jest + ESLint | Pruebas y calidad de código en CI |
| SSH Deploy | Despliegue al servidor via SSH desde el pipeline |
| Secrets de GitHub | Variables sensibles para el pipeline |

---

## 📐 Pipeline Completo

```
Push a main
     │
     ▼
┌─── CI Stage ────────────────────────┐
│  1. Checkout código                  │
│  2. Setup Node.js 20                 │
│  3. npm ci (install reproducible)    │
│  4. ESLint (calidad de código)       │
│  5. TypeScript Check (tsc --noEmit)  │
│  6. Jest (tests unitarios e integr.) │
└──────────────────────────────────────┘
     │ (solo si todos los pasos pasan)
     ▼
┌─── CD Stage ────────────────────────┐
│  7. Build Docker image               │
│  8. Push a Docker Hub                │
│  9. SSH al servidor de producción    │
│ 10. docker compose pull && up -d     │
└──────────────────────────────────────┘
```

---

## 📋 Requisitos del Proyecto

### 1. Configurar Secrets en GitHub

En `Settings → Secrets → Actions`:

| Secret | Descripción |
|---|---|
| `DOCKERHUB_USERNAME` | Tu usuario de Docker Hub |
| `DOCKERHUB_TOKEN` | Token de acceso de Docker Hub |
| `PROD_SSH_HOST` | IP del servidor de producción |
| `PROD_SSH_USER` | Usuario SSH del servidor |
| `PROD_SSH_KEY` | Clave privada SSH (contenido completo) |
| `JWT_SECRET` | Secret de la app para el `.env` en producción |

### 2. Workflow de CI (`.github/workflows/ci.yml`)

```yaml
# .github/workflows/ci.yml
name: CI — Tests & Quality

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    name: 🧪 Tests & Quality Check
    runs-on: ubuntu-latest

    services:
      # Levantar PostgreSQL para los tests de integración
      postgres:
        image: postgres:15-alpine
        env:
          POSTGRES_DB: test_db
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - name: 📥 Checkout código
        uses: actions/checkout@v4

      - name: 🟢 Setup Node.js 20
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: 📦 Instalar dependencias
        run: npm ci

      - name: 🔍 TypeScript Check
        run: npx tsc --noEmit

      - name: 📏 ESLint
        run: npm run lint

      - name: 🧪 Ejecutar Tests
        run: npm test -- --coverage --ci
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test_db
          JWT_SECRET: test-secret-ci

      - name: 📊 Upload Coverage Report
        uses: codecov/codecov-action@v4
        with:
          file: ./coverage/lcov.info
```

### 3. Workflow de CD (`.github/workflows/cd.yml`)

```yaml
# .github/workflows/cd.yml
name: CD — Build, Push & Deploy

on:
  workflow_run:
    workflows: ["CI — Tests & Quality"]
    types: [completed]
    branches: [main]

jobs:
  build-and-push:
    name: 🐳 Build & Push Docker Image
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' }}

    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}

    steps:
      - uses: actions/checkout@v4

      - name: 🔑 Login a Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: 🏷️ Generar metadata de la imagen
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ secrets.DOCKERHUB_USERNAME }}/mi-api
          tags: |
            type=sha,prefix=sha-
            type=raw,value=latest

      - name: 🔨 Build y Push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    name: 🚀 Deploy a Producción
    runs-on: ubuntu-latest
    needs: build-and-push

    steps:
      - name: 🔑 Configurar SSH
        uses: webfactory/ssh-agent@v0.9.0
        with:
          ssh-private-key: ${{ secrets.PROD_SSH_KEY }}

      - name: 🚀 Deploy via SSH
        run: |
          ssh -o StrictHostKeyChecking=no ${{ secrets.PROD_SSH_USER }}@${{ secrets.PROD_SSH_HOST }} << 'EOF'
            cd /var/www/mi-app
            
            # Actualizar el archivo .env con el nuevo tag de imagen
            echo "IMAGE_TAG=${{ needs.build-and-push.outputs.image-tag }}" >> .env
            
            # Descargar nueva imagen y reiniciar
            docker compose pull api
            docker compose up -d api
            
            # Limpiar imágenes antiguas
            docker image prune -f
            
            echo "✅ Deploy completado!"
          EOF
```

---

## 🧠 Conceptos Clave Aprendidos

| Concepto | Descripción |
|---|---|
| **CI (Continuous Integration)** | Integrar y probar el código en cada push para detectar errores temprano |
| **CD (Continuous Deployment)** | Desplegar automáticamente a producción si el CI pasa |
| **GitHub Secrets** | Variables seguras que no se exponen en los logs del runner |
| **Matrix Strategy** | Probar en múltiples versiones de Node/OS en paralelo |
| **workflow_run trigger** | Encadenar workflows (CD solo si CI pasa) |
| **Docker Build Cache** | `cache-from: type=gha` reutiliza capas de Docker entre runs para builds más rápidos |

---

## ✅ Entregables

- [ ] Workflow de CI que corre ESLint, TypeScript check y Jest en cada PR.
- [ ] Workflow de CD que construye y empuja la imagen a Docker Hub.
- [ ] Deploy automático al servidor de producción vía SSH.
- [ ] Todos los secrets configurados en GitHub (nunca hardcodeados).
- [ ] Badge de CI en el `README.md` del proyecto.

---

*← [Semana 20](../5-mes/semana-20-microservices.md) | [Volver al Roadmap](../../README.md) | [Semana 22 →](../6-mes/semana-22-aws-lambda-ec2.md)*
