# 🐧 Semana 18 — Linux & Bash Scripting

> **Mes 5 · DevOps, Docker & CI/CD** | Stack: `Linux` · `Bash` · `SSH` · `VIM` · `Servidores`

---

## 🎯 Objetivo del Proyecto

Dominar los **comandos esenciales de Linux** y la escritura de **scripts Bash** para automatizar tareas de servidor. Configurar un servidor remoto vía SSH y desplegar manualmente una aplicación Node.js, como base para entender lo que herramientas como Docker y CI/CD automatizan.

---

## 🛠️ Herramientas

| Herramienta | Rol |
|---|---|
| Ubuntu Server 22.04 | Sistema operativo de servidor |
| SSH | Acceso remoto seguro al servidor |
| VIM / Nano | Editores de texto en terminal |
| Bash | Shell de comandos para scripting |
| Nginx | Proxy inverso para servir la aplicación |
| PM2 | Process manager para Node.js en producción |

---

## 📋 Comandos Esenciales de Linux

### Navegación y archivos

```bash
# Navegación
pwd              # Directorio actual
ls -la           # Listar archivos (incluyendo ocultos y permisos)
cd /ruta/a/dir   # Cambiar directorio
cd ~             # Ir al home del usuario
cd -             # Volver al directorio anterior

# Manipulación de archivos
touch archivo.txt        # Crear archivo vacío
mkdir -p carpeta/subcarpeta  # Crear directorio (con padres)
cp origen destino        # Copiar
mv origen destino        # Mover / renombrar
rm -rf carpeta/          # Eliminar directorio recursivamente ⚠️
cat archivo.txt          # Ver contenido
less archivo.txt         # Ver contenido paginado (q para salir)
grep "texto" archivo.txt # Buscar texto en archivo
grep -r "texto" ./src/   # Buscar recursivamente
```

### Permisos

```bash
# Permisos: rwxrwxrwx (owner|group|others)
chmod 755 script.sh     # rwxr-xr-x (user: total, group/others: read+execute)
chmod +x script.sh      # Añadir permiso de ejecución
chown mike:mike archivo  # Cambiar propietario:grupo
```

### Procesos y sistema

```bash
top / htop              # Monitor de procesos en tiempo real
ps aux | grep node      # Ver procesos de node
kill -9 PID             # Matar proceso por ID
df -h                   # Espacio en disco
free -h                 # Uso de memoria RAM
netstat -tulpn          # Ver puertos en uso
```

### Gestión de paquetes (Ubuntu/Debian)

```bash
sudo apt update          # Actualizar índice de paquetes
sudo apt upgrade -y      # Instalar actualizaciones
sudo apt install nginx   # Instalar Nginx
sudo apt install -y curl git build-essential
```

---

## 🔑 Configuración de Acceso SSH

```bash
# En tu máquina local: generar par de claves SSH
ssh-keygen -t ed25519 -C "tu@email.com"
# Clave privada: ~/.ssh/id_ed25519
# Clave pública: ~/.ssh/id_ed25519.pub

# Copiar clave pública al servidor
ssh-copy-id usuario@IP_DEL_SERVIDOR

# Conectarse al servidor
ssh usuario@IP_DEL_SERVIDOR

# En el servidor: deshabilitar login por contraseña (más seguro)
sudo nano /etc/ssh/sshd_config
# Cambiar: PasswordAuthentication no
sudo systemctl restart sshd
```

---

## 📜 Bash Scripting

### Script de deploy de Node.js

```bash
#!/bin/bash
# deploy.sh — Script de despliegue automático

set -e  # Salir si cualquier comando falla
set -u  # Salir si se usa variable no definida

# Variables
APP_DIR="/var/www/mi-api"
GIT_REPO="https://github.com/usuario/mi-api.git"
APP_NAME="mi-api"

log() { echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1"; }

log "🚀 Iniciando deploy..."

# 1. Clonar o actualizar el repositorio
if [ -d "$APP_DIR" ]; then
  log "📦 Actualizando repositorio..."
  cd "$APP_DIR"
  git pull origin main
else
  log "📦 Clonando repositorio..."
  git clone "$GIT_REPO" "$APP_DIR"
  cd "$APP_DIR"
fi

# 2. Instalar dependencias
log "📦 Instalando dependencias..."
npm ci --production

# 3. Compilar TypeScript
log "🔨 Compilando TypeScript..."
npm run build

# 4. Reiniciar con PM2
log "♻️  Reiniciando aplicación con PM2..."
pm2 restart "$APP_NAME" 2>/dev/null || pm2 start dist/app.js --name "$APP_NAME"
pm2 save

log "✅ Deploy completado exitosamente!"
```

```bash
# Hacer el script ejecutable y ejecutarlo
chmod +x deploy.sh
./deploy.sh
```

---

## 🌐 Configurar Nginx como Proxy Inverso

```nginx
# /etc/nginx/sites-available/mi-api
server {
    listen 80;
    server_name mi-api.com www.mi-api.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```bash
# Activar site y recargar Nginx
sudo ln -s /etc/nginx/sites-available/mi-api /etc/nginx/sites-enabled/
sudo nginx -t           # Verificar configuración
sudo systemctl reload nginx
```

---

## 🧠 Conceptos Clave Aprendidos

- **SSH Keys**: Más seguras que contraseñas. La clave privada nunca sale de tu máquina.
- **`set -e` en Bash**: Hace que el script falle rápido si hay errores, evitando estados inconsistentes.
- **Nginx como Proxy Inverso**: Recibe tráfico en el puerto 80/443 y lo redirige a la app Node.js en el 3000.
- **PM2**: Asegura que la app Node.js se reinicie automáticamente si falla o si el servidor se reinicia.

---

## ✅ Entregables

- [ ] Script `deploy.sh` que clona/actualiza el repo, instala dependencias y reinicia PM2.
- [ ] Nginx configurado como proxy inverso apuntando a la app de Node.
- [ ] Acceso SSH con claves (sin contraseña).
- [ ] App Node.js corriendo en el servidor y accesible por HTTP.

---

*← [Semana 17](../4-mes/semana-17-rn-advanced.md) | [Volver al Roadmap](../../README.md) | [Semana 19 →](../5-mes/semana-19-docker-compose.md)*
