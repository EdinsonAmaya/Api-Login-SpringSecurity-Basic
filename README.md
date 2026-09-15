# API Prueba Login

API REST para un sistema de autenticación y autorización básico construido con **Spring Boot**, **Spring Security** y **JWT (JSON Web Token)**. Permite el registro de usuarios, inicio de sesion y acceso a endpoints protegidos mediante tokens JWT.

---

## Stack Tecnologico

| Tecnologia | Version | Descripcion |
|---|---|---|
| Java | 17 | Lenguaje de programacion |
| Spring Boot | 4.1.1 | Framework principal |
| Spring Security | - | Autenticacion y autorizacion |
| Spring Data JPA | - | Acceso a base de datos |
| MySQL | - | Base de datos relacional |
| JJWT | 0.12.6 | Generacion y validacion de JWT |
| Lombok | - | Reduccion de codigo boilerplate |
| Maven | - | Herramienta de construccion |

---

## Estructura del Proyecto

```
api-prueba-login/
├── src/main/java/com/edinsonAmaya/api_prueba_login/
│   ├── ApiPruebaLoginApplication.java          # Punto de entrada
│   ├── auth/
│   │   ├── AuthController.java                 # Endpoints de autenticacion
│   │   ├── AuthService.java                    # Logica de negocio de auth
│   │   ├── AuthResponse.java                   # DTO - respuesta con token
│   │   ├── LoginRequest.java                   # DTO - solicitud de login
│   │   └── RegisterRequest.java                # DTO - solicitud de registro
│   ├── config/
│   │   ├── ApplicationConfig.java              # Configuracion de beans de seguridad
│   │   └── SecurityConfig.java                 # Cadena de filtros de seguridad
│   ├── demo/
│   │   └── DemoController.java                 # Endpoint de prueba protegido
│   ├── jwt/
│   │   ├── JwtAuthenticationFilter.java        # Filtro JWT para cada peticion
│   │   └── JwtService.java                     # Servicio de creacion y validacion JWT
│   └── user/
│       ├── Role.java                           # Enum de roles (ADMIN, USER)
│       ├── UserRepository.java                 # Repositorio JPA
│       └── Users.java                          # Entidad de usuario
└── src/main/resources/
    └── application.properties                  # Configuracion de la aplicacion
```

---

## Configuracion

### Base de Datos

El proyecto utiliza MySQL. Configura las propiedades en `application.properties`:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/securitydb?useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=
spring.jpa.hibernate.ddl-auto=update
```

Crea la base de datos `securitydb` en MySQL antes de ejecutar la aplicacion:

```sql
CREATE DATABASE securitydb;
```

### Ejecucion

```bash
./mvnw spring-boot:run
```

La aplicacion estara disponible en `http://localhost:8080`.

---

## Endpoints

### Autenticacion (Publicos)

#### Registro de Usuario

```
POST /auth/register
```

**Request Body:**

```json
{
  "username": "usuario1",
  "password": "contrasena123",
  "firstname": "Edinson",
  "lastname": "Amaya",
  "country": "Colombia"
}
```

**Response (200):**

```json
{
  "token": "eyJhbGciOiJIUzI1NiJ9..."
}
```

> **Nota:** El usuario se registra con el rol `ROLE_USER` por defecto. La contrasena se almacena en formato BCrypt.

---

#### Inicio de Sesion

```
POST /auth/login
```

**Request Body:**

```json
{
  "username": "usuario1",
  "password": "contrasena123"
}
```

**Response (200):**

```json
{
  "token": "eyJhbGciOiJIUzI1NiJ9..."
}
```

---

### Endpoints Protegidos (Requieren JWT)

#### Demo

```
POST /api/v1/demo
```

**Headers:**

```
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

**Response (200):**

```
welcome from secure endpoint
```

> Cualquier peticion a un endpoint que no sea `/auth/**` requiere un token JWT valido en el header `Authorization`.

---

## Flujo de Autenticacion

### Flujo de Registro

```
Cliente
  │
  ▼
POST /auth/register (username, password, datos personales)
  │
  ▼
AuthService.register()
  ├── Cifrar contrasena con BCrypt
  ├── Asignar rol ROLE_USER
  ├── Guardar usuario en MySQL
  └── Generar JWT con el username como subject
  │
  ▼
Respuesta: { "token": "eyJ..." }
```

### Flujo de Login

```
Cliente
  │
  ▼
POST /auth/login (username, password)
  │
  ▼
AuthenticationManager.authenticate()
  ├── Buscar usuario en MySQL por username
  ├── Verificar contrasena con BCrypt
  └── Si es correcto → Generar JWT
  │
  ▼
Respuesta: { "token": "eyJ..." }
```

### Flujo de Peticion Protegida

```
Cliente
  │
  ▼
POST /api/v1/demo con header "Authorization: Bearer eyJ..."
  │
  ▼
JwtAuthenticationFilter
  ├── Extraer token del header Authorization
  ├── Parsear y validar el token
  ├── Cargar usuario desde la base de datos
  └── Si el token es valido → Autenticar el usuario en el contexto de seguridad
  │
  ▼
DemoController
  └── Retorna "welcome from secure endpoint"
```

---

## JWT (JSON Web Token)

| Propiedad | Valor |
|---|---|
| Algoritmo | HMAC-SHA256 (HS256) |
| Claims | `sub` (username), `iat` (fecha de emision), `exp` (fecha de expiracion) |
| Expiracion | 24 minutos |
| Libreria | JJWT 0.12.6 |

El token se genera al registrar o iniciar sesion y debe enviarse en cada peticion a endpoints protegidos dentro del header `Authorization` con el prefijo `Bearer`.

---

## Modelo de Datos

### Tabla `users`

| Campo | Tipo | Restricciones | Descripcion |
|---|---|---|---|
| `id` | BIGINT | PK, auto-incremental | Identificador unico |
| `username` | VARCHAR | NOT NULL, unico | Nombre de usuario |
| `password` | VARCHAR | | Contrasena cifrada (BCrypt) |
| `firstname` | VARCHAR | | Nombre |
| `lastname` | VARCHAR | | Apellido |
| `country` | VARCHAR | | Pais |
| `role` | VARCHAR | | Rol (ROLE_ADMIN o ROLE_USER) |

---

## Roles

| Rol | Descripcion |
|---|---|
| `ROLE_ADMIN` | Administrador del sistema |
| `ROLE_USER` | Usuario regular |

> **Nota:** Actualmente todos los usuarios se registran con el rol `ROLE_USER`. El campo `role` esta almacenado en la base de datos pero no se aplica autorizacion basada en roles en los endpoints.

---

## Configuracion de Seguridad

- **CSRF:** Deshabilitado (API stateless)
- **Sesiones:** Deshabilitadas (estado manejado exclusivamente via JWT)
- **Endpoints publicos:** Todo bajo `/auth/**`
- **Endpoints protegidos:** Todo lo que no coincida con `/auth/**`
- **Filtro JWT:** Se ejecuta antes de `UsernamePasswordAuthenticationFilter` en cada peticion
- **Cifrado de contrasena:** BCrypt (`PasswordEncoder`)

---

## Requisitos

- Java 17 o superior
- MySQL 8.x o superior
- Maven 3.x (o usar el wrapper incluido `./mvnw`)
