# Semana 22: Infraestructura y Escalado Cloud (AWS)

## 🎯 Objetivo
Comprender los cimientos básicos del proveedor de nube más utilizado del mundo (Amazon Web Services), abandonando poco a poco tu PC local por la nube pública.

## 🛠️ Stack Tecnológico
- AWS EC2 (Máquinas Virtuales)
- AWS S3 (Buckets de almacenamiento)
- AWS IAM (Roles y Permisos)
- AWS RDS (Relational Database Service) o Funciones Serverless (AWS Lambda)

## 📋 Requisitos del Proyecto
1. Desplegar el E-Commerce completo (MERN o PERN) en una instancia **AWS EC2 (tier gratuito)** apuntando a **AWS RDS** como Base de datos PostgreSQL alojada y gestionada por Amazon.
2. Modificar el Backend Node para cargar imágenes/documentos estáticos de productos utilizando un **AWS S3 Bucket** y el SDK de `aws-sdk` o `@aws-sdk/client-s3`.
3. Evitar a toda costa quemar credenciales en código público, aprender a crear un usuario seguro desde Amazon IAM que solo tenga acceso (Policy) estricto de lectura/escritura a un Bucket y proveer esas `.env`.

## ✅ Entregables
- El dashboard web sirviendo imágenes de forma estática en CDN desde S3 Bucket real corriendo 100% hospedado sobre una red EC2 con una IP Pública dedicada.
