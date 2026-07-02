# Modelo ER, Relacional y Normalización - Proyecto (Sistema de Gestión de Citas para Hotel)

**Autores:** Angel Uriel Monterrosas Gonzalez - Noe Tlachi Zenteno  
**Matrícula:** 202339872 - 202162606  
**Institución:** Benemérita Universidad Autónoma de Puebla (BUAP)  
**Carrera:** Licenciatura en Ingeniería en Ciencias de la Computación  
**Materia:** Base de Datos para Ingeniería  
**Profesora:** Beltran Martínez Beatriz  
**Fecha:** 16 de junio de 2026

---

## 1. Descripción del Proyecto (Análisis de Requerimientos)
# Descripción del Proyecto (Análisis de Requerimientos)

El objetivo es diseñar una base de datos relacional robusta para gestionar las reservas de un hotel. El sistema abarca la gestión de clientes, catálogo de habitaciones con tarifas dinámicas por temporada, control de inventario, registro de servicios adicionales, pagos y un historial de auditoría. Se incluye un esquema de roles (Administrador y Recepcionista) para controlar el acceso a las funcionalidades.

## Requerimientos Funcionales (RF)

- **RF1 - Gestión de clientes:** Registro, modificación y borrado lógico de huéspedes.
- **RF2 - Catálogo de tipos de habitación:** Definición de categorías (simple, doble, suite) y tarifas estacionales (alta/baja).
- **RF3 - Inventario de habitaciones:** Control de estado (disponible, ocupada, en mantenimiento) por cada unidad física.
- **RF4 - Creación de reservas:** Asignación de fechas sin solapamientos, vinculando clientes y habitaciones. Además, al momento de crear la reserva se debe almacenar una copia de los datos del titular (nombre, apellidos, teléfono, correo) para conservar la información histórica.
- **RF5 - Cancelación de reservas:** Liberación de habitaciones y aplicación de políticas de cancelación.
- **RF6 - Servicios adicionales:** Inclusión de extras (desayuno, spa) con congelamiento de precios al momento de la reserva.
- **RF7 - Registro de pagos:** Control de abonos totales o parciales indicando método de pago.
- **RF8 - Consulta de disponibilidad:** Búsqueda ágil de habitaciones libres por rango de fechas.
- **RF9 - Facturación al checkout:** Cálculo automático de saldo pendiente (noches + servicios - abonos).
- **RF10 - Historial de cambios:** Auditoría para registrar qué usuario modificó el estado de una reserva y cuándo.

## Requerimientos No Funcionales Clave (RNF)

- **RNF3 - Consistencia:** Prevención a nivel de base de datos de dobles reservas y fechas ilógicas (salida antes de entrada).
- **RNF5 - Seguridad:** Protección de datos financieros, guardando solo el método de pago sin datos sensibles de tarjetas. Además, las contraseñas de los usuarios se almacenan únicamente mediante un hash criptográfico (bcrypt/Argon2) para garantizar su confidencialidad.
- **RNF6 - Escalabilidad:** Diseño normalizado que permite incorporar nuevas sucursales, habitaciones o servicios sin alterar el esquema base.

## Roles y Permisos

Se definen dos roles:

### Administrador (ADMIN)

- Acceso total a todas las funciones del sistema.
- Gestión de usuarios (crear, modificar, eliminar) y asignación de roles.
- Gestión de catálogos (tipos de habitación, tarifas, servicios).
- Consulta y modificación de cualquier reserva.
- Auditoría completa (historial).

### Recepcionista (RECEPCIONISTA)

- Gestión de clientes (crear, modificar, consultar).
- Creación y cancelación de reservas (sobre las habitaciones disponibles).
- Registro de pagos.
- Consulta de disponibilidad.
- No puede modificar tarifas, catálogos ni usuarios.

## 2. Evolución del Modelo y Normalización

Para garantizar la integridad referencial, evitar anomalías de actualización y eliminar redundancias, el modelo conceptual inicial fue sometido a un proceso de normalización hasta alcanzar la **Tercera Forma Normal (3NF)**.

### 2.1 Modelo Entidad-Relación Inicial (Antes)

En la fase conceptual, se identifican las entidades principales y las reglas de negocio. En este punto, existen relaciones de muchos a muchos (M:N), como la de `RESERVAS` y `SERVICIOS`.

```mermaid
erDiagram
    USUARIOS {
        int id_usuario PK
        string nombre_usuario UK
        string nombre
        string apellido_paterno
        string apellido_materno
        string contrasena_hash
        enum rol "ADMIN|RECEPCIONISTA"
    }
    CLIENTES {
        int id_cliente PK
        string nombre
        string apellido_paterno
        string apellido_materno
        string telefono
        string correo UK
        bool activo
    }
    RESERVAS {
        int id_reserva PK
        int id_cliente FK
        int id_habitacion FK
        date fecha_entrada
        date fecha_salida
        enum estado_reserva "ACTIVA|CONFIRMADA|CANCELADA|COMPLETADA"
        datetime fecha_creacion
        decimal penalizacion
        datetime fecha_cancelacion
        string nombre_titular
        string apellido_paterno_titular
        string apellido_materno_titular
        string telefono_titular
        string correo_titular
    }
    TIPOS_HABITACION {
        int id_tipo_habitacion PK
        string nombre
        int capacidad
        text descripcion
    }
    TARIFAS_TEMPORADA {
        int id_tarifa PK
        int id_tipo_habitacion FK
        enum temporada "ALTA|BAJA"
        decimal precio_noche
        date fecha_inicio
        date fecha_fin
    }
    HABITACIONES {
        int id_habitacion PK
        int id_tipo_habitacion FK
        int piso
        enum estado_actual "DISPONIBLE|OCUPADA|MANTENIMIENTO"
        bool activa
    }
    SERVICIOS {
        int id_servicio PK
        string nombre
        decimal precio_vigente
        text descripcion
    }
    RESERVA_SERVICIOS {
        int id_reserva_servicio PK
        int id_reserva FK
        int id_servicio FK
        int cantidad
        decimal precio_unitario
    }
    PAGOS {
        int id_pago PK
        int id_reserva FK
        decimal monto
        datetime fecha_pago
        enum metodo_pago "EFECTIVO|TARJETA|TRANSFERENCIA"
    }
    HISTORIAL_RESERVAS {
        int id_historial PK
        int id_reserva FK
        int id_usuario FK
        string estado_anterior
        string estado_nuevo
        datetime fecha_cambio
        text comentario
    }

    USUARIOS ||--o{ HISTORIAL_RESERVAS : "registra"
    CLIENTES ||--o{ RESERVAS : "realiza"
    TIPOS_HABITACION ||--o{ TARIFAS_TEMPORADA : "tiene"
    TIPOS_HABITACION ||--o{ HABITACIONES : "clasifica"
    HABITACIONES ||--o{ RESERVAS : "asignada"
    RESERVAS ||--o{ RESERVA_SERVICIOS : "incluye"
    SERVICIOS ||--o{ RESERVA_SERVICIOS : "ofrece"
    RESERVAS ||--o{ PAGOS : "registra"
    RESERVAS ||--o{ HISTORIAL_RESERVAS : "genera"
```


