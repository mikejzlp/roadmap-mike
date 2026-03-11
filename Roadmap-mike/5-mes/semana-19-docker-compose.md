# Semana 19: Contenerización y Aislamiento (Docker)

## 🎯 Objetivo
Resolver para siempre el problema de: "En mi máquina sí funciona". Aprender a empaquetar tu código y sus dependencias en contenedores ligeros portables.

## 🛠️ Stack Tecnológico
- Docker Engine
- Docker Compose
- Node.js (Alpine Images)

## 📋 Requisitos del Proyecto
1. Tomar tu API RESTful gigante del mes 1/3 y empaquetarla escribiendo un excelente `Dockerfile` multi-etapa (Multi-stage build).
2. Optimizar el contenedor basándolo en distribuciones pequeñas como `node:alpine` para no pesar gigabytes inútiles.
3. Usar `docker-compose.yml` para levantar TU Backend NodeJS Y TU Base de datos (MongoDB o PostgreSQL) con un solo comando `docker-compose up -d`.
4. Mapear los volúmenes para hacer persistente la BD y los puertos ("bindings").

## ✅ Entregables
- Contenedores Docker eficientes (< 300MB de peso final si es Node), corriendo nativamente.
