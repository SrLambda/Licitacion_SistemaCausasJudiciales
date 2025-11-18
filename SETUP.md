# 🚀 Guía de Instalación para Compañeros de Equipo

## ⚠️ IMPORTANTE: Pasos Obligatorios

Si estás clonando este proyecto por primera vez, **DEBES** seguir estos pasos:

### 1️⃣ Clonar el Repositorio

```bash
git clone git@github.com:SrLambda/ProyectoLicitacion_Admin2025_2.git
cd ProyectoLicitacion_Admin2025_2
```

### 2️⃣ Checkout a la Branch Correcta

```bash
# Usar la branch de desarrollo actual
git checkout merge-demian-cacata-hibrido

# Verificar que estás en la branch correcta
git branch
```

### 3️⃣ Crear el Archivo `.env` (OBLIGATORIO)

El archivo `.env` contiene las credenciales y configuraciones del sistema. **NO está en el repo por seguridad**.

```bash
# Copiar el template
cp .env.example .env
```

### 4️⃣ Configurar Variables Críticas en `.env`

Abre el archivo `.env` y **DEBES configurar estas variables**:

#### 🔑 Base de Datos (Obligatorio)
```bash
MYSQL_ROOT_PASSWORD=root_password_2025
MYSQL_DATABASE=causas_judiciales_db
MYSQL_USER=admin_db
MYSQL_PASSWORD=password
```

#### 🔐 JWT y Autenticación (Obligatorio)
```bash
JWT_SECRET=secreto_2025
JWT_EXPIRE=24h
JWT_SECRET_KEY=deaa123
CASOS_SECRET_KEY=7c1c8bd5bf5cf8d6be0c534c67c2e9f0ecad01f5d521bc90f8e45a550e4f4cf2
```

#### 🔴 Redis (Obligatorio)
```bash
REDIS_PASSWORD=redis_2025
```

#### 🤖 IA con Gemini (Opcional - pero recomendado)
```bash
GEMINI_API_KEY=tu-api-key-de-gemini-aqui
```

**Cómo obtener tu API Key de Gemini (GRATIS):**
1. Ve a https://aistudio.google.com/app/apikey
2. Inicia sesión con tu cuenta de Google
3. Click en "Get API key" → "Create API key"
4. Copia la key y pégala en `.env`

⚠️ **Importante**: El tier gratuito tiene límite de 15 requests/minuto. El sistema ya tiene rate limiting implementado.

#### 📧 Email (Opcional)
```bash
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=tu_email@gmail.com
SMTP_PASSWORD=tu_password_de_aplicacion
```

### 5️⃣ Levantar el Sistema

```bash
# Construir e iniciar todos los servicios
docker-compose up -d

# Ver los logs en tiempo real (CTRL+C para salir)
docker-compose logs -f

# Verificar que todos los servicios estén corriendo
docker-compose ps
```

**Deberías ver 21 contenedores corriendo:**
- 2 Frontend (frontend-1, frontend-2)
- 2 Casos (casos-1, casos-2)
- 1 Autenticación
- 1 Documentos
- 1 Notificaciones
- 1 Reportes
- 1 IA Seguridad
- 1 Gateway (Traefik)
- 1 MySQL Master
- 1 MySQL Slave
- 1 ProxySQL
- 1 Redis Master
- 1 Redis Replica
- 1 Prometheus
- 1 Grafana
- 1 MailHog
- 1 Backup Service
- 1 Failover Daemon
- 1 Cron

### 6️⃣ Acceder al Sistema

#### 🌐 URLs Principales
- **Frontend**: http://localhost
- **API Gateway Dashboard**: http://localhost:8080
- **Grafana (Monitoreo)**: http://localhost:3000
  - Usuario: `admin`
  - Contraseña: `admin_2025`
- **MailHog (Email Testing)**: http://localhost:8025

#### 👤 Usuarios de Prueba
| Rol | Usuario | Contraseña |
|-----|---------|-----------|
| Administrador | admin@judicial.cl | Admin123! |
| Abogado | abogado@judicial.cl | Abogado123! |