### 2.2 Modelo Relacional Normalizado (Después)

El siguiente diagrama de clases muestra el modelo relacional con todas las tablas, sus claves primarias (PK) y foráneas (FK), garantizando que cada tabla cumple con 3NF (atributos atómicos, sin dependencias parciales ni transitivas).

```mermaid
classDiagram
    class usuarios {
        +int id_usuario PK
        +string nombre_usuario UK
        +string nombre
        +string apellido_paterno
        +string apellido_materno
        +string contrasena_hash
        +enum rol
    }
    class clientes {
        +int id_cliente PK
        +string nombre
        +string apellido_paterno
        +string apellido_materno
        +string telefono
        +string correo UK
        +bool activo
    }
    class reservas {
        +int id_reserva PK
        +int id_cliente FK
        +int id_habitacion FK
        +date fecha_entrada
        +date fecha_salida
        +enum estado_reserva
        +datetime fecha_creacion
        +decimal penalizacion
        +datetime fecha_cancelacion
        +string nombre_titular
        +string apellido_paterno_titular
        +string apellido_materno_titular
        +string telefono_titular
        +string correo_titular
    }
    class tipos_habitacion {
        +int id_tipo_habitacion PK
        +string nombre
        +int capacidad
        +text descripcion
    }
    class tarifas_temporada {
        +int id_tarifa PK
        +int id_tipo_habitacion FK
        +enum temporada
        +decimal precio_noche
        +date fecha_inicio
        +date fecha_fin
    }
    class habitaciones {
        +int id_habitacion PK
        +int id_tipo_habitacion FK
        +int piso
        +enum estado_actual
        +bool activa
    }
    class servicios {
        +int id_servicio PK
        +string nombre
        +decimal precio_vigente
        +text descripcion
    }
    class reserva_servicios {
        +int id_reserva_servicio PK
        +int id_reserva FK
        +int id_servicio FK
        +int cantidad
        +decimal precio_unitario
    }
    class pagos {
        +int id_pago PK
        +int id_reserva FK
        +decimal monto
        +datetime fecha_pago
        +enum metodo_pago
    }
    class historial_reservas {
        +int id_historial PK
        +int id_reserva FK
        +int id_usuario FK
        +string estado_anterior
        +string estado_nuevo
        +datetime fecha_cambio
        +text comentario
    }

    usuarios "1" --> "0..*" historial_reservas : "id_usuario"
    clientes "1" --> "0..*" reservas : "id_cliente"
    tipos_habitacion "1" --> "0..*" tarifas_temporada : "id_tipo_habitacion"
    tipos_habitacion "1" --> "0..*" habitaciones : "id_tipo_habitacion"
    habitaciones "1" --> "0..*" reservas : "id_habitacion"
    reservas "1" --> "0..*" reserva_servicios : "id_reserva"
    servicios "1" --> "0..*" reserva_servicios : "id_servicio"
    reservas "1" --> "0..*" pagos : "id_reserva"
    reservas "1" --> "0..*" historial_reservas : "id_reserva"
```

---
## Observaciones

- Todas las tablas tienen una clave primaria simple y autoincrementable.
- Las claves foráneas garantizan la integridad referencial con eliminación en cascada (ON DELETE CASCADE).
- No existen dependencias transitivas: los atributos no clave dependen directamente de la clave primaria de su tabla.
- Los campos como `nombre_titular`, etc., en reservas son copias históricas, no dependen de clientes para preservar la información.
## 3. Diccionario de Datos (Modelo Normalizado 3NF)

A continuación, se define la estructura técnica de cada entidad en su estado final.

## Diccionario de Datos

| Tabla | Descripción | Permisos |
|-------|-------------|----------|
| **usuarios** | Personal que gestiona el sistema. | ADMIN o RECEPCIONISTA |

| Columna | Tipo | Nulabilidad | Descripción |
|---------|------|-------------|-------------|
| `id_usuario` | INT | NO NULO | PK, autoincrementable |
| `nombre_usuario` | VARCHAR(50) | NO NULO | ÚNICO |
| `nombre` | VARCHAR(50) | NO NULO | - |
| `apellido_paterno` | VARCHAR(50) | NO NULO | - |
| `apellido_materno` | VARCHAR(50) | NULO | - |
| `contrasena_hash` | VARCHAR(255) | NO NULO | Almacena el hash de la contraseña |
| `rol` | ENUM('ADMIN','RECEPCIONISTA') | NO NULO | - |

---

| Tabla | Descripción |
|-------|-------------|
| **clientes** | Datos de los huéspedes. Borrado lógico con `activo`. |

