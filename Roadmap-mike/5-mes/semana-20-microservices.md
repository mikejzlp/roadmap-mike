# 🏗️ Semana 20 — Microservicios Básicos con Nginx

> **Mes 5 · DevOps, Docker & CI/CD** | Stack: `Microservicios` · `Nginx` · `Docker Compose` · `API Gateway` · `Node.js`

---

## 🎯 Objetivo del Proyecto

Descomponer una aplicación monolítica en **microservicios independientes** y configurar **Nginx como API Gateway** y balanceador de carga. Entender cuándo los microservicios son la solución correcta y los desafíos que introducen.

---

## 🛠️ Stack Tecnológico

| Herramienta | Rol |
|---|---|
| Nginx | API Gateway + Load Balancer + Proxy inverso |
| Docker Compose | Orquestación de múltiples microservicios |
| Node.js + Express | Cada microservicio es una app Express independiente |
| RabbitMQ / Redis | Mensajería asíncrona entre servicios |

---

## 📐 Arquitectura de Microservicios

```
                     CLIENTE (Browser / Mobile)
                              │
                              ▼
                    ┌─────────────────┐
                    │  NGINX (Puerto 80) │
                    │   API Gateway    │
                    └────────┬────────┘
          ┌──────────────────┼───────────────────┐
          ▼                  ▼                   ▼
  /api/auth →        /api/users →         /api/products →
  ┌──────────┐      ┌──────────┐         ┌──────────────┐
  │   Auth   │      │  Users   │         │   Products   │
  │ Service  │      │ Service  │         │   Service    │
  │ :3001    │      │ :3002    │         │   :3003      │
  └──────────┘      └──────────┘         └──────────────┘
       │                 │                      │
  ┌────┴────┐      ┌─────┴────┐         ┌──────┴──────┐
  │ Postgres│      │ Postgres │         │   MongoDB   │
  │ (auth)  │      │ (users)  │         │ (products)  │
  └─────────┘      └──────────┘         └─────────────┘
```

---

## 📋 Requisitos del Proyecto

### 1. Nginx como API Gateway

```nginx
# nginx/nginx.conf

upstream auth_service {
    server auth-service:3001;
}

upstream users_service {
    # Balanceo de carga entre 2 instancias
    server users-service-1:3002;
    server users-service-2:3002;
}

upstream products_service {
    server products-service:3003;
}

server {
    listen 80;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

    location /api/auth/ {
        limit_req zone=api burst=20 nodelay;
        proxy_pass http://auth_service/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /api/users/ {
        proxy_pass http://users_service/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        # Pasar el JWT de Auth para que los servicios puedan verificarlo
        proxy_set_header Authorization $http_authorization;
    }

    location /api/products/ {
        proxy_pass http://products_service/;
        proxy_set_header Host $host;
    }
}
```

### 2. Docker Compose para todos los Microservicios

```yaml
# docker-compose.yml
version: '3.9'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - auth-service
      - users-service-1
      - users-service-2
      - products-service
    networks:
      - gateway

  auth-service:
    build: ./services/auth
    environment:
      DATABASE_URL: postgresql://admin:pass@auth-db:5432/auth_db
      JWT_SECRET: ${JWT_SECRET}
    depends_on:
      - auth-db
    networks:
      - gateway
      - auth-network

  users-service-1:
    build: ./services/users
    environment:
      DATABASE_URL: postgresql://admin:pass@users-db:5432/users_db
      JWT_SECRET: ${JWT_SECRET}
    depends_on:
      - users-db
    networks:
      - gateway
      - users-network

  users-service-2:    # Segunda instancia para balanceo de carga
    build: ./services/users
    environment:
      DATABASE_URL: postgresql://admin:pass@users-db:5432/users_db
      JWT_SECRET: ${JWT_SECRET}
    depends_on:
      - users-db
    networks:
      - gateway
      - users-network

  products-service:
    build: ./services/products
    environment:
      MONGO_URI: mongodb://mongo:27017/products_db
    depends_on:
      - mongo
    networks:
      - gateway
      - products-network

  auth-db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: auth_db
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: pass
    volumes:
      - auth_postgres_data:/var/lib/postgresql/data
    networks:
      - auth-network

  users-db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: users_db
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: pass
    volumes:
      - users_postgres_data:/var/lib/postgresql/data
    networks:
      - users-network

  mongo:
    image: mongo:7-jammy
    volumes:
      - mongo_data:/data/db
    networks:
      - products-network

volumes:
  auth_postgres_data:
  users_postgres_data:
  mongo_data:

networks:
  gateway:        # Red pública (Nginx ↔ servicios)
  auth-network:   # Red privada (auth-service ↔ auth-db)
  users-network:  # Red privada (users-service ↔ users-db)
  products-network:
```

### 3. Comunicación entre microservicios (via HTTP)

```typescript
// services/orders/src/services/user.service.ts
// Cuando el servicio de órdenes necesita datos del servicio de usuarios
import axios from 'axios';

export const getUserById = async (userId: string, authToken: string) => {
  // Se comunica vía el API Gateway (Nginx) o directamente por red interna
  const { data } = await axios.get(`http://users-service:3002/users/${userId}`, {
    headers: { Authorization: `Bearer ${authToken}` },
  });
  return data;
};
```

---

## ⚖️ Monolito vs Microservicios

| Aspecto | Monolito | Microservicios |
|---|---|---|
| **Complejidad** | Baja | Alta (redes, descubrimiento, logs distribuidos) |
| **Escalabilidad** | Escala toda la app | Escala servicios individuales |
| **Deploy** | Un solo artefacto | Múltiples servicios independientes |
| **Ideal para** | Startups, equipos pequeños | Empresas con múltiples equipos |
| **Cuando migrar** | Cuando el equipo crece | Cuando hay cuellos de botella en servicios específicos |

---

## 🧠 Conceptos Clave Aprendidos

- **API Gateway**: Punto único de entrada que enruta, autentica y hace rate limiting de las requests.
- **Balanceo de carga (Round Robin)**: Nginx distribuye las peticiones entre múltiples instancias del mismo servicio.
- **Redes de Docker separadas**: Aíslan la comunicación. Los servicios no pueden comunicarse entre sí a menos que compartan una red.
- **Rate Limiting con Nginx**: `limit_req_zone` protege contra ataques de fuerza bruta y DDoS básicos.

---

## ✅ Entregables

- [ ] 3 microservicios independientes (auth, users, products) con sus propias DBs.
- [ ] Nginx configurado como API Gateway con routing por path.
- [ ] Balanceo de carga con al menos 2 instancias del servicio de usuarios.
- [ ] Rate limiting configurado en Nginx.
- [ ] Todos los servicios accesibles desde `http://localhost` vía Nginx.

---

*← [Semana 19](../5-mes/semana-19-docker-compose.md) | [Volver al Roadmap](../../README.md) | [Semana 21 →](../5-mes/semana-21-github-actions.md)*
