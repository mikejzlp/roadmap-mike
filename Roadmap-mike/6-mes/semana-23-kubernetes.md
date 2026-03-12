# ⎈ Semana 23 — Kubernetes (K8s) 101

> **Mes 6 · Cloud & Arquitectura Avanzada** | Stack: `Kubernetes` · `kubectl` · `Minikube` · `Pods` · `Deployments` · `Services`

---

## 🎯 Objetivo del Proyecto

Aprender los conceptos fundamentales de **Kubernetes**, el orquestador de contenedores estándar de la industria. Desplegar una aplicación en un cluster local con **Minikube**, entendiendo los objetos base: Pods, Deployments, Services y ConfigMaps.

---

## 🛠️ Herramientas

| Herramienta | Rol |
|---|---|
| Minikube | Cluster de Kubernetes local para desarrollo |
| kubectl | CLI para interactuar con el cluster de K8s |
| Helm | Package manager para K8s (gestor de charts) |
| k9s | Terminal UI para monitorear el cluster |

---

## 📐 Objeto Fundamentales de Kubernetes

```
Cluster
├── Node (VM o máquina física)
│   └── Pod (1+ contenedores)
│       ├── Container (imagen Docker)
│       └── Container (sidecar)
│
├── Deployment (gestiona Pods con replicas)
├── Service (expone Pods con IP estable)
├── ConfigMap (configuración no sensible)
├── Secret (credenciales cifradas)
└── Ingress (entrada HTTP/HTTPS al cluster)
```

---

## 📋 Requisitos del Proyecto

### 1. Instalar y Configurar Minikube

```bash
# macOS
brew install minikube kubectl

# Windows (con Chocolatey)
choco install minikube kubernetes-cli

# Iniciar el cluster
minikube start --cpus=2 --memory=4096 --driver=docker

# Ver estado del cluster
kubectl cluster-info
kubectl get nodes

# Acceder al dashboard web de K8s
minikube dashboard
```

### 2. Deployment — Gestionar la app con réplicas

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
  labels:
    app: mi-api
spec:
  replicas: 3                     # 3 instancias del Pod
  selector:
    matchLabels:
      app: mi-api
  template:                       # Template del Pod
    metadata:
      labels:
        app: mi-api
    spec:
      containers:
        - name: api
          image: tu-usuario/mi-api:latest
          ports:
            - containerPort: 3000
          env:
            - name: NODE_ENV
              value: production
            - name: PORT
              value: "3000"
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: api-secrets
                  key: database-url
          resources:
            requests:
              cpu: "100m"         # 0.1 CPU core mínimo garantizado
              memory: "128Mi"     # 128 MB RAM mínimo
            limits:
              cpu: "500m"         # Máximo 0.5 CPU core
              memory: "512Mi"     # Máximo 512 MB RAM
          readinessProbe:         # K8s envía tráfico SOLO si el probe pasa
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 5
          livenessProbe:          # K8s reinicia el Pod si el probe falla
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 30
            periodSeconds: 10
```

### 3. Service — Exponer los Pods internamente

```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  selector:
    app: mi-api               # Selecciona Pods con este label
  ports:
    - protocol: TCP
      port: 80                # Puerto del Service
      targetPort: 3000        # Puerto del contenedor
  type: ClusterIP             # Solo accesible dentro del cluster
```

### 4. ConfigMap y Secret

```yaml
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: api-config
data:
  NODE_ENV: production
  LOG_LEVEL: info
  MAX_CONNECTIONS: "100"

---
# k8s/secret.yaml (valores en base64)
apiVersion: v1
kind: Secret
metadata:
  name: api-secrets
type: Opaque
data:
  # echo -n "valor" | base64
  database-url: cG9zdGdyZXNxbDovLy4uLg==
  jwt-secret: dHVfc2VjcmV0b19qd3Q=
```

### 5. Comandos esenciales de kubectl

```bash
# Aplicar configuración
kubectl apply -f k8s/

# Ver recursos
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get all

# Inspeccionar un Pod
kubectl describe pod <nombre-del-pod>

# Ver logs
kubectl logs <nombre-del-pod>
kubectl logs -f <nombre-del-pod>  # Follow (en tiempo real)

# Escalar la aplicación
kubectl scale deployment api-deployment --replicas=5

# Actualizar la imagen (rolling update sin downtime)
kubectl set image deployment/api-deployment api=tu-usuario/mi-api:v2.0

# Ver el estado del rolling update
kubectl rollout status deployment/api-deployment

# Revertir a la versión anterior
kubectl rollout undo deployment/api-deployment

# Ejecutar shell en un Pod
kubectl exec -it <nombre-del-pod> -- /bin/sh

# Exponer externamente en Minikube (dev)
minikube service api-service --url
```

### 6. Endpoint de Health Check en la API

```typescript
// src/routes/health.routes.ts
import { Router } from 'express';
import { prisma } from '../lib/prisma';

const router = Router();

router.get('/health', async (req, res) => {
  try {
    await prisma.$queryRaw`SELECT 1`;  // Verificar conexión a la BD
    res.json({ status: 'healthy', db: 'connected', timestamp: new Date() });
  } catch {
    res.status(503).json({ status: 'unhealthy', db: 'disconnected' });
  }
});

export default router;
```

---

## 🧠 Conceptos Clave Aprendidos

| Concepto | Descripción |
|---|---|
| **Pod** | Unidad mínima de despliegue. 1+ contenedores que comparten red y storage |
| **Deployment** | Declara el estado deseado (3 réplicas). K8s se encarga de mantenerlo |
| **Service** | IP estable que distribuye tráfico entre Pods (load balancing interno) |
| **Rolling Update** | K8s actualiza los Pods gradualmente sin downtime |
| **Readiness Probe** | Determina si un Pod está listo para recibir tráfico |
| **Liveness Probe** | Detecta Pods colgados y los reinicia automáticamente |
| **Resource Limits** | Evitan que un Pod consuma todos los recursos del nodo |

---

## ✅ Entregables

- [ ] Cluster de Minikube funcionando localmente.
- [ ] Deployment con 3 réplicas de la API corriendo.
- [ ] Service de tipo `ClusterIP` expuesto temporalmente con `minikube service`.
- [ ] Endpoint `/health` en la API con readiness y liveness probes.
- [ ] Rolling update sin downtime probado con `kubectl rollout status`.

---

*← [Semana 22](../6-mes/semana-22-aws-lambda-ec2.md) | [Volver al Roadmap](../../README.md) | [Semana 24 →](../6-mes/semana-24-k8s-fullstack.md)*
