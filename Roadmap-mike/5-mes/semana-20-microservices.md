# Semana 20: Arquitectura de Microservicios Básicos

## 🎯 Objetivo
Hacer que varios contenedores (Dockers) se comuniquen internamente en una red privada y resolver DNS y Puertos exponiendo un Proxy inverso al mundo externo.

## 🛠️ Stack Tecnológico
- Nginx (Reverse Proxy)
- Docker Compose Network
- Node.js (Múltiples APIs miniatura)

## 📋 Requisitos del Proyecto
1. Levantar 2 Backends simples ("Servicio A" y "Servicio B") en dos puertos distintos (3001, 3002).
2. Crear un contenedor con la imagen oficial de Nginx (`nginx:alpine`).
3. Escribir un bloque `nginx.conf` donde si el usuario manda `/api/a`, Nginx rutea hacia el Backend 1 (pasando de largo el puerto), y si pone `/api/b`, al Backend 2.
4. No exponer los puertos de los servicios en Node hacia afuera (tu host local). Solamente el puerto `80` de Nginx.
5. Poner todos los contenedores en la misma red `backend-net` predefinida en compose.

## ✅ Entregables
- Arquitectura micro-servicial simulada corriendo bajo un solo servidor web.
