# SIGCB-QR - Sistema Integral de GestiOn, Control y Seguimiento de Lectura Interna

Backend del sistema de biblioteca universitaria con autenticacion JWT, Spring Data JPA y PostgreSQL 16.

## Tecnologias

- **Backend:** Spring Boot 3.2 + Java 17
- **Auth:** Spring Security 6 + JWT (jjwt 0.12, HS256)
- **Base de datos:** PostgreSQL 16 con Flyway migrations
- **Cache/Blacklist:** Redis 7
- **Frontend:** Angular + Nginx
- **Tests:** JUnit 5 + MockMvc
- **Infraestructura:** Docker Compose

## Requisitos

- Docker y Docker Compose
- Java 17+ (para desarrollo local)
- Maven 3.9+ (para desarrollo local)

## Ejecucion con Docker (5 pasos)

### Paso 1: Clonar el repositorio

```bash
git clone <repo-url>
cd biblioteca-api
```

### Paso 2: Construir la aplicacion

```bash
mvn clean package -DskipTests
```

### Paso 3: Iniciar los servicios con Docker Compose

```bash
docker compose up --build -d
```

### Paso 4: Verificar que todos los servicios esten activos

```bash
docker compose ps
```

### Paso 5: Probar la API

```bash
# Health check
curl http://localhost:8080/api/health

# Registrar usuario
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"cedula":"1234567890","nombre":"Ana","apellido":"Lopez","correo":"ana@uteq.edu.ec","password":"pass1234","rol":"ESTUDIANTE"}'

# Iniciar sesion
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"correo":"ana@uteq.edu.ec","password":"pass1234"}'
```

El frontend estara disponible en: http://localhost

## Estructura del proyecto

```
biblioteca-api/
  src/
    main/java/com/biblioteca/
      config/         # Configuraciones (Security, CORS, Redis)
      security/       # JWT, filtros, blacklist
      entity/         # Entidades JPA
      dto/            # DTOs de entrada/salida
      repository/     # Spring Data JPA repositorios
      service/        # Logica de negocio
      controller/     # Controladores REST
    main/resources/
      db/migration/   # Migraciones Flyway
      application.yml # Configuracion principal
  frontend/           # Frontend estatico (Nginx)
  Dockerfile          # Imagen del backend
  docker-compose.yml  # Orquestacion de servicios
```

## Endpoints de la API

### Autenticacion
| Metodo | Ruta | Descripcion |
|--------|------|-------------|
| POST | /api/auth/register | Registrar nuevo usuario |
| POST | /api/auth/login | Iniciar sesion |
| POST | /api/auth/logout | Cerrar sesion (revoca token) |
| POST | /api/auth/refresh | Refrescar token |

### Material (CRUD)
| Metodo | Ruta | Descripcion |
|--------|------|-------------|
| GET | /api/material | Listar con paginacion |
| GET | /api/material/{id} | Obtener por ID |
| GET | /api/material/buscar/titulo?q= | Buscar por titulo |
| GET | /api/material/buscar/categoria?q= | Buscar por categoria |
| GET | /api/material/disponibles | Materiales disponibles |
| POST | /api/material | Crear (BIBLIOTECARIO/ADMIN) |
| PUT | /api/material/{id} | Actualizar (BIBLIOTECARIO/ADMIN) |
| DELETE | /api/material/{id} | Eliminar (ADMIN) |

### Salud
| Metodo | Ruta | Descripcion |
|--------|------|-------------|
| GET | /api/health | Health check del sistema |

## Seguridad OWASP implementada

1. **BCrypt costo 12** - PasswordEncoder con strength 12
2. **JWT de vida corta** - Access token: 1 hora, Refresh token: 7 dias
3. **Blacklist Redis** - JTI revocados con TTL igual a expiracion del token
4. **Cabeceras HTTP** - X-Content-Type-Options, X-Frame-Options, CSP
5. **CORS explicito** - Solo origenes permitidos configurados
6. **Roles con @PreAuthorize** - Control de acceso por metodo

## Migraciones Flyway

Las migraciones se ejecutan automaticamente al iniciar la aplicacion:

- `V1__schema_inicial.sql` - Creacion de tablas, indices, constraints y triggers