| Columna | Tipo | Nulabilidad | Descripción |
|---------|------|-------------|-------------|
| `id_cliente` | INT | NO NULO | PK, autoincrementable |
| `nombre` | VARCHAR(50) | NO NULO | - |
| `apellido_paterno` | VARCHAR(50) | NO NULO | - |
| `apellido_materno` | VARCHAR(50) | NULO | - |
| `telefono` | VARCHAR(20) | NULO | - |
| `correo` | VARCHAR(100) | NULO | ÚNICO |
| `activo` | BOOLEAN | NO NULO | DEFAULT TRUE |

---

| Tabla | Descripción |
|-------|-------------|
| **tipos_habitacion** | Catálogo de categorías de habitaciones. |

| Columna | Tipo | Nulabilidad | Descripción |
|---------|------|-------------|-------------|
| `id_tipo_habitacion` | INT | NO NULO | PK, autoincrementable |
| `nombre` | VARCHAR(50) | NO NULO | - |
| `capacidad` | INT | NO NULO | - |
| `descripcion` | TEXT | NULO | - |

---

| Tabla | Descripción |
|-------|-------------|
| **tarifas_temporada** | Precios por tipo de habitación y temporada. |

| Columna | Tipo | Nulabilidad | Descripción |
|---------|------|-------------|-------------|
| `id_tarifa` | INT | NO NULO | PK, autoincrementable |
| `id_tipo_habitacion` | INT | NO NULO | FK |
| `temporada` | ENUM('ALTA','BAJA') | NO NULO | - |
| `precio_noche` | DECIMAL(10,2) | NO NULO | - |
| `fecha_inicio` | DATE | NO NULO | - |
| `fecha_fin` | DATE | NO NULO | - |

---

| Tabla | Descripción |
|-------|-------------|
| **habitaciones** | Registro físico de cada habitación. |

| Columna | Tipo | Nulabilidad | Descripción |
|---------|------|-------------|-------------|
| `id_habitacion` | INT | NO NULO | PK, autoincrementable |
| `id_tipo_habitacion` | INT | NO NULO | FK |
| `piso` | INT | NO NULO | - |
| `estado_actual` | ENUM('DISPONIBLE','OCUPADA','MANTENIMIENTO') | NO NULO | DEFAULT 'DISPONIBLE' |
| `activa` | BOOLEAN | NO NULO | DEFAULT TRUE |

---

| Tabla | Descripción |
|-------|-------------|
| **reservas** | Entidad central que vincula cliente, habitación y fechas. Incluye instantánea del titular. |

| Columna | Tipo | Nulabilidad | Descripción |
|---------|------|-------------|-------------|
| `id_reserva` | INT | NO NULO | PK, autoincrementable |
| `id_cliente` | INT | NO NULO | FK |
| `id_habitacion` | INT | NO NULO | FK |
| `fecha_entrada` | DATE | NO NULO | - |
| `fecha_salida` | DATE | NO NULO | - |
| `estado_reserva` | ENUM('ACTIVA','CONFIRMADA','CANCELADA','COMPLETADA') | NO NULO | DEFAULT 'ACTIVA' |
| `fecha_creacion` | DATETIME | NO NULO | DEFAULT CURRENT_TIMESTAMP |
| `penalizacion` | DECIMAL(10,2) | NULO | - |
| `fecha_cancelacion` | DATETIME | NULO | - |
| `nombre_titular` | VARCHAR(50) | NO NULO | Copia histórica |
| `apellido_paterno_titular` | VARCHAR(50) | NO NULO | Copia histórica |
| `apellido_materno_titular` | VARCHAR(50) | NULO | Copia histórica |
| `telefono_titular` | VARCHAR(20) | NULO | Copia histórica |
| `correo_titular` | VARCHAR(100) | NULO | Copia histórica |

---

| Tabla | Descripción |
|-------|-------------|
| **servicios** | Catálogo de servicios extras. |

| Columna | Tipo | Nulabilidad | Descripción |
|---------|------|-------------|-------------|
| `id_servicio` | INT | NO NULO | PK, autoincrementable |
| `nombre` | VARCHAR(100) | NO NULO | - |
| `precio_vigente` | DECIMAL(10,2) | NO NULO | - |
| `descripcion` | TEXT | NULO | - |

---

| Tabla | Descripción |
|-------|-------------|
| **reserva_servicios** | Tabla puente para servicios consumidos, con precio congelado. |

| Columna | Tipo | Nulabilidad | Descripción |
|---------|------|-------------|-------------|
| `id_reserva_servicio` | INT | NO NULO | PK, autoincrementable |
| `id_reserva` | INT | NO NULO | FK |
| `id_servicio` | INT | NO NULO | FK |
| `cantidad` | INT | NO NULO | - |
| `precio_unitario` | DECIMAL(10,2) | NO NULO | Precio congelado al momento de la reserva |

---

| Tabla | Descripción |
|-------|-------------|
| **pagos** | Registro de abonos a una reserva. |

| Columna | Tipo | Nulabilidad | Descripción |
|---------|------|-------------|-------------|
| `id_pago` | INT | NO NULO | PK, autoincrementable |
| `id_reserva` | INT | NO NULO | FK |
| `monto` | DECIMAL(10,2) | NO NULO | - |
| `fecha_pago` | DATETIME | NO NULO | DEFAULT CURRENT_TIMESTAMP |
| `metodo_pago` | ENUM('EFECTIVO','TARJETA','TRANSFERENCIA') | NO NULO | - |

---

| Tabla | Descripción |
|-------|-------------|
| **historial_reservas** | Bitácora de auditoría de cambios de estado. |

| Columna | Tipo | Nulabilidad | Descripción |
|---------|------|-------------|-------------|
| `id_historial` | INT | NO NULO | PK, autoincrementable |
| `id_reserva` | INT | NO NULO | FK |
| `id_usuario` | INT | NO NULO | FK |
| `estado_anterior` | VARCHAR(20) | NO NULO | - |
| `estado_nuevo` | VARCHAR(20) | NO NULO | - |
| `fecha_cambio` | DATETIME | NO NULO | DEFAULT CURRENT_TIMESTAMP |
| `comentario` | TEXT | NULO | - |
---

## 4. Script de Base de Datos (SQL DDL)

