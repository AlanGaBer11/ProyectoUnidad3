# Solución: Error de Puerto 80 en Windows

## Problema

```
Error response from daemon: ports are not available: exposing port TCP 0.0.0.0:80 -> 127.0.0.1:0: 
listen tcp 0.0.0.0:80: bind: Intento de acceso a un socket no permitido por sus permisos de acceso.
```

## Causa

En Windows, el puerto 80 (HTTP) y 443 (HTTPS) son puertos privilegiados que requieren **permisos de administrador** para ser utilizados. Esto es por seguridad del sistema operativo.

## Solución Implementada

Se han cambiado los puertos de Nginx a puertos no privilegiados:

### Cambios Realizados

| Servicio | Puerto Anterior | Puerto Nuevo | Descripción |
|----------|-----------------|--------------|-------------|
| Nginx (HTTP) | 80 | **9080** | Load balancer principal |
| Nginx (HTTPS) | 443 | **9443** | SSL (para futuro) |
| Nginx Blue (directo) | 8080 | **9081** | Acceso directo a Blue |
| Nginx Green (directo) | 8081 | **9082** | Acceso directo a Green |

## Nuevas URLs de Acceso

### A través del Load Balancer (Recomendado)
```
http://localhost:9080              → Tráfico balanceado (Blue/Green)
http://localhost:9080/health       → Health check del servicio activo
http://localhost:9080/api/v1/validate → Endpoint de validación
http://localhost:9080/metrics      → Métricas de Prometheus
```

### Acceso Directo a Ambientes
```
http://localhost:9081              → Acceso directo a Blue
http://localhost:9082              → Acceso directo a Green
```

### Servicios Directos (Sin Load Balancer)
```
http://localhost:8000              → Blue directo (puerto del contenedor)
http://localhost:8001              → Green directo (puerto del contenedor)
```

## Cómo Reiniciar

### 1. Detener servicios actuales
```powershell
docker-compose down
```

### 2. Iniciar todos los servicios
```powershell
docker-compose up -d
```

### 3. Verificar que todos estén corriendo
```powershell
docker-compose ps
```

Deberías ver **8/8 servicios** en estado "Up":
- ✅ transaction-validator-blue
- ✅ prometheus
- ✅ grafana
- ✅ jaeger
- ✅ elasticsearch
- ✅ logstash
- ✅ kibana
- ✅ nginx-lb

### 4. Probar el servicio
```powershell
# Test a través del load balancer
Invoke-WebRequest -Uri http://localhost:9080/health

# Test directo a Blue
Invoke-WebRequest -Uri http://localhost:8000/health

# Ver en navegador
Start-Process http://localhost:9080
```

## Alternativa: Ejecutar Docker Desktop como Administrador

Si prefieres usar el puerto 80 original:

1. Cerrar Docker Desktop
2. Click derecho en Docker Desktop → "Ejecutar como administrador"
3. Revertir los cambios en `docker-compose.yml` (puerto 8080 → 80)
4. `docker-compose up -d`

**⚠️ No recomendado:** Ejecutar Docker como admin puede ser un riesgo de seguridad.

## Verificación de Puertos

Para ver qué está usando cada puerto:

```powershell
# Ver todos los puertos en uso
netstat -ano | findstr "9080 9081 9082"

# Ver procesos de Docker
docker ps --format "table {{.Names}}\t{{.Ports}}"
```

## Actualización del Script de Deployment

El script de deployment `deploy-blue-green.ps1` ha sido actualizado automáticamente para usar `http://localhost:8080` en lugar de `http://localhost:80`.

## URLs Actualizadas en Toda la Documentación

✅ README.md  
✅ QUICKSTART.md  
✅ docker-compose.yml  
✅ monitoring/nginx/nginx.conf  
✅ scripts/deploy-blue-green.ps1  

---

## Resumen

**Antes:**
```
http://localhost/          ❌ Error de permisos
http://localhost:8080/     ❌ Puerto ocupado por Sistema
```

**Ahora:**
```
http://localhost:9080/     ✅ Funciona sin problemas
```

Todo funciona exactamente igual, solo cambió el número de puerto. No se requieren permisos de administrador.
