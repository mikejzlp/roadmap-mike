# Semana 24: Despliegue Multi-Tier Kubernetes

## 🎯 Objetivo
Llevar a producción todo el conocimiento unificado, y crear despliegues de grado corporativo con LoadBalancers, bases de datos y secrets cifrados.

## 🛠️ Stack Tecnológico
- Kubernetes Avanzado (Ingress, StatefulSets)
- Helm Charts
- Cert-Manager / LetsEncrypt (HTTPS)

## 📋 Requisitos del Proyecto
1. Convertir la infraestructura de EC2 / Compose (Semana 22) a un Manifiesto completo de Kubernetes de múltiples capas (Frontend React en un lado, API Node en otro, y Redis/Postgres si corresponde).
2. Crear un archivo de ConfigMap (`configmap.yaml`) y Secrets (`secrets.yaml`) en K8s para inyectar allí por ambiente las URL de bases de datos/passwords seguras de tu proyecto.
3. Empaquetar el despliegue aprendiendo a escribir Archivos y Variables usando *Helm Charts* (`helm install my-ecommerce ./mi-chart`).
4. Añadir a un `ingress.yaml` redireccionamiento de tráfico con DNS reales (ej. apuntando subdominios como `api.tudominio.com` y `app.tudominio.com`).

## ✅ Entregables
- Arquitectura Multi-Capas orquestada en un clúster K8s bajo tu propio Chart de Helm parametrizado.
