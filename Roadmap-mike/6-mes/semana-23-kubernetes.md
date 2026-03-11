# Semana 23: Kubernetes Clusters (101)

## 🎯 Objetivo
Entender qué pasa cuando Docker o EC2 se te queda pequeño. Orquestar, curar (Self-Healing) y escalar contenedores automáticamente en un clúster utilizando el líder del mercado, K8s.

## 🛠️ Stack Tecnológico
- Kubernetes Avanzado / Minikube (para Local) o K3s
- Kubectl CLI
- Manifiestos (YAML) Pods, Deployments, Services

## 📋 Requisitos del Proyecto
1. Instalar `minikube` y `kubectl` en tu máquina local.
2. Emular un clúster de al menos 1 Master y 2 Nodes (Nodos Trabajadores).
3. Construir un archivo `deployment.yaml` para indicarle a K8s: "Asegúrate siempre de tener levantados mínimo 3 contenedores (Replicas) de mi Backend Node".
4. Ver tu backend correr, conectarte al clúster y matar a propósito uno de los pod (contenedores) vía CLI para observar cómo Kubernetes automáticamente recrea otro sin interacción humana.
5. Crear y asociar el `service.yaml` (Ej. Tipo LoadBalancer/NodePort) para poder consultar la API desde localhost.

## ✅ Entregables
- Aplicación a prueba de fallas levantada en Kubectl local, con Alta Disponibilidad.
