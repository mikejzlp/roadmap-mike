# Semana 10: App en Tiempo Real (WebSockets)

## 🎯 Objetivo
Entender el paradigma de comunicación bidireccional cliente/servidor fuera del ciclo tradicional de petición/respuesta HTTP.

## 🛠️ Stack Tecnológico
- ReactJS
- NodeJS + Socket.io
- Eventos Realtime

## 📋 Requisitos del Proyecto
1. Levantar un servidor WebSocket junto a Express usando `socket.io`.
2. Crear un Chat global o un "Pizarrón Colaborativo" en React.
3. Emitir eventos desde el cliente (ej. *'send_message'*).
4. Escuchar eventos desde el servidor y hacer un *Broadcast* a todos los clientes conectados exceptuando el emisor original.
5. Gestionar la conexión y desconexión en el ciclo de vida del componente React (`useEffect` cleanup function).

## ✅ Entregables
- Aplicación de mensajería instantánea funcional abriendo dos ventanas o navegadores distintos al mismo tiempo.
