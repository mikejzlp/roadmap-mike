# Semana 04: Bases de Datos Relacionales (PostgreSQL)

## 🎯 Objetivo
Dar el salto al mundo de las bases de datos relacionales conectando Node con PostgreSQL y gestionándolas con un ORM moderno.

## 🛠️ Stack Tecnológico
- PostgreSQL
- Prisma ORM
- Node.js

## 📋 Requisitos del Proyecto
1. Crear una base de datos PostgreSQL (local o en la nube como Supabase/Neon).
2. Inicializar Prisma (`npx prisma init`) y crear el esquema `schema.prisma`.
3. Modelar relaciones relacionales estrictas (1 a muchos, muchos a muchos). Por ejemplo: Usuarios, Productos, y Pedidos.
4. Ejecutar migraciones (`prisma migrate dev`) y poblar la base de datos.
5. Crear un controlador Express que exponga los datos y maneje transacciones.

## ✅ Entregables
- Modelo de base de datos relacional claro.
- Consultas eficientes realizadas a través del cliente de Prisma.