El siguiente script crea la base de datos y todas las tablas respetando el orden de dependencias para garantizar la integridad referencial de las claves foráneas.


```sql
-- =============================================================================
-- SCRIPT DE CREACIÓN DE BASE DE DATOS: SISTEMA DE GESTIÓN HOTELERA (BUAP)
-- =============================================================================
DROP DATABASE IF EXISTS hotel_gestion;
CREATE DATABASE hotel_gestion CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE hotel_gestion;

-- 1. Tabla de Roles para el Personal
CREATE TABLE roles (
id_rol INT AUTO_INCREMENT PRIMARY KEY,
nombre_rol ENUM('ADMIN', 'RECEPCIONISTA') NOT NULL,
descripcion VARCHAR(255)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 2. Tabla de Usuarios (Personal del Hotel)
CREATE TABLE usuarios (
id_usuario INT AUTO_INCREMENT PRIMARY KEY,
id_rol INT NOT NULL,
nombre VARCHAR(50) NOT NULL,
apellido_paterno VARCHAR(50) NOT NULL,
apellido_materno VARCHAR(50),
correo VARCHAR(100) NOT NULL UNIQUE,
password_hash VARCHAR(255) NOT NULL,
activo BOOLEAN NOT NULL DEFAULT TRUE,
fecha_creacion DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
FOREIGN KEY (id_rol) REFERENCES roles(id_rol) ON DELETE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 3. Tabla de Clientes (Huéspedes)
CREATE TABLE clientes (
id_cliente INT AUTO_INCREMENT PRIMARY KEY,
nombre VARCHAR(50) NOT NULL,
apellido_paterno VARCHAR(50) NOT NULL,
apellido_materno VARCHAR(50),
telefono VARCHAR(20),
correo VARCHAR(100) NOT NULL UNIQUE,
activo BOOLEAN NOT NULL DEFAULT TRUE,
fecha_creacion DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 4. Tabla de Tipos de Habitación
CREATE TABLE tipos_habitacion (
id_tipo_habitacion INT AUTO_INCREMENT PRIMARY KEY,
nombre_tipo VARCHAR(50) NOT NULL UNIQUE, -- E.g., 'Simple', 'Doble', 'Suite'
descripcion TEXT,
capacidad_max INT NOT NULL CONSTRAINT chk_capacidad CHECK (capacidad_max > 0)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 5. Tabla de Tarifas por Temporada
CREATE TABLE tarifas_temporada (
id_tarifa INT AUTO_INCREMENT PRIMARY KEY,
id_tipo_habitacion INT NOT NULL,
nombre_temporada VARCHAR(50) NOT NULL, -- E.g., 'Alta Verano', 'Baja Invierno'
precio_noche DECIMAL(10,2) NOT NULL CONSTRAINT chk_precio CHECK (precio_noche >= 0),
fecha_inicio DATE NOT NULL,
fecha_fin DATE NOT NULL,
FOREIGN KEY (id_tipo_habitacion) REFERENCES tipos_habitacion(id_tipo_habitacion) ON DELETE CASCADE,
CONSTRAINT chk_fechas_tarifa CHECK (fecha_fin >= fecha_inicio)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 6. Tabla de Habitaciones (ID Manual para número físico)
CREATE TABLE habitaciones (
id_habitacion INT PRIMARY KEY, -- ID Manual (Ej. 101, 102, 201)
id_tipo_habitacion INT NOT NULL,
estado_actual ENUM('DISPONIBLE', 'OCUPADA', 'MANTENIMIENTO') NOT NULL DEFAULT 'DISPONIBLE',
activa BOOLEAN NOT NULL DEFAULT TRUE,
FOREIGN KEY (id_tipo_habitacion) REFERENCES tipos_habitacion(id_tipo_habitacion) ON DELETE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 7. Tabla Central de Reservas (Precio Congelado)
CREATE TABLE reservas (
id_reserva INT AUTO_INCREMENT PRIMARY KEY,
id_cliente INT NOT NULL,
id_habitacion INT NOT NULL,
fecha_entrada DATE NOT NULL,
fecha_salida DATE NOT NULL,
precio_noche_congelado DECIMAL(10,2) NOT NULL, -- Almacena la tarifa fija del momento
estado_reserva ENUM('ACTIVA', 'CONFIRMADA', 'CANCELADA', 'COMPLETADA') NOT NULL DEFAULT 'ACTIVA',
penalizacion DECIMAL(10,2) DEFAULT 0.00,
fecha_cancelacion DATETIME,
fecha_creacion DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
-- Instantánea histórica del cliente titular
nombre_titular VARCHAR(50) NOT NULL,
apellido_paterno_titular VARCHAR(50) NOT NULL,
apellido_materno_titular VARCHAR(50),
telefono_titular VARCHAR(20),
correo_titular VARCHAR(100),
FOREIGN KEY (id_cliente) REFERENCES clientes(id_cliente) ON DELETE RESTRICT,
FOREIGN KEY (id_habitacion) REFERENCES habitaciones(id_habitacion) ON DELETE RESTRICT,
CONSTRAINT chk_fechas_reserva CHECK (fecha_salida > fecha_entrada),
CONSTRAINT chk_precio_congelado CHECK (precio_noche_congelado >= 0)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 8. Catálogo de Servicios Adicionales
CREATE TABLE servicios_extras (
id_servicio INT AUTO_INCREMENT PRIMARY KEY,
nombre_servicio VARCHAR(50) NOT NULL UNIQUE,
descripcion VARCHAR(255),
precio_actual DECIMAL(10,2) NOT NULL CONSTRAINT chk_precio_servicio CHECK (precio_actual >= 0),
activo BOOLEAN NOT NULL DEFAULT TRUE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 9. Detalle de Servicios contratados por Reserva (Precio Congelado)
CREATE TABLE reserva_servicios (
id_reserva_servicio INT AUTO_INCREMENT PRIMARY KEY,
id_reserva INT NOT NULL,
id_servicio INT NOT NULL,
cantidad INT NOT NULL DEFAULT 1 CONSTRAINT chk_cantidad CHECK (cantidad > 0),
precio_unitario DECIMAL(10,2) NOT NULL, -- Precio congelado al consumir el servicio
fecha_consumo DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
FOREIGN KEY (id_reserva) REFERENCES reservas(id_reserva) ON DELETE CASCADE,
FOREIGN KEY (id_servicio) REFERENCES servicios_extras(id_servicio) ON DELETE RESTRICT,
CONSTRAINT chk_precio_unitario CHECK (precio_unitario >= 0)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 10. Tabla de Pagos
CREATE TABLE pagos (
id_pago INT AUTO_INCREMENT PRIMARY KEY,
id_reserva INT NOT NULL,
monto DECIMAL(10,2) NOT NULL CONSTRAINT chk_monto_pago CHECK (monto > 0),
metodo_pago ENUM('EFECTIVO', 'TARJETA_CREDITO', 'TARJETA_DEBITO', 'TRANSFERENCIA') NOT NULL,
fecha_pago DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
FOREIGN KEY (id_reserva) REFERENCES reservas(id_reserva) ON DELETE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 11. Tabla de Bitácora / Auditoría Global
CREATE TABLE auditoria (
id_auditoria INT AUTO_INCREMENT PRIMARY KEY,
id_usuario INT, -- Quién realizó la acción (NULL si es del sistema automático)
tabla_afectada VARCHAR(50) NOT NULL,
accion ENUM('INSERT', 'UPDATE', 'DELETE') NOT NULL,
descripcion_cambio TEXT NOT NULL,
fecha_accion DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
FOREIGN KEY (id_usuario) REFERENCES usuarios(id_usuario) ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- =============================================================================
-- ÍNDICES OPTIMIZADOS PARA OPERACIONES FRECUENTES
-- =============================================================================
CREATE INDEX idx_reservas_fechas ON reservas(fecha_entrada, fecha_salida, estado_reserva);
CREATE INDEX idx_tarifas_periodo ON tarifas_temporada(fecha_inicio, fecha_fin);

```
Datos de prueba:
```sql
-- ==========================================
-- 5. Datos de Prueba 
-- ==========================================

INSERT INTO tipos_habitacion (nombre, capacidad, descripcion) VALUES
('Simple', 1, 'Habitación individual con cama sencilla'),
('Doble', 2, 'Habitación con cama matrimonial o dos camas individuales'),
('Suite', 4, 'Suite con sala y recámara separada');

INSERT INTO servicios (nombre, precio_vigente, descripcion) VALUES
('Desayuno buffet', 150.00, 'Desayuno completo tipo buffet'),
('Spa', 500.00, 'Acceso al spa y masaje de 30 min'),
('Estacionamiento', 100.00, 'Estacionamiento cubierto por noche');

-- Usuarios de prueba (las contraseñas se gestionan con password_hash() en la aplicación)
-- Los hash mostrados son ilustrativos; se deben generar con password_hash('admin123', PASSWORD_BCRYPT)
INSERT INTO usuarios (nombre_usuario, nombre, apellido_paterno, apellido_materno, contrasena_hash, rol) VALUES
('admin', 'Admin', 'Principal', NULL, '$2y$10$e0MYzXjYx7ZQZJ2QZJ2QZuZQZJ2QZJ2QZJ2QZJ2QZJ2QZJ2QZJ2QZJ2', 'ADMIN'),
('recepcionista1', 'Ana', 'Pérez', 'Gómez', '$2y$10$f0FyYkYlYkYlYkYlYkYlYkYlYkYlYkYlYkYlYkYlYkYlYkYlYkYlY', 'RECEPCIONISTA');
```

