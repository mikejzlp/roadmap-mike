# 💬 Semana 10 — WebSockets & Chat en Tiempo Real (Socket.io)

> **Mes 3 · Full Stack Integration** | Stack: `Socket.io` · `Node.js` · `React` · `TypeScript`

---

## 🎯 Objetivo del Proyecto

Implementar comunicación **bidireccional en tiempo real** entre el servidor y múltiples clientes usando **WebSockets** con Socket.io. Construir una aplicación de chat grupal con salas, notificaciones de escritura y persistencia de mensajes.

---

## 🛠️ Stack Tecnológico

| Herramienta | Rol |
|---|---|
| Socket.io (server) | WebSockets con fallback automático a long-polling |
| Socket.io-client | Cliente WebSocket para React |
| MongoDB | Persistencia del historial de mensajes |
| React + TypeScript | Frontend del chat |

---

## 📐 Arquitectura del Sistema

```
semana-10-websockets-chat/
├── backend/
│   └── src/
│       ├── socket/
│       │   └── chat.socket.ts      # Toda la lógica de eventos Socket.io
│       ├── models/
│       │   └── Message.model.ts   # Persistencia de mensajes
│       └── app.ts
├── frontend/
│   └── src/
│       ├── hooks/
│       │   └── useSocket.ts       # Hook reutilizable para Socket.io
│       ├── components/
│       │   ├── ChatRoom.tsx
│       │   ├── MessageList.tsx
│       │   └── MessageInput.tsx
│       └── App.tsx
```

---

## 📋 Requisitos del Proyecto

### 1. Instalación

```bash
# Backend
npm install socket.io

# Frontend
npm install socket.io-client
```

### 2. Servidor Socket.io

```typescript
// backend/src/socket/chat.socket.ts
import { Server, Socket } from 'socket.io';

interface ChatMessage {
  userId: string;
  username: string;
  room: string;
  text: string;
  timestamp: Date;
}

export const initSocket = (io: Server) => {
  const activeUsers: Map<string, string> = new Map(); // socketId → username

  io.on('connection', (socket: Socket) => {
    console.log(`🟢 Cliente conectado: ${socket.id}`);

    // Unirse a una sala
    socket.on('join_room', ({ username, room }: { username: string; room: string }) => {
      socket.join(room);
      activeUsers.set(socket.id, username);

      // Notificar a la sala que alguien entró
      socket.to(room).emit('user_joined', { username, room });
      console.log(`${username} se unió a la sala: ${room}`);
    });

    // Enviar mensaje
    socket.on('send_message', async (message: ChatMessage) => {
      // Persistir en MongoDB
      // await Message.create(message);

      // Retransmitir a todos en la sala (incluyendo el emisor)
      io.to(message.room).emit('receive_message', {
        ...message,
        timestamp: new Date(),
      });
    });

    // Indicador "está escribiendo"
    socket.on('typing', ({ username, room }: { username: string; room: string }) => {
      socket.to(room).emit('user_typing', { username });
    });

    socket.on('stop_typing', ({ room }: { room: string }) => {
      socket.to(room).emit('user_stop_typing');
    });

    // Desconexión
    socket.on('disconnect', () => {
      const username = activeUsers.get(socket.id);
      activeUsers.delete(socket.id);
      console.log(`🔴 Cliente desconectado: ${username}`);
    });
  });
};
```

### 3. Integrar Socket.io con el servidor HTTP

```typescript
// backend/src/app.ts
import express from 'express';
import http from 'http';
import { Server } from 'socket.io';
import { initSocket } from './socket/chat.socket';

const app = express();
const httpServer = http.createServer(app);

const io = new Server(httpServer, {
  cors: {
    origin: 'http://localhost:5173',
    methods: ['GET', 'POST'],
  },
});

initSocket(io);

httpServer.listen(3000, () => console.log('Server on :3000'));
```

### 4. Hook de Socket.io en React

