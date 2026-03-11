# Semana 18: Linux, VIM y Bash Scripting (Fundamentos DevOps)

## 🎯 Objetivo
Olvidarte de las interfaces gráficas de los servidores. Perderle el miedo a la Terminal para poder gestionar despliegues productivos.

## 🛠️ Stack Tecnológico
- Linux Ubuntu/Debian
- Shell / Bash (Zsh)
- VIM / Nano
- SSH Protocol (RSA/Ed25519)

## 📋 Requisitos del Proyecto
1. Instalar un servidor local Linux (vía Máquina Virtual o WSL) o contratar un VPS básico (Hetzner, AWS EC2, DigitalOcean).
2. Conectarse al servidor por SSH usando llaves criptográficas (no contraseñas).
3. Usar VIM o Nano directamente en la terminal remota para editar archivos de configuración (`.env`, `nginx.conf`).
4. Escribir un *Bash Script* (`deploy.sh`) que automáticamente haga `git pull`, instale dependencias (`npm ci`), construya el proyecto y lo reinicie (`pm2 restart app`).

## ✅ Entregables
- Scripts de bash funcionales automatizando tareas mínimas (ej. Backup de la Base de datos y reseteo del servicio).