---

## 🐛 Solución de Problemas Comunes

### ❌ Error: "Cannot connect to database"
**Causa**: El archivo `.env` no existe o las credenciales son incorrectas.

**Solución**:
```bash
# Verificar que .env existe
ls -la .env

# Si no existe, crearlo desde el template
cp .env.example .env
```

### ❌ Error: "Port 80 is already allocated"
**Causa**: Otro servicio está usando el puerto 80.

**Solución**:
```bash
# En macOS, detener Apache si está corriendo
sudo apachectl stop

# O cambiar el puerto en docker-compose.yml
# Buscar "80:80" y cambiar por "8000:80"
```

### ❌ Error: "docker: command not found"
**Causa**: Docker Desktop no está instalado.

**Solución**:
1. Descargar Docker Desktop: https://www.docker.com/products/docker-desktop/
2. Instalar y abrir Docker Desktop
3. Verificar instalación: `docker --version`

### ❌ Los contenedores se reinician constantemente
**Causa**: Falta de memoria RAM o conflictos de red.

**Solución**:
```bash
# Limpiar contenedores y volúmenes
docker-compose down -v

# Limpiar imágenes huérfanas
docker system prune -a

# Volver a levantar
docker-compose up -d
```

### ⚠️ IA no funciona (Error 429)
**Causa**: Límite de rate de Gemini API excedido (15 RPM).

**Solución**: El sistema ya tiene rate limiting. Espera 1 hora para que se resetee el límite, o:
```bash
# Detener el servicio de IA temporalmente
docker stop ia-seguridad

# El sistema funciona sin IA, solo sin análisis de logs
```

---

## 📊 Verificar que Todo Funciona

```bash
# Ver estado de todos los contenedores
docker-compose ps

# Ver logs de un servicio específico
docker-compose logs -f frontend-1
docker-compose logs -f db-master
docker-compose logs -f ia-seguridad

# Verificar replicación MySQL
docker exec db-master mysql -u root -proot_password_2025 -e "SHOW MASTER STATUS\G"
docker exec db-slave mysql -u root -proot_password_2025 -e "SHOW REPLICA STATUS\G"

# Verificar replicación Redis
docker exec redis redis-cli -a redis_2025 INFO replication
docker exec redis-replica redis-cli -a redis_2025 INFO replication

# Ver estadísticas de ProxySQL
docker exec db-proxy mysql -u admin -padmin -h127.0.0.1 -P6032 -e "SELECT * FROM stats.stats_mysql_query_digest;"
```

---

## 🔄 Actualizar el Código

Cuando alguien haga cambios y los suba a GitHub:

```bash
# Descargar últimos cambios
git pull origin merge-demian-cacata-hibrido

# Reconstruir servicios que cambiaron
docker-compose up -d --build

# Si hay cambios en la base de datos
docker-compose restart db-master db-slave
```

---

## 🛑 Detener el Sistema

```bash
# Detener todos los servicios (mantiene volúmenes)
docker-compose down

# Detener y ELIMINAR volúmenes (perderás los datos)
docker-compose down -v

# Detener solo un servicio
docker-compose stop frontend-1
```

---

## 📞 ¿Necesitas Ayuda?

Si algo no funciona:
1. Lee los logs: `docker-compose logs -f [nombre-servicio]`
2. Verifica que `.env` esté configurado correctamente
3. Asegúrate de estar en la branch `merge-demian-cacata-hibrido`
4. Contacta al equipo en el grupo

---

## 🎯 Checklist de Setup Exitoso

- [ ] Clonaste el repo
- [ ] Checkout a `merge-demian-cacata-hibrido`
- [ ] Creaste `.env` desde `.env.example`
- [ ] Configuraste las variables obligatorias en `.env`
- [ ] Ejecutaste `docker-compose up -d`
- [ ] Viste 21 contenedores corriendo en `docker-compose ps`
- [ ] Accediste a http://localhost y viste el frontend
- [ ] Probaste login con `admin@judicial.cl` / `Admin123!`

**Si completaste todo ✅, ¡estás listo para desarrollar!**
