# Semana 02: Autenticación y Seguridad (JWT)

## 🎯 Objetivo
Aprender a asegurar endpoints protegiendo las rutas de una API REST mediante el manejo de contraseñas seguras y tokens de acceso.

## 🛠️ Stack Tecnológico
- Node.js & Express
- JSON Web Tokens (JWT)
- bcrypt (Hashing)

## 📋 Requisitos del Proyecto
1. Tomar la API de la semana pasada y añadir un flujo de registro (`/register`) y login (`/login`).
2. Al registrar, encriptar la contraseña del usuario utilizando la librería `bcrypt`.
3. Al loguear correctamente, devolver un JWT firmado con una clave secreta.
4. Crear un Middleware `authMiddleware` que verifique la validez del JWT en las cabeceras (`Authorization: Bearer <token>`).
5. Proteger las rutas CRUD de la semana 1 para que solo usuarios autenticados puedan acceder.

## ✅ Entregables
- Código actualizado en GitHub.
- Pruebas en Postman enviando el Token en las cabeceras.
