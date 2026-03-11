# Semana 21: CI/CD Pipeline (GitHub Actions)

## 🎯 Objetivo
Dejar de subir código a los servidores copiándolo manualmente o usando FTP. Automatizar la construcción, testing y subida del contenedor cuando alguien apruebe un Pull Request en GitHub.

## 🛠️ Stack Tecnológico
- GitHub Actions (`.yml`)
- Docker Hub/GHCR (Registries)
- Vercel/Render/VPS para Deploy automático

## 📋 Requisitos del Proyecto
1. Crear una carpeta oculta `.github/workflows` en tu repositorio.
2. Hacer un Job (`ci.yml`) que corra npm run lint y las Pruebas (Test Semana 12) *solamente* cuando se haga un Push al `main`.
3. Si y solo si los tests pasan (verde), proceder a construir la Imagen de Docker y subirla a tu cuenta de Docker Hub (usando GH Secrets de `DOCKER_USER`/`DOCKER_PASS`).
4. (Opcional Prod) Hacer que el servidor haga un Pull automático de esa nueva imagen mediante un "Webhook" (ej. Watchtower) o conexión SSH.

## ✅ Entregables
- Un flujo de Integración Continua validando código sin interactuar tú mismo localmente.
