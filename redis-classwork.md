# Ejercicios Prácticos de Redis

## Nivel Básico

### 1. Sistema de Caché Simple
**Objetivo**: Implementar un sistema de caché para resultados de consultas costosas.

```bash
# 1. Almacena en caché el resultado de una consulta a la base de datos
SET cache:usuario:1001 '{"id": 1001, "nombre": "Ana García", "puntos": 1500}'
EXPIRE cache:usuario:1001 300  # Expira en 5 minutos

# 2. Recupera los datos del caché
GET cache:usuario:1001

# 3. Verifica cuánto tiempo queda para que expire la clave
TTL cache:usuario:1001
```

### 2. Contador de Visitas
**Objetivo**: Implementar un contador de visitas para páginas web.

```bash
# 1. Incrementa el contador de visitas para la página de inicio
INCR visitas:pagina:inicio

# 2. Incrementa en 5 las visitas
INCRBY visitas:pagina:inicio 5

# 3. Obtén el total de visitas
GET visitas:pagina:inicio
```

## Nivel Intermedio

### 3. Sistema de Votos
**Objetivo**: Implementar un sistema de votación para publicaciones.

```bash
# 1. Un usuario vota por una publicación
SADD votos:post:42 usuario123

# 2. Verifica si un usuario ya votó
SISMEMBER votos:post:42 usuario123

# 3. Cuenta el total de votos
SCARD votos:post:42

# 4. Lista todos los usuarios que votaron
SMEMBERS votos:post:42
```

### 4. Almacenamiento de Perfiles de Usuario
**Objetivo**: Almacenar y actualizar perfiles de usuario.

```bash
# 1. Crea un perfil de usuario
HMSET usuario:1001 nombre "Carlos" email "carlos@ejemplo.com" edad 30

# 2. Actualiza el email
HSET usuario:1001 email "carlos.nuevo@ejemplo.com"

# 3. Obtén todos los campos del perfil
HGETALL usuario:1001

# 4. Incrementa la edad en 1
HINCRBY usuario:1001 edad 1
```

## Nivel Avanzado

### 5. Sistema de Mensajería
**Objetivo**: Implementar un sistema de mensajería simple.

```bash
# 1. Usuario1 envía mensaje a Usuario2
LPUSH mensajes:usuario2 '{"de": "usuario1", "texto": "Hola, ¿cómo estás?", "fecha": "2025-10-02T12:00:00"}'

# 2. Usuario2 lee sus mensajes
LRANGE mensajes:usuario2 0 10

# 3. Usuario2 lee solo mensajes no leídos (usando índices)
# Guardar el índice del último mensaje leído
SET ultimo_leido:usuario2 5
# Obtener mensajes nuevos
LRANGE mensajes:usuario2 0 -6
```

### 6. Sistema de Recomendaciones
**Objetivo**: Implementar un sistema básico de recomendaciones basado en intereses comunes.

```bash
# 1. Usuarios con sus intereses
SADD intereses:usuario1 "programación" "tecnología" "videojuegos"
SADD intereses:usuario2 "programación" "música"
SADD intereses:usuario3 "deportes" "música"

# 2. Encontrar intereses en común entre usuario1 y usuario2
SINTER intereses:usuario1 intereses:usuario2

# 3. Recomendar usuarios con intereses similares
SUNIONSTORE recomendaciones:usuario1 intereses:usuario2 intereses:usuario3
SDIFFSTORE recomendaciones:usuario1 recomendaciones:usuario1 intereses:usuario1
SMEMBERS recomendaciones:usuario1
```

## Ejercicios de Desafío

### 7. Rate Limiting
**Objetivo**: Implementar un limitador de peticiones por IP.

```bash
# 1. Incrementar el contador para una IP
INCR rate_limit:192.168.1.1

# 2. Establecer expiración (solo la primera vez)
EXPIRE rate_limit:192.168.1.1 60  # 1 minuto

# 3. Verificar si se ha excedido el límite (ej: 10 peticiones por minuto)
GET rate_limit:192.168.1.1
# Si el valor es mayor que 10, rechazar la petición
```

### 8. Sistema de Seguimiento de Eventos
**Objetivo**: Rastrear eventos de usuario en tiempo real.

```bash
# 1. Registrar un evento de usuario
ZADD eventos:usuario123 1643727600 "{"tipo": "login", "ip": "192.168.1.1"}"
ZADD eventos:usuario123 1643727615 "{"tipo": "busqueda", "termino": "redis"}"

# 2. Obtener los últimos 5 eventos
ZREVRANGE eventos:usuario123 0 4 WITHSCORES

# 3. Contar eventos en la última hora
ZCOUNT eventos:usuario123 1643724000 1643727600
```
