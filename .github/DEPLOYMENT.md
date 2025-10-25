# 🚀 Guía de Despliegue Automático

Este proyecto incluye workflows de GitHub Actions para desplegar automáticamente a un servidor remoto.

## 📋 Requisitos Previos

1. **Servidor remoto con:**
   - Docker y Docker Compose instalados
   - Git instalado
   - Acceso SSH habilitado
   - Puerto 5678 abierto para n8n

2. **Repositorio clonado en el servidor:**
   ```bash
   git clone git@github.com:Victorgf87/n8n-automation-k8s.git
   cd n8n-automation-k8s
   cp .env.example .env
   # Editar .env con tus valores
   ```

## 🔐 Configurar GitHub Secrets

En tu repositorio de GitHub, ve a **Settings → Secrets and variables → Actions** y añade:

### Secrets Necesarios:

1. **`SERVER_HOST`**: IP o dominio del servidor
   ```
   Ejemplo: 192.168.1.100 o ejemplo.com
   ```

2. **`SERVER_USER`**: Usuario SSH
   ```
   Ejemplo: ubuntu o root
   ```

3. **`SSH_PRIVATE_KEY`**: Clave privada SSH
   ```bash
   # Generar si no tienes:
   ssh-keygen -t ed25519 -C "github-actions"
   
   # Añadir la clave pública al servidor:
   ssh-copy-id usuario@servidor
   
   # Copiar el contenido de ~/.ssh/id_ed25519
   ```

## 🎯 Workflows Disponibles

### 1. Deploy Automático (Solo Código)

**Cuándo se ejecuta:** Cada push a `main`

**Qué hace:**
- Despliega cambios de configuración
- Actualiza Docker Compose
- **NO** incluye datos de n8n (workflows, credentials)

**Cómo usarlo:**
```bash
git push origin main
```

### 2. Deploy con Datos (Workflows + Credentials)

**Cuándo se ejecuta:** Manualmente desde GitHub Actions

**Qué hace:**
- Despliega configuración Y datos
- Incluye todos los workflows
- Incluye todas las credentials
- Restaura el estado completo de n8n

**Cómo usarlo:**
1. Ve a **Actions** en GitHub
2. Selecciona **"Deploy with Data"**
3. Haz clic en **"Run workflow"**
4. Selecciona si incluir datos o no
5. Haz clic en **"Run workflow"**

## 📝 Proceso de Despliegue

### Despliegue Automático (Solo Código):

```mermaid
Push a main → GitHub Actions → 
  Conectar al servidor → 
    Pull latest code → 
      Backup actual → 
        Stop services → 
          Pull images → 
            Start services → 
              ✅ Deployment complete
```

### Despliegue con Datos:

```mermaid
Manual trigger → GitHub Actions → 
  Copiar backup al servidor → 
    Restaurar datos → 
      Restart n8n → 
        ✅ All data deployed
```

## 🔄 Flujo de Trabajo Recomendado

### Desarrollo Local:
```bash
# Hacer cambios
./scripts/backup.sh  # Backup manual

# Si quieres compartir solo configuración:
git add .
git commit -m "Update config"
git push origin main  # Deploy automático

# Si quieres compartir CON datos:
# 1. Primero push normal
# 2. Luego ejecutar "Deploy with Data" manualmente
```

### Despliegue al Servidor:

**Opción A: Solo configuración**
```bash
git push origin main  # Automático
```

**Opción B: Con datos**
1. `./scripts/backup.sh` (en tu equipo local)
2. Commit y push el backup
3. Ejecutar workflow "Deploy with Data"

## ⚠️ Importante

- **Los backups deben estar en el repositorio** (o usar sistema de almacenamiento externo)
- **El servidor debe tener acceso SSH** desde GitHub Actions
- **Las variables de entorno** (.env) deben estar configuradas en ambos equipos

## 🐛 Troubleshooting

### Error: "Permission denied"

**Solución:** Verifica que la clave SSH pública esté en el servidor:
```bash
ssh usuario@servidor "cat ~/.ssh/authorized_keys"
```

### Error: "git pull failed"

**Solución:** Verifica permisos en el servidor:
```bash
chown -R usuario:usuario /path/to/n8n-automation-k8s
```

### Error: "docker compose not found"

**Solución:** Instala Docker Compose en el servidor:
```bash
sudo apt install docker-compose-plugin
```

## 📞 Soporte

Si tienes problemas, verifica:
1. Logs de GitHub Actions
2. Logs en el servidor: `docker compose logs`
3. Estado de servicios: `docker compose ps`