## 5. Estructura de Carpetas del Proyecto
```text
proyecto-hotel/
├── backend/
│   ├── config/
│   │   └── database.php              # Configuración de conexión a BD
│   ├── models/
│   │   ├── Usuario.php
│   │   ├── Cliente.php
│   │   ├── TipoHabitacion.php        # NUEVO
│   │   ├── TarifaTemporada.php       # NUEVO
│   │   ├── Habitacion.php
│   │   ├── Reserva.php
│   │   ├── Servicio.php
│   │   ├── ReservaServicio.php       # NUEVO (tabla puente)
│   │   ├── Pago.php
│   │   └── Historial.php
│   ├── controllers/
│   │   ├── AuthController.php        # Login/Logout
│   │   ├── UsuarioController.php     # NUEVO (gestión de usuarios solo ADMIN)
│   │   ├── ClienteController.php
│   │   ├── TipoHabitacionController.php   # NUEVO (catálogo de tipos)
│   │   ├── TarifaController.php           # NUEVO (tarifas por temporada)
│   │   ├── HabitacionController.php  # Inventario físico
│   │   ├── ReservaController.php     # Creación, cancelación, checkout, historial
│   │   ├── ServicioController.php    # Catálogo de servicios
│   │   ├── PagoController.php
│   │   └── ReporteController.php
│   ├── routes/
│   │   └── api.php                   # Definición de rutas (endpoints)
│   ├── helpers/
│   │   ├── validator.php
│   │   └── auth.php                  # Middleware de autenticación y roles
│   └── middleware/
│       └── RoleMiddleware.php        # Verificación de permisos por rol
├── frontend/
│   ├── public/
│   │   ├── css/
│   │   │   └── style.css             # TODOS los estilos visuales del sistema
│   │   ├── js/
│   │   │   └── app.js                # TODAS las validaciones en cliente (JavaScript)
│   │   └── images/
│   ├── views/
│   │   ├── auth/
│   │   │   ├── login.php
│   │   │   └── logout.php
│   │   ├── dashboard/
│   │   │   └── index.php
│   │   ├── clientes/
│   │   │   ├── listar.php
│   │   │   ├── crear.php
│   │   │   └── editar.php
│   │   ├── catalogos/                     # NUEVO (módulo de catálogos)
│   │   │   ├── tipos_habitacion.php       # Listado y gestión de tipos
│   │   │   └── tarifas.php                # Gestión de tarifas por temporada
│   │   ├── habitaciones/
│   │   │   ├── disponibilidad.php
│   │   │   └── gestion.php                # Inventario físico y estado
│   │   ├── reservas/
│   │   │   ├── crear.php
│   │   │   ├── listar.php
│   │   │   ├── detalle.php                # Incluye facturación y historial
│   │   │   └── cancelar.php
│   │   ├── servicios/
│   │   │   └── catalogo.php               # Gestión de servicios adicionales
│   │   ├── pagos/
│   │   │   └── registrar.php
│   │   ├── reportes/
│   │   │   └── index.php                  # Reportes (diferenciados por rol)
│   │   ├── usuarios/                      # Solo ADMIN
│   │   │   ├── listar.php
│   │   │   ├── crear.php
│   │   │   └── editar.php                 # Incluye cambio de contraseña
│   │   ├── historial/                     # NUEVO (auditoría global, solo ADMIN)
│   │   │   └── index.php
│   │   └── layout/
│   │       ├── header.php
│   │       └── footer.php
│   └── index.php                         # Punto de entrada (redirige a login o dashboard)
├── database/
│   └── schema.sql                        # Script SQL completo (el generado arriba)
├── docs/
│   ├── manual_usuario.md
│   └── manual_tecnico.md
├── .htaccess                             # Configuración de rutas amigables
├── composer.json                         # Dependencias PHP (si aplica)
└── README.md
``` 
# Logica de Negocio y Flujos de Trabajo (Funcionalidad Detallada)

