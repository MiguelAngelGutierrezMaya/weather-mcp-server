# Weather MCP Server - Docker Deployment

Este documento describe cómo desplegar el servidor MCP de clima usando Docker Compose en producción.

## 🚀 Despliegue Rápido

### 1. Configuración Inicial

```bash
# Copiar archivo de variables de entorno
cp env.example .env

# Editar el archivo .env con tu API key
nano .env
```

### 2. Configurar API Keys

Edita el archivo `.env` y configura tu API key:

```env
API_KEYS=tu-api-key-aqui
USER_AGENT=weather-mcp-server/1.0
ENVIRONMENT=production
LOG_LEVEL=INFO
```

### 3. Desplegar el Servicio

```bash
./deploy.sh
```

## 📁 Estructura de Archivos

```
class10/
├── docker-compose.yml          # Configuración de producción
├── deploy.sh                   # Script de despliegue automatizado
├── env.example                 # Ejemplo de variables de entorno
├── .env                        # Variables de entorno (crear manualmente)
└── Dockerfile                  # Imagen del contenedor
```

## 🔧 Comandos Útiles

### Gestión del Servicio

```bash
# Ver logs en tiempo real
docker-compose logs -f weather-mcp-server

# Detener el servicio
docker-compose down

# Reiniciar el servicio
docker-compose restart

# Reconstruir la imagen
docker-compose build --no-cache

# Ver estado de los servicios
docker-compose ps

# Ver estado del servicio
docker-compose ps
```

### Despliegue

```bash
# Desplegar en producción
./deploy.sh

# Desplegar con rebuild completo
docker-compose up --build -d
```

## 🌐 Endpoints Disponibles

- **SSE Endpoint**: `http://localhost:8000/sse`
- **Health Check**: `http://localhost:8000/sse`
- **Messages Endpoint**: `http://localhost:8000/messages`

## 🔐 Autenticación

El servicio requiere autenticación mediante API key en el header `x-api-key`. Ejemplo:

```bash
curl -H "x-api-key: tu-api-key-aqui" http://localhost:8000/sse
```

## 📊 Monitoreo

### Health Check
El servicio incluye health checks automáticos que verifican:
- Disponibilidad del endpoint `/sse` con autenticación
- Estado del contenedor
- Uso de recursos

### Logs
Los logs se almacenan en:
- `./logs/` (volumen montado)
- `docker-compose logs weather-mcp-server`

## 🔧 Configuración Avanzada

### Variables de Entorno

| Variable | Descripción | Valor por Defecto |
|----------|-------------|-------------------|
| `API_KEYS` | API keys válidas (separadas por coma) | `your-api-key-here` |
| `USER_AGENT` | User agent para requests HTTP | `weather-mcp-server/1.0` |
| `ENVIRONMENT` | Entorno de ejecución | `production` |
| `LOG_LEVEL` | Nivel de logging | `INFO` |

### Recursos de Producción

- **CPU**: 0.5 cores (reserva) - 1.0 cores (límite)
- **Memoria**: 256MB (reserva) - 512MB (límite)
- **Réplicas**: 1 (configuración simple)
- **Restart Policy**: Always
- **Security**: No new privileges, read-only filesystem

## 🚨 Troubleshooting

### Problemas Comunes

1. **Error de API Key**:
   ```bash
   # Verificar que API_KEYS esté configurado en .env
   cat .env | grep API_KEYS
   ```

2. **Puerto ocupado**:
   ```bash
   # Verificar qué usa el puerto 8000
   lsof -i :8000
   ```

3. **Contenedor no inicia**:
   ```bash
   # Ver logs del contenedor
   docker-compose logs weather-mcp-server
   
   # Ver estado de contenedores
   docker-compose ps
   ```

4. **Health check falla**:
   ```bash
   # Verificar que curl esté disponible en el contenedor
   docker exec weather-mcp-server which curl
   
   # Probar health check manualmente
   curl -H "x-api-key: $API_KEYS" http://localhost:8000/sse
   ```

5. **Problemas de recursos**:
   ```bash
   # Ver uso de recursos
   docker stats weather-mcp-server
   
   # Ver logs del servicio
   docker-compose logs weather-mcp-server
   ```

### Soluciones

- **Reconstruir imagen**: `docker-compose build --no-cache`
- **Limpiar volúmenes**: `docker-compose down -v`
- **Reiniciar Docker**: `sudo systemctl restart docker`
- **Monitorear recursos**: `docker stats weather-mcp-server`

## 🔒 Seguridad

- El contenedor se ejecuta con `no-new-privileges`
- Sistema de archivos de solo lectura
- Variables de entorno para configuración sensible
- Health checks con autenticación
- Múltiples réplicas para alta disponibilidad

## 📈 Monitoreo

Para monitorear el servicio:

```bash
# Ver estadísticas de recursos
docker stats weather-mcp-server

# Ver logs en tiempo real
docker-compose logs -f weather-mcp-server

# Ver estado del servicio
docker-compose ps
```

## 🔗 Integración con VS Code

Configuración MCP para VS Code (`mcp.json`):

```json
{
    "inputs": [
        {
            "type": "promptString",
            "id": "weather-api-key",
            "description": "Weather API Key",
            "password": true
        }
    ],
    "servers": {
        "weather-sse": {
            "type": "sse",
            "url": "http://localhost:8000/sse",
            "headers": {
                "x-api-key": "${input:weather-api-key}"
            }
        }
    }
}
```

## 💻 Desarrollo Local

Para desarrollo local sin Docker, puedes ejecutar directamente:

```bash
# Instalar dependencias
uv sync

# Configurar API key
export API_KEYS=tu-api-key-aqui

# Ejecutar en modo desarrollo
uv run fastapi dev main.py --port 8000 --host 0.0.0.0
```

## 📋 Notas Importantes

- **Configuración simple**: Un solo contenedor para facilitar el despliegue
- **Recursos optimizados**: Configuración balanceada para producción
- **Health checks**: Monitoreo automático del servicio
- **Logs centralizados**: Todos los logs en un solo contenedor 