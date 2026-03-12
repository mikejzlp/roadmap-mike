# 🎯 Semana 24 — Despliegue Multi-Tier en Kubernetes (Helm + Ingress)

> **Mes 6 · Cloud & Arquitectura Avanzada** | Stack: `Kubernetes` · `Helm` · `NGINX Ingress` · `K8s Secrets` · `EKS/GKE`

---

## 🎯 Objetivo del Proyecto

Desplegar la aplicación Full Stack completa en Kubernetes usando **Helm** como package manager, configurar un **Ingress Controller** para el enrutamiento HTTP, y gestionar configuraciones por entorno (dev/staging/prod) con **Helm values**.

---

## 🛠️ Stack Tecnológico

| Herramienta | Rol |
|---|---|
| Helm | Package manager para K8s (templates + values por entorno) |
| NGINX Ingress Controller | Enrutamiento HTTP/HTTPS hacia los servicios del cluster |
| cert-manager | Certificados SSL/TLS automáticos con Let's Encrypt |
| AWS EKS / GCP GKE | Cluster de Kubernetes gestionado en producción |
| ExternalSecrets | Sincronizar secrets de AWS Secret Manager a K8s |

---

## 📐 Arquitectura del Despliegue Completo

```
Internet
    │
    ▼
NGINX Ingress Controller (puerto 80/443)
    │
    ├── /api/* ──► api-service (ClusterIP) ──► api Pods (x3 réplicas)
    │                                               ↓
    │                                          postgres-service ──► PostgreSQL Pod
    │
    └── /     ──► frontend-service ──► frontend Pods (x2 réplicas, Nginx)

Secrets:
  - JWT_SECRET
  - DATABASE_URL
  - AWS_ACCESS_KEY
```

---

## 📋 Requisitos del Proyecto

### 1. Instalar Helm y crear un Chart

```bash
# Instalar Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Crear estructura del chart
helm create mi-app-chart
```

```
mi-app-chart/
├── Chart.yaml         # Metadata del chart
├── values.yaml        # Valores por defecto
├── values.prod.yaml   # Override para producción
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   └── hpa.yaml       # Auto-scaling
```

### 2. Template de Deployment con Helm

```yaml
# mi-app-chart/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-api
  labels:
    app: {{ .Chart.Name }}
    version: {{ .Chart.AppVersion }}
spec:
  replicas: {{ .Values.api.replicas }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}-api
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}-api
    spec:
      containers:
        - name: api
          image: "{{ .Values.api.image.repository }}:{{ .Values.api.image.tag }}"
          imagePullPolicy: {{ .Values.api.image.pullPolicy }}
          ports:
            - containerPort: {{ .Values.api.port }}
          envFrom:
            - configMapRef:
                name: {{ .Release.Name }}-config
            - secretRef:
                name: {{ .Release.Name }}-secrets
          resources:
            {{- toYaml .Values.api.resources | nindent 12 }}
```

### 3. Values por entorno

```yaml
# values.yaml (desarrollo - por defecto)
api:
  replicas: 1
  image:
    repository: tu-usuario/mi-api
    tag: latest
    pullPolicy: Always
  port: 3000
  resources:
    requests:
      cpu: "50m"
      memory: "64Mi"
    limits:
      cpu: "200m"
      memory: "256Mi"

frontend:
  replicas: 1
  image:
    repository: tu-usuario/mi-frontend
    tag: latest

ingress:
  enabled: true
  host: localhost
  tls: false
```

```yaml
# values.prod.yaml (producción — override)
api:
  replicas: 3
  image:
    tag: "1.5.0"     # Versión específica en producción (nunca: latest)
    pullPolicy: IfNotPresent
  resources:
    requests:
      cpu: "200m"
      memory: "256Mi"
    limits:
      cpu: "1000m"
      memory: "1Gi"

frontend:
  replicas: 2

ingress:
  host: mi-app.com
  tls: true          # HTTPS con Let's Encrypt
```

### 4. Ingress Controller con TLS

```yaml
# mi-app-chart/templates/ingress.yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    {{- if .Values.ingress.tls }}
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    {{- end }}
spec:
  ingressClassName: nginx
  {{- if .Values.ingress.tls }}
  tls:
    - hosts:
        - {{ .Values.ingress.host }}
      secretName: {{ .Release.Name }}-tls
  {{- end }}
  rules:
    - host: {{ .Values.ingress.host }}
      http:
        paths:
          - path: /api(/|$)(.*)
            pathType: Prefix
            backend:
              service:
                name: {{ .Release.Name }}-api
                port:
                  number: {{ .Values.api.port }}
          - path: /
            pathType: Prefix
            backend:
              service:
                name: {{ .Release.Name }}-frontend
                port:
                  number: 80
{{- end }}
```

### 5. HPA (Horizontal Pod Autoscaler)

```yaml
# mi-app-chart/templates/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ .Release.Name }}-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ .Release.Name }}-api
  minReplicas: {{ .Values.api.replicas }}
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70  # Escalar cuando CPU > 70%
```

### 6. Comandos de Helm

```bash
# Instalar el chart en desarrollo
helm install mi-release ./mi-app-chart

# Instalar en producción con values override
helm install mi-release ./mi-app-chart -f values.prod.yaml

# Actualizar (upgrade) con nuevo values
helm upgrade mi-release ./mi-app-chart -f values.prod.yaml

# Listar releases instalados
helm list

# Ver historial de un release
helm history mi-release

# Rollback a la versión anterior
helm rollback mi-release 1

# Desinstalar
helm uninstall mi-release

# Verificar los templates renderizados (sin aplicar)
helm template mi-release ./mi-app-chart -f values.prod.yaml
```

---

## 🧠 Conceptos Clave Aprendidos

| Concepto | Descripción |
|---|---|
| **Helm Chart** | Colección de templates K8s parametrizados. Equivalente a un "paquete" |
| **Values Files** | Separan la configuración del template. `values.prod.yaml` sobreescribe defaults |
| **Ingress Controller** | El único LoadBalancer externo. Enruta por path/host a los servicios internos |
| **TLS con cert-manager** | Certificados SSL automáticos y renovados desde Let's Encrypt |
| **HPA** | Escala los Pods automáticamente basándose en métricas de CPU/Memoria |

---

## ✅ Entregables

- [ ] Helm Chart completo con templates para API, Frontend, Ingress y HPA.
- [ ] Despliegue exitoso con `helm install` en Minikube.
- [ ] Ingress con routing `/api/*` → backend, `/` → frontend.
- [ ] HPA configurado para escalar entre 1 y 10 réplicas según CPU.
- [ ] `values.prod.yaml` diferenciado con imágenes fijas y recursos ajustados.

---

*← [Semana 23](../6-mes/semana-23-kubernetes.md) | [Volver al Roadmap](../../README.md) | [Semana 25 →](../6-mes/semana-25-capstone-erp.md)*