Esta seccion describe como interactuan los componentes para cumplir con los requerimientos funcionales, garantizando la consistencia de los datos y la correcta ejecucion de los procesos.

---

## 6.1. Autenticacion de Usuarios (Login)

- El formulario de login solicita `nombre_usuario` y `contrasena`.
- El backend (controlador `AuthController`) recibe las credenciales, busca al usuario por `nombre_usuario` y, si existe, verifica la contraseña usando `password_verify()` contra el hash almacenado en `contrasena_hash`.
- Si la verificacion es exitosa, se inicia la sesion y se guarda el rol del usuario para control de acceso.
- En caso de fallo, se registra el intento (opcional) y se devuelve un error generico.
- Las contraseñas nunca se almacenan en texto plano; solo el hash generado con `password_hash()` (bcrypt por defecto) se guarda en la base de datos.

---

## 6.2. Gestion de Clientes (RF1)

- El recepcionista o administrador puede dar de alta un nuevo cliente. Se validan campos (nombre, apellidos, telefono, correo) mediante JavaScript en el frontend.
- El correo debe ser unico en la tabla `clientes` (restriccion `UNIQUE`). Si se intenta duplicar, el backend devuelve un error.
- El borrado es logico: se cambia el campo `activo` a `FALSE`. Las reservas historicas permanecen, pero el cliente no aparecera en las busquedas activas.
- Al modificar un cliente, se actualiza su registro. No se afectan reservas pasadas (los datos del titular ya fueron copiados en `reservas`).

---

## 6.3. Catalogo de Tipos de Habitacion y Tarifas (RF2)

- El administrador gestiona los tipos de habitacion (`tipos_habitacion`) a traves del controlador `TipoHabitacionController` y las vistas de `catalogos/tipos_habitacion.php`.
- Las tarifas por temporada se gestionan mediante `TarifaController` y la vista `catalogos/tarifas.php`.
- Para cada tipo, se definen precios para temporada `ALTA` y `BAJA` con fechas de vigencia. El sistema valida que `fecha_fin >= fecha_inicio`.
- Al calcular el costo de una reserva, se busca la tarifa que corresponda a la `fecha_entrada` de la estancia (se toma la tarifa vigente al dia de entrada como precio fijo para toda la estancia, simplificando).
- Si un precio cambia, solo afecta a nuevas reservas, no a las ya creadas (porque el precio se congela en `reservas` a traves del calculo que se hace al momento de la reserva).

---

## 6.4. Inventario de Habitaciones (RF3)

- Cada habitacion fisica (`habitaciones`) tiene un estado: `DISPONIBLE`, `OCUPADA` o `MANTENIMIENTO`.
- Solo las habitaciones con estado `DISPONIBLE` y activa (`activa = TRUE`) pueden ser reservadas.
- Cuando se crea una reserva, la habitacion pasa automaticamente a `OCUPADA` (si la reserva es confirmada). Al finalizar la estancia (checkout) se actualiza a `DISPONIBLE`.
- El mantenimiento puede ser activado manualmente por el administrador a traves de `HabitacionController`.

---

## 6.5. Creacion de Reservas (RF4) – Paso a paso

1. El usuario (recepcionista o admin) selecciona un cliente existente o lo crea en el momento.
2. Se eligen las fechas de entrada y salida. JavaScript valida que `fecha_salida > fecha_entrada`.
3. Se consulta la disponibilidad (RF8) mediante una consulta SQL que verifica que no existan reservas activas o confirmadas para esa habitacion en el rango de fechas.
4. Se selecciona una habitacion disponible del tipo deseado.
5. Se calcula el costo de las noches: se obtiene la tarifa de `tarifas_temporada` para el tipo de habitacion y la temporada de la fecha de entrada.
6. Se toman los datos del cliente y se copian en los campos `nombre_titular`, `apellido_paterno_titular`, etc., de la tabla `reservas`. Esto asegura que aunque el cliente modifique sus datos despues, la reserva conserve la informacion original.
7. Se crea el registro en `reservas` con estado `ACTIVA` (o `CONFIRMADA` si se requiere un paso adicional). Se registra la fecha de creacion.
8. La habitacion se marca como `OCUPADA` (si la reserva es confirmada) o se deja en `DISPONIBLE` si solo es una reserva provisional (según politica del hotel).
9. Se genera un registro en `historial_reservas` con el cambio de estado (de `NULL` a `ACTIVA` o `CONFIRMADA`) y el usuario que la creo.
10. Se pueden agregar servicios adicionales (RF6) en este paso o posteriormente.