```typescript
// frontend/src/hooks/useSocket.ts
import { useEffect, useRef } from 'react';
import { io, Socket } from 'socket.io-client';

export const useSocket = (url: string) => {
  const socketRef = useRef<Socket | null>(null);

  useEffect(() => {
    socketRef.current = io(url, {
      autoConnect: true,
      transports: ['websocket'], // Forzar WebSocket, sin polling
    });

    return () => {
      socketRef.current?.disconnect();
    };
  }, [url]);

  return socketRef.current;
};
```

### 5. Componente ChatRoom

```typescript
// frontend/src/components/ChatRoom.tsx
import { useState, useEffect } from 'react';
import { io } from 'socket.io-client';

const socket = io('http://localhost:3000');

interface Message {
  username: string;
  text: string;
  timestamp: Date;
}

export const ChatRoom = ({ username, room }: { username: string; room: string }) => {
  const [messages, setMessages] = useState<Message[]>([]);
  const [input, setInput] = useState('');
  const [isTyping, setIsTyping] = useState('');

  useEffect(() => {
    socket.emit('join_room', { username, room });

    socket.on('receive_message', (msg: Message) => {
      setMessages((prev) => [...prev, msg]);
    });

    socket.on('user_typing', ({ username: u }: { username: string }) => {
      setIsTyping(`${u} está escribiendo...`);
    });

    socket.on('user_stop_typing', () => setIsTyping(''));

    return () => {
      socket.off('receive_message');
      socket.off('user_typing');
      socket.off('user_stop_typing');
    };
  }, [room]);

  const sendMessage = () => {
    if (!input.trim()) return;
    socket.emit('send_message', { username, room, text: input, timestamp: new Date() });
    socket.emit('stop_typing', { room });
    setInput('');
  };

  return (
    <div className="flex flex-col h-screen max-w-2xl mx-auto">
      <div className="flex-1 overflow-y-auto p-4 space-y-2">
        {messages.map((m, i) => (
          <div key={i} className={`p-3 rounded-lg max-w-xs ${m.username === username ? 'ml-auto bg-blue-500 text-white' : 'bg-gray-100'}`}>
            <p className="text-xs opacity-70">{m.username}</p>
            <p>{m.text}</p>
          </div>
        ))}
        {isTyping && <p className="text-sm text-gray-400 italic">{isTyping}</p>}
      </div>
      <div className="p-4 border-t flex gap-2">
        <input
          className="flex-1 border rounded-lg px-3 py-2"
          value={input}
          onChange={(e) => {
            setInput(e.target.value);
            socket.emit('typing', { username, room });
          }}
          onKeyDown={(e) => e.key === 'Enter' && sendMessage()}
          placeholder="Escribe un mensaje..."
        />
        <button onClick={sendMessage} className="bg-blue-500 text-white px-4 py-2 rounded-lg">
          Enviar
        </button>
      </div>
    </div>
  );
};
```

---

## 🧠 Conceptos Clave Aprendidos

- **WebSockets vs HTTP**: Los WebSockets mantienen una conexión persistente bidireccional, ideal para tiempo real. HTTP es request-response unidireccional.
- **Rooms (Salas)**: `socket.join(room)` y `io.to(room).emit()` permiten broadcast a grupos específicos.
- **Eventos**: Socket.io usa el patrón Pub/Sub: `emit` publica, `on` suscribe.
- **Cleanup de efectos**: Fundamental hacer `socket.off()` en el return del `useEffect` para evitar memory leaks con múltiples listeners.

---

## ✅ Entregables

- [ ] Servidor Socket.io con salas, mensajes y detección de escritura.
- [ ] Chat funcional con múltiples usuarios (probar con varias pestañas del navegador).
- [ ] Indicador "está escribiendo..." con debounce.
- [ ] Historial de mensajes persistido en MongoDB.

---

*← [Semana 09](../3-mes/semana-09-mern-dashboard.md) | [Volver al Roadmap](../../README.md) | [Semana 11 →](../3-mes/semana-11-pern-ecommerce.md)*
