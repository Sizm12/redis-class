# Escenarios Reales de Implementación con Redis

## 1. Sistema de Carrito de Compras en Tiempo Real

### Contexto
Un sitio de comercio electrónico necesita manejar miles de carritos de compras simultáneamente, con actualizaciones en tiempo real y sin sobrecargar la base de datos principal.

### Implementación con Redis

```bash
# 1. Añadir producto al carrito
HSET carrito:usuario123 producto:42 2  # 2 unidades del producto 42
HSET carrito:usuario123 producto:15 1  # 1 unidad del producto 15

# 2. Incrementar cantidad
HINCRBY carrito:usuario123 producto:42 1

# 3. Ver contenido del carrito
HGETALL carrito:usuario123

# 4. Calcular total de productos
EVAL "local total = 0 for k,v in pairs(redis.call('HGETALL', KEYS[1])) do if k:find('producto:') then total = total + tonumber(v) end end return total" 1 carrito:usuario123

# 5. Establecer expiración de 1 día para carritos inactivos
EXPIRE carrito:usuario123 86400
```

### Beneficios
- Baja latencia para actualizaciones frecuentes
- No sobrecarga la base de datos principal
- Los carritos inactivos se limpian automáticamente
- Fácil de escalar horizontalmente

## 2. Sistema de Notificaciones en Tiem Real

### Contexto
Una aplicación móvil necesita enviar notificaciones push a millones de usuarios, con soporte para diferentes canales (app, email, SMS) y prioridades.

### Implementación con Redis

```bash
# 1. Publicar notificación en múltiples canales
PUBLISH notificaciones:usuario123 '{"tipo": "mensaje", "texto": "Tienes un nuevo mensaje", "canal": "app"}'
PUBLISH notificaciones:usuario123 '{"tipo": "promocion", "texto": "Oferta especial", "canal": "email"}'

# 2. Cola de notificaciones pendientes (para reintentos)
LPUSH cola:notificaciones '{"usuario": "usuario123", "mensaje": "...", "intentos": 0, "proximo_intento": 1643727600}'

# 3. Procesamiento programado (usando Redis Sorted Sets)
ZADD programadas 1643727600 'notificacion:promo:verano:usuario123'

# 4. Obtener notificaciones programadas para el próximo minuto
ZRANGEBYSCORE programadas 1643727540 1643727600
```

### Beneficios
- Baja latencia para entrega de notificaciones
- Soporte para millones de conexiones simultáneas
- Garantía de entrega con sistema de reintentos
- Programación flexible de notificaciones

## 3. Sistema de Análisis en Tiempo Real

### Contexto
Una plataforma de streaming necesita analizar el comportamiento de los usuarios en tiempo real para ofrecer recomendaciones personalizadas y detectar tendencias.

### Implementación con Redis

```bash
# 1. Seguimiento de reproducciones
ZINCRBY estadisticas:reproducciones:daily 1 video:42

# 2. Conteo de usuarios únicos por video
PFADD usuarios_unicos:video:42 usuario123 usuario456

# 3. Almacenamiento de eventos de visualización
XADD stream:eventos:video:42 * usuario usuario123 accion "reproduccion_iniciada" tiempo 120
XADD stream:eventos:video:42 * usuario usuario123 accion "pausa" tiempo 180

# 4. Análisis de sesiones
ZADD sesion:usuario123 1643727600 '{"evento": "login", "ip": "192.168.1.1"}'
ZADD sesion:usuario123 1643727660 '{"evento": "busqueda", "termino": "tutorial redis"}'
ZADD sesion:usuario123 1643727720 '{"evento": "reproduccion", "video": "introduccion-redis"}'

# 5. Análisis de tendencias (usando HyperLogLog para conteo único)
PFADD tendencias:dia:2025-10-02:busquedas "redis" "docker" "kubernetes"
PFCOUNT tendencias:dia:2025-10-02:busquedas
```

### Beneficios
- Procesamiento de eventos en tiempo real
- Escalabilidad para manejar grandes volúmenes de datos
- Bajo consumo de recursos para conteos únicos
- Análisis de patrones de comportamiento

## 4. Sistema de Cola de Tareas con Prioridad

### Contexto
Una plataforma de procesamiento de imágenes necesita manejar tareas con diferentes niveles de prioridad, donde las tareas de mayor prioridad deben procesarse primero.

### Implementación con Redis

```bash
# 1. Añadir tareas con diferentes prioridades
ZADD cola:tareas 1 '{"id": "tarea1", "tipo": "procesar_imagen", "prioridad": 1, "datos": "..."}'
ZADD cola:tareas 3 '{"id": "tarea2", "tipo": "generar_miniatura", "prioridad": 3, "datos": "..."}'
ZADD cola:tareas 1 '{"id": "tarea3", "tipo": "procesar_imagen", "prioridad": 1, "datos": "..."}'

# 2. Obtener la siguiente tarea a procesar (la de menor puntuación = mayor prioridad)
ZRANGE cola:tareas 0 0 WITHSCORES

# 3. Bloquear tarea durante el procesamiento
ZREM cola:tareas '{"id": "tarea1", ...}'
SET tarea:procesando:tarea1 1 EX 300  # Bloquear por 5 minutos

# 4. Si la tarea falla, reintentar más tarde
ZADD cola:tareas 5 '{"id": "tarea1", ..., "intentos": 1}'
```

### Beneficios
- Manejo eficiente de prioridades
- Garantía de procesamiento (no se pierden tareas)
- Distribución de carga entre múltiples workers
- Reintentos automáticos para tareas fallidas

## 5. Sistema de Caché Distribuido

### Contexto
Un sitio web de alto tráfico necesita reducir la carga en su base de datos principal almacenando en caché resultados de consultas frecuentes.

### Implementación con Redis

```bash
# 1. Almacenar en caché con tiempo de expiración
SETEX cache:producto:detalles:42 3600 '{"id": 42, "nombre": "Producto X", "precio": 99.99}'

# 2. Patrón de cache-aside
# Verificar caché primero
GET cache:producto:detalles:42
# Si no está en caché, obtener de la base de datos y almacenar
SETEX cache:producto:detalles:42 3600 '{"id": 42, ...}'

# 3. Invalidar caché cuando los datos cambian
DEL cache:producto:detalles:42

# 4. Caché de fragmentos para páginas web
HSET cache:fragmentos:header usuario:123 '{"notificaciones": 5, "mensajes": 2}'
HSET cache:fragmentos:recomendaciones usuario:123 '[...]'
```

### Beneficios
- Reducción significativa de la carga en la base de datos
- Mejora en los tiempos de respuesta
- Fácil implementación de estrategias de invalidación
- Soporte para diferentes tiempos de expiración por tipo de dato
