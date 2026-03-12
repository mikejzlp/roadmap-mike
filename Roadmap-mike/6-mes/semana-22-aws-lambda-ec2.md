# ☁️ Semana 22 — Infraestructura AWS (EC2, S3, Lambda & RDS)

> **Mes 6 · Cloud & Arquitectura Avanzada** | Stack: `AWS EC2` · `AWS S3` · `AWS Lambda` · `RDS` · `IAM` · `AWS CLI`

---

## 🎯 Objetivo del Proyecto

Desplegar una aplicación completa en **Amazon Web Services (AWS)** usando los servicios más usados en la industria: EC2 para servidores, S3 para almacenamiento de archivos, Lambda para funciones serverless y RDS para bases de datos gestionadas.

---

## 🛠️ Stack Tecnológico

| Servicio AWS | Rol |
|---|---|
| **EC2** (Elastic Compute Cloud) | Servidor virtual para la API Node.js |
| **S3** (Simple Storage Service) | Almacenamiento de archivos e imágenes |
| **RDS** (Relational Database Service) | PostgreSQL gestionado en AWS |
| **Lambda + API Gateway** | Función serverless para procesamiento de imágenes |
| **IAM** (Identity Access Management) | Permisos y roles para los servicios |
| **CloudWatch** | Monitoreo de logs y métricas |
| **Route 53** | DNS para el dominio personalizado |

---

## 📐 Arquitectura AWS

```
Internet
    │
    ▼
Route 53 (DNS)
    │
    ▼
Application Load Balancer
    │
    ├──► EC2 Instance (t3.micro) — API Node.js + Nginx + PM2
    │         │
    │         └──► RDS PostgreSQL (db.t3.micro) — Private Subnet
    │
    ├──► S3 Bucket ──► CloudFront CDN (Frontend React Build)
    │
    └──► API Gateway ──► Lambda Function (serverless tasks)
```

---

## 📋 Requisitos del Proyecto

### 1. Configurar AWS CLI

```bash
# Instalar AWS CLI
pip install awscli

# Configurar credenciales
aws configure
# AWS Access Key ID: (desde IAM → Users → Security credentials)
# AWS Secret Access Key: (nunca compartir)
# Default region: us-east-1
# Default output format: json

# Verificar configuración
aws sts get-caller-identity
```

### 2. Lanzar una instancia EC2

```bash
# Crear key pair para SSH
aws ec2 create-key-pair \
  --key-name mi-key-pair \
  --query 'KeyMaterial' \
  --output text > mi-key-pair.pem

chmod 400 mi-key-pair.pem

# Lanzar instancia EC2 (Ubuntu 22.04 t3.micro en Free Tier)
aws ec2 run-instances \
  --image-id ami-0c7217cdde317cfec \  # Ubuntu 22.04 (us-east-1)
  --instance-type t3.micro \
  --key-name mi-key-pair \
  --security-group-ids sg-xxxxxxxx \
  --count 1 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=api-server}]'

# Conectarse via SSH
ssh -i mi-key-pair.pem ubuntu@<IP_PUBLICA>
```

### 3. Usar S3 para almacenamiento de archivos

```bash
# Crear bucket
aws s3 mb s3://mi-app-archivos-2024 --region us-east-1

# Subir imagen desde la CLI
aws s3 cp imagen.jpg s3://mi-app-archivos-2024/uploads/imagen.jpg

# Listar archivos
aws s3 ls s3://mi-app-archivos-2024/uploads/
```

```typescript
// backend/src/services/s3.service.ts — Subir archivos desde Node.js
import { S3Client, PutObjectCommand, GetObjectCommand } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';

const s3 = new S3Client({
  region: process.env.AWS_REGION!,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID!,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY!,
  },
});

export const uploadToS3 = async (file: Buffer, key: string, contentType: string) => {
  await s3.send(new PutObjectCommand({
    Bucket: process.env.S3_BUCKET_NAME!,
    Key: key,
    Body: file,
    ContentType: contentType,
  }));
  return `https://${process.env.S3_BUCKET_NAME}.s3.amazonaws.com/${key}`;
};

// Generar URL pre-firmada para subida directa desde el cliente (evita pasar por el servidor)
export const getPresignedUploadUrl = async (key: string) => {
  const command = new PutObjectCommand({
    Bucket: process.env.S3_BUCKET_NAME!,
    Key: key,
  });
  return getSignedUrl(s3, command, { expiresIn: 3600 }); // URL válida por 1 hora
};
```

### 4. Lambda Function (Serverless)

```typescript
// lambda/resize-image/index.ts
import { S3Client, GetObjectCommand, PutObjectCommand } from '@aws-sdk/client-s3';
import sharp from 'sharp'; // Para redimensionar imágenes

const s3 = new S3Client({});

// Esta función se ejecuta automáticamente cuando se sube una imagen a S3
export const handler = async (event: any) => {
  const record = event.Records[0];
  const bucket = record.s3.bucket.name;
  const key = decodeURIComponent(record.s3.object.key);

  // Descargar imagen original
  const { Body } = await s3.send(new GetObjectCommand({ Bucket: bucket, Key: key }));
  const imageBuffer = Buffer.from(await Body!.transformToByteArray());

  // Redimensionar a thumbnail (200x200)
  const thumbnail = await sharp(imageBuffer).resize(200, 200, { fit: 'cover' }).toBuffer();

  // Subir thumbnail al bucket de thumbnails
  const thumbnailKey = `thumbnails/${key.split('/').pop()}`;
  await s3.send(new PutObjectCommand({
    Bucket: process.env.THUMBNAILS_BUCKET!,
    Key: thumbnailKey,
    Body: thumbnail,
    ContentType: 'image/jpeg',
  }));

  console.log(`✅ Thumbnail creado: ${thumbnailKey}`);
};
```

```bash
# Desplegar la Lambda
zip -r function.zip dist/ node_modules/
aws lambda create-function \
  --function-name resize-image \
  --runtime nodejs20.x \
  --handler index.handler \
  --zip-file fileb://function.zip \
  --role arn:aws:iam::ACCOUNT_ID:role/LambdaS3Role
```

---

## 🧠 Conceptos Clave Aprendidos

| Servicio | Modelo de Precios | Cuándo Usar |
|---|---|---|
| **EC2** | Pago por hora | Apps que necesitan servidor persistente |
| **S3** | Pago por GB almacenado + transferencia | Archivos estáticos, backups, imágenes |
| **RDS** | Pago por hora + almacenamiento | BD SQL gestionada (parches, backups automáticos) |
| **Lambda** | Pago por ejecución (~$0.00001667/GB-s) | Procesamiento por eventos, tareas cortas |

---

## ✅ Entregables

- [ ] EC2 con Ubuntu, Node.js, Nginx y PM2 corriendo la API.
- [ ] S3 con bucket configurado y carga de archivos desde Node.js.
- [ ] Lambda que redimensiona imágenes automáticamente al subirlas a S3.
- [ ] RDS PostgreSQL privado accesible solo desde la EC2.
- [ ] IAM roles/policies con permisos mínimos necesarios.

---

*← [Semana 21](../5-mes/semana-21-github-actions.md) | [Volver al Roadmap](../../README.md) | [Semana 23 →](../6-mes/semana-23-kubernetes.md)*
