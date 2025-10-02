# Redis Commands Cheatsheet

## Operaciones Básicas (Basic Operations)

| Comando | Descripción | Ejemplo | Uso típico |
|---------|-------------|---------|------------|
| `SET key value` | Almacena un valor | `SET usuario:1 "Samuel"` | Guardar el nombre de un usuario |
| `GET key` | Recupera un valor | `GET usuario:1` | Consultar datos en caché |
| `DEL key` | Elimina una clave | `DEL usuario:1` | Borrar datos obsoletos |
| `EXPIRE key segundos` | Define caducidad de clave | `EXPIRE usuario:1 60` | Cache con TTL (ej: token válido 60s) |
| `KEYS pattern` | Lista claves con patrón | `KEYS usuario:*` | Ver todas las claves de usuarios |
| `TTL key` | Tiempo de vida restante | `TTL usuario:1` | Verificar si un token aún es válido |

## Listas (Lists)

| Comando | Descripción | Ejemplo | Uso típico |
|---------|-------------|---------|------------|
| `LPUSH key value` | Inserta al inicio | `LPUSH tareas "limpiar"` | Agregar tareas pendientes |
| `RPUSH key value` | Inserta al final | `RPUSH tareas "comprar pan"` | Cola de mensajes |
| `LPOP key` | Saca primer elemento | `LPOP tareas` | Procesar primer trabajo en cola |
| `RPOP key` | Saca último elemento | `RPOP tareas` | Procesar último insertado |
| `LRANGE key start stop` | Obtiene un rango | `LRANGE tareas 0 -1` | Ver todas las tareas |

## Hashes

| Comando | Descripción | Ejemplo | Uso típico |
|---------|-------------|---------|------------|
| `HSET key campo valor` | Inserta campo-valor | `HSET user:1 nombre "Samuel"` | Perfil de usuario |
| `HGET key campo` | Obtiene valor de un campo | `HGET user:1 nombre` | Obtener nombre del perfil |
| `HGETALL key` | Devuelve todos los campos | `HGETALL user:1` | Mostrar perfil completo |
| `HDEL key campo` | Borra un campo | `HDEL user:1 nombre` | Borrar dato específico |
| `HEXISTS key campo` | Verifica si existe | `HEXISTS user:1 email` | Validar si hay correo |

## Conjuntos (Sets)

| Comando | Descripción | Ejemplo | Uso típico |
|---------|-------------|---------|------------|
| `SADD key valor` | Agrega elemento único | `SADD tags "redis"` | Lista de etiquetas sin duplicados |
| `SMEMBERS key` | Lista elementos | `SMEMBERS tags` | Ver todas las etiquetas |
| `SISMEMBER key valor` | Verifica existencia | `SISMEMBER tags "docker"` | Saber si un tag existe |
| `SREM key valor` | Elimina valor | `SREM tags "redis"` | Quitar una etiqueta |
| `SUNION key1 key2` | Unión de conjuntos | `SUNION tags1 tags2` | Combinar etiquetas de artículos |

## Ejemplos de Uso

### 1. Manejo de Sesiones de Usuario
```bash
# Establecer sesión de usuario con expiración
SET session:abc123 "user123"
EXPIRE session:abc123 3600  # Expira en 1 hora

# Verificar sesión
GET session:abc123
TTL session:abc123  # Ver tiempo restante
```

### 2. Sistema de Tareas Pendientes
```bash
# Agregar tareas
LPUSH tareas:usuario1 "Revisar correo"
LPUSH tareas:usuario1 "Hacer compras"

# Ver tareas
LRANGE tareas:usuario1 0 -1

# Completar tarea (procesar la más reciente)
LPOP tareas:usuario1
```

### 3. Perfil de Usuario con Hashes
```bash
# Crear perfil
HSET usuario:1001 nombre "Ana García" email "ana@ejemplo.com" edad 28

# Actualizar campo
HSET usuario:1001 edad 29

# Obtener perfil completo
HGETALL usuario:1001
```

### 4. Sistema de Etiquetas
```bash
# Añadir etiquetas a un artículo
SADD articulo:42:tags "tecnologia" "programación" "redis"

# Buscar artículos por etiqueta
SADD busqueda:tecnologia articulo:42
SADD busqueda:programacion articulo:42

# Encontrar artículos con múltiples etiquetas
SINTER busqueda:tecnologia busqueda:programacion
```