---

## 6.6. Servicios Adicionales (RF6)

- Los servicios se eligen de un catalogo (`servicios`) gestionado por `ServicioController`.
- Al agregarlos a una reserva, se guardan en `reserva_servicios` con el precio unitario vigente en ese momento (`precio_unitario`), congelando asi el costo.
- Se permite modificar la cantidad de un servicio antes del checkout; el precio unitario permanece congelado.
- El total de servicios se calcula como `SUM(cantidad * precio_unitario)` para la facturacion.

---

## 6.7. Registro de Pagos (RF7)

- Los pagos se registran en la tabla `pagos` asociados a una reserva. Se puede registrar un abono parcial o el total.
- El metodo de pago se almacena (`EFECTIVO`, `TARJETA`, `TRANSFERENCIA`). No se almacenan datos sensibles (RNF5).
- El monto debe ser positivo y no se valida contra el total adeudado en el momento del registro (se puede aceptar sobrepago, pero el sistema lo mostrara en el saldo).
- Cada pago queda registrado con su fecha.

---

## 6.8. Cancelacion de Reservas (RF5)

- Solo se pueden cancelar reservas con estado `ACTIVA` o `CONFIRMADA`.
- Al cancelar, se calcula una penalizacion según la politica (por ejemplo, si la cancelacion es con menos de 48 horas de anticipacion, se cobra el 50% de la primera noche). Este valor se almacena en `penalizacion`.
- La habitacion se libera (cambia a `DISPONIBLE`) si no esta ocupada fisicamente.
- El estado de la reserva pasa a `CANCELADA` y se registra la `fecha_cancelacion`.
- Se genera un registro en `historial_reservas` indicando el cambio y el usuario que lo realizo.
- Los pagos ya registrados permanecen; el saldo pendiente se recalcula para la facturacion final.

---

## 6.9. Consulta de Disponibilidad (RF8)

- Se realiza una consulta SQL que utiliza el indice `idx_reservas_fechas` para obtener rapidamente las habitaciones que no tienen reservas activas o confirmadas en el rango solicitado.
- La consulta filtra tambien por `estado_actual = 'DISPONIBLE'` y `activa = TRUE`.
- La interfaz muestra las habitaciones disponibles con su tipo y precio estimado.

---

## 6.10. Facturacion al Checkout (RF9)

Al finalizar la estancia, el sistema calcula el total a pagar:

- **Costo de noches:** `DATEDIFF(fecha_salida, fecha_entrada) * precio_noche` (precio tomado de la tarifa correspondiente a la fecha de entrada).
- **Total servicios:** suma de `cantidad * precio_unitario` de `reserva_servicios`.
- **Total pagado:** suma de `monto` de `pagos`.
- **Saldo pendiente:** `costo_noches + total_servicios - total_pagado - penalizacion` (si aplica).

El sistema presenta esta factura al recepcionista, quien puede registrar un pago final para saldar la cuenta.

Una vez saldada, se cambia el estado de la reserva a `COMPLETADA` y la habitacion se libera (`DISPONIBLE`).

Se registra el evento en el historial.

---

## 6.11. Auditoria (RF10)

- Cada cambio de estado de una reserva (creacion, confirmacion, cancelacion, checkout) se registra en `historial_reservas`.
- Se almacena el estado anterior, el nuevo, el usuario que realizo el cambio y un comentario opcional.
- Esto permite rastrear cualquier modificacion y es visible solo para el administrador (y en parte para el recepcionista sobre sus propias reservas).
- Se ha añadido una vista `frontend/views/historial/index.php` (solo ADMIN) para consultar el historial global de todas las reservas, con filtros por fecha y usuario.

---

## 6.12. Gestion de Usuarios (solo ADMIN)

- El administrador puede listar, crear, editar y eliminar usuarios a traves de `UsuarioController` y las vistas en `frontend/views/usuarios/`.
- Al crear o editar un usuario, se puede asignar el rol (`ADMIN` o `RECEPCIONISTA`) y establecer o cambiar la contraseña (si se proporciona una nueva, se genera su hash).
- Esta funcionalidad no esta disponible para el recepcionista.

---

## 6.13. Consistencia y Prevencion de Anomalias

- La base de datos utiliza claves foraneas con `ON DELETE CASCADE` para mantener la integridad referencial, pero se complementa con restricciones `CHECK` y logica de aplicacion.
- Para evitar dobles reservas, la consulta de disponibilidad se ejecuta dentro de una transaccion al momento de crear la reserva, bloqueando la habitacion (nivel de aislamiento `READ COMMITTED` o `SERIALIZABLE` según el motor).
- Las fechas se validan tanto en frontend (JS) como en backend (PHP) y en la base de datos (`CHECK`).
- Los precios se congelan al momento de la reserva, evitando que cambios posteriores en tarifas o servicios afecten las reservas ya creadas.

---

## Estrategia de Validacion y Estilos (Separacion de Capas)

Para garantizar un codigo limpio, mantenible y alineado con las buenas practicas de desarrollo web, se establece una separacion tajante entre la logica de presentacion (frontend) y la logica de negocio (backend).

### Validaciones con JavaScript (Frontend)

- Todas las validaciones de campos en el lado del cliente se realizaran **EXCLUSIVAMENTE** mediante JavaScript, alojado en el archivo `frontend/public/js/app.js`.
- Esto incluye: verificacion de campos obligatorios, formato de correo electronico, longitud de numeros telefonicos, fechas logicas (que la fecha de salida sea posterior a la de entrada), montos positivos y formatos de numeros decimales.
- El objetivo es proporcionar retroalimentacion inmediata al usuario y reducir el envio de peticiones innecesarias al servidor, mejorando la experiencia de usuario.

### Estilos y Diseno con CSS (Frontend)

- Todo el diseno visual, layout, colores, tipografias, efectos de hover, sombras y responsividad se definiran **EXCLUSIVAMENTE** mediante hojas de estilo CSS, alojadas en el archivo `frontend/public/css/style.css`.
- Las reglas se aplicaran mediante clases e identificadores en el HTML. No se permitira el uso de estilos en linea (`style="..."`) ni etiquetas `<style>` dentro de las vistas PHP para mantener la consistencia y facilitar el mantenimiento.

