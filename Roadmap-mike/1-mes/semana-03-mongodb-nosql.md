# Semana 03: Bases de Datos NoSQL con MongoDB

## 🎯 Objetivo
Conectar la API REST a una base de datos real (MongoDB) para que la información persista, aprendiendo a modelar los datos.

## 🛠️ Stack Tecnológico
- MongoDB (Atlas)
- Mongoose (ODM)
- Node.js

## 📋 Requisitos del Proyecto
1. Crear un clúster gratuito en MongoDB Atlas y obtener la URI de conexión.
2. Definir un Modelo (Schema) de `Usuario` y un nuevo recurso (por ejemplo, `Post` o `Tarea`) usando Mongoose.
3. Reemplazar la lógica de array en memoria de las semanas pasadas por consultas reales a la BD (`.find()`, `.save()`, `.findByIdAndUpdate()`).
4. Entender y aplicar relaciones "referenciadas" en MongoDB (un Post pertenece a un Usuario).

## ✅ Entregables
- Bases de datos conectadas correctamente.
- Uso de variables de entorno (`.env`) para ocultar las credenciales de la BD.