### Logica de Negocio con PHP (Backend)

- Los archivos PHP (controladores y modelos en `backend/`) **NO** contendran codigo HTML, CSS ni JavaScript. Su funcion se limitara a:
    - Recibir peticiones del frontend.
    - Realizar validaciones de seguridad y reglas de negocio en el servidor (ej. verificar que el usuario tenga permisos, evitar inyecciones SQL, comprobar integridad de datos antes de guardar, y verificar la contraseña mediante `password_verify()`).
    - Interactuar con la base de datos.
    - Devolver respuestas en formato JSON o redirigir a las vistas correspondientes.
- Esta arquitectura MVC (Modelo-Vista-Controlador) asegura que los programadores de frontend y backend puedan trabajar en paralelo sin conflictos.

---

## 7. Control de Acceso por Rol (Permisos)

| Modulo / Funcionalidad | Administrador (ADMIN) | Recepcionista (RECEPCIONISTA) |
|------------------------|----------------------|-------------------------------|
| Autenticacion (Login con usuario y contraseña) | Si, verifica hash | Si, verifica hash |
| Dashboard | Visualizacion completa | Visualizacion resumida |
| Clientes | Crear, Leer, Actualizar, Borrar logico | Crear, Leer, Actualizar, Borrar logico |
| Tipos de Habitacion (catalogo) | Crear, Leer, Actualizar, Eliminar | Solo lectura |
| Tarifas Temporada (catalogo) | Crear, Leer, Actualizar, Eliminar | Solo lectura |
| Habitaciones (inventario fisico) | Crear, Leer, Actualizar, Eliminar | Leer, actualizar estado (disponible/ocupada) |
| Reservas | Crear, Leer, Actualizar, Cancelar, Completar | Crear, Leer, Actualizar, Cancelar (solo propias) |
| Servicios (catalogo) | Crear, Leer, Actualizar, Eliminar | Solo lectura |
| Reserva-Servicios (asignacion) | Gestionar (añadir/quitar) | Gestionar (añadir/quitar) sobre reservas propias |
| Pagos | Registrar, Leer, Anular | Registrar, Leer, Anular (sobre reservas propias) |
| Historial / Auditoria | Lectura completa (global) | Solo lectura de reservas propias |
| Usuarios (incluye cambio de contraseña) | Crear, Leer, Actualizar, Eliminar | No permitido |
| Reportes | Todos los reportes | Solo reportes basicos de ocupacion |
| Configuracion | Acceso total | No permitido |

## 8. Consultas Útiles para Validación y Reportes(tomar como ejemplo.)
Consulta de Disponibilidad de Habitaciones (Sin Colisiones)
Esta consulta verifica si una habitación específica física está libre dentro de un rango de fechas dado, ignorando las reservas canceladas.
```sql
SELECT h.id_habitacion, h.estado_actual, th.nombre_tipo
FROM habitaciones h
JOIN tipos_habitacion th ON h.id_tipo_habitacion = th.id_tipo_habitacion
WHERE h.id_habitacion = ?
AND h.activa = TRUE
AND NOT EXISTS (
SELECT 1
FROM reservas r
WHERE r.id_habitacion = h.id_habitacion
AND r.estado_reserva IN ('ACTIVA', 'CONFIRMADA')
-- Lógica de colisión de intervalos: (A < Fin_B) Y (B_Inicio < Fin_A)
AND (r.fecha_entrada < ? AND r.fecha_salida > ?)
);
```
Obtención de la Tarifa Estacional Vigente (Para Congelar en Reserva)
Antes de insertar la reserva, el sistema debe buscar qué precio corresponde al día de entrada del huésped según el tipo de habitación.
```sql
SELECT precio_noche
FROM tarifas_temporada
WHERE id_tipo_habitacion = ?
AND ? BETWEEN fecha_inicio AND fecha_fin
LIMIT 1;
```
Consulta de Saldos Corregida (Manejo de Cuentas por Subconsultas)
Esta consulta unifica el costo total del hospedaje, los extras y los abonos realizados. Resuelve por completo el producto cartesiano agrupando en subtablas antes de realizar el LEFT JOIN.
```sql
SELECT
r.id_reserva,
r.nombre_titular,
r.apellido_paterno_titular,
r.fecha_entrada,
r.fecha_salida,
DATEDIFF(r.fecha_salida, r.fecha_entrada) AS noches_estancia,
r.precio_noche_congelado,
-- 1. Costo base de hospedaje
(DATEDIFF(r.fecha_salida, r.fecha_entrada) * r.precio_noche_congelado) AS costo_hospedaje,
-- 2. Consumos adicionales consolidados externamente
COALESCE(sub_servicios.total_servicios, 0) AS total_servicios,
-- 3. Penalizaciones añadidas directamente
COALESCE(r.penalizacion, 0) AS penalizaciones,
-- 4. Pagos consolidados externamente
COALESCE(sub_pagos.total_pagado, 0) AS total_abonado,
-- 5. Cálculo final neto
((DATEDIFF(r.fecha_salida, r.fecha_entrada) * r.precio_noche_congelado)
+ COALESCE(sub_servicios.total_servicios, 0)
+ COALESCE(r.penalizacion, 0)
- COALESCE(sub_pagos.total_pagado, 0)) AS saldo_pendiente
FROM reservas r
LEFT JOIN (
SELECT id_reserva, SUM(cantidad * precio_unitario) AS total_servicios
FROM reserva_servicios
GROUP BY id_reserva
) sub_servicios ON r.id_reserva = sub_servicios.id_reserva
LEFT JOIN (
SELECT id_reserva, SUM(monto) AS total_pagado
FROM pagos
GROUP BY id_reserva
) sub_pagos ON r.id_reserva = sub_pagos.id_reserva
WHERE r.id_reserva = ?;
```