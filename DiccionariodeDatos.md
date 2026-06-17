# Análisis de Requerimientos – Sistema de Gestión de Citas para Hotel (RF, RNF, Errores)
**Enfoque académico – Descripción funcional y de calidad**

**Autores:** Angel Uriel Monterrosas Gonzalez  - Noe Tlachi Zenteno.
**Matrícula:** 202339872  - 202162606
**Institución:** Benemérita Universidad Autónoma de Puebla (BUAP)  
**Estudios:** Licenciatura en Ingeniería en Ciencias de la Computación  
**Proyecto:** Base de datos para gestión de reservas de hotel.
**Materia:** Base de datos para ingenierias.
**Fecha:** 15 - 06 - 2026

---
### Tabla: usuarios
**Descripción:** Almacena a los empleados o administradores del hotel. Es necesaria para cumplir con el RF10 (saber quién modifica una reserva).

| Columna | Tipo de dato | Longitud / Precisión | ¿Nulo? | Clave | Descripción | Restricciones / Validaciones |
|---------|--------------|----------------------|--------|-------|-------------|------------------------------|
| `id_usuario` | INT | - | No | PK | Identificador único del empleado. | Autoincremental. |
| `nombre` | VARCHAR | 100 | No | - | Nombre completo del empleado. | - |
| `rol` | VARCHAR | 50 | No | - | Rol dentro del sistema. | Ej: 'RECEPCIONISTA', 'ADMIN'. |
| `activo` | BOOLEAN | - | No | - | Estado del usuario en el sistema. | Default: `TRUE`. |

**Relaciones:** Ninguna.

---

### Tabla: clientes
**Descripción:** Registra la información de los huéspedes. Contempla el borrado lógico (RF1).

| Columna | Tipo de dato | Longitud / Precisión | ¿Nulo? | Clave | Descripción | Restricciones / Validaciones |
|---------|--------------|----------------------|--------|-------|-------------|------------------------------|
| `id_cliente` | INT | - | No | PK | Identificador único del cliente. | Autoincremental. |
| `nombre_completo` | VARCHAR | 150 | No | - | Nombre completo del huésped. | - |
| `telefono` | VARCHAR | 20 | Sí | - | Número de contacto. | Solo caracteres numéricos y `+`. |
| `correo` | VARCHAR | 100 | No | - | Correo electrónico principal. | Formato válido de email. UNIQUE. |
| `activo` | BOOLEAN | - | No | - | Control de borrado lógico (RF1). | Default: `TRUE`. |

**Relaciones:** Ninguna.

---

### Tabla: tipos_habitacion
**Descripción:** Catálogo maestro para los diferentes tipos de habitación (RF2).

| Columna | Tipo de dato | Longitud / Precisión | ¿Nulo? | Clave | Descripción | Restricciones / Validaciones |
|---------|--------------|----------------------|--------|-------|-------------|------------------------------|
| `id_tipo_habitacion` | INT | - | No | PK | ID único del tipo de habitación. | Autoincremental. |
| `nombre` | VARCHAR | 50 | No | - | Categoría (Simple, Doble, Suite). | UNIQUE. |
| `capacidad` | INT | - | No | - | Número máximo de huéspedes. | `CHECK (capacidad > 0)` |
| `descripcion` | TEXT | - | Sí | - | Detalles y amenidades de la habitación.| - |

**Relaciones:** Ninguna.

---

### Tabla: tarifas_temporada
**Descripción:** Gestiona los precios por temporada para cada tipo de habitación (RF2). Separarlo permite escalar a nuevas temporadas o reglas de precios (RNF6).

| Columna | Tipo de dato | Longitud / Precisión | ¿Nulo? | Clave | Descripción | Restricciones / Validaciones |
|---------|--------------|----------------------|--------|-------|-------------|------------------------------|
| `id_tarifa` | INT | - | No | PK | Identificador único de la tarifa. | Autoincremental. |
| `id_tipo_habitacion`| INT | - | No | FK | Referencia al tipo de habitación. | - |
| `temporada` | VARCHAR | 20 | No | - | Temporada aplicable (Alta, Baja). | Valores: 'ALTA', 'BAJA', 'MEDIA'. |
| `precio_noche` | DECIMAL | 10,2 | No | - | Costo por noche en la temporada. | `CHECK (precio_noche >= 0)` |

**Relaciones:**
- `id_tipo_habitacion` → `tipos_habitacion(id_tipo_habitacion)`
  *(Nota: UNIQUE compuesto para `id_tipo_habitacion` + `temporada`)*

---

### Tabla: habitaciones
**Descripción:** Inventario físico de habitaciones y su estado actual (RF3).

| Columna | Tipo de dato | Longitud / Precisión | ¿Nulo? | Clave | Descripción | Restricciones / Validaciones |
|---------|--------------|----------------------|--------|-------|-------------|------------------------------|
| `id_habitacion` | INT | - | No | PK | ID interno de la habitación. | Autoincremental. |
| `numero` | VARCHAR | 10 | No | - | Número visible de la habitación. | UNIQUE (Ej: '101A'). |
| `piso` | INT | - | No | - | Planta en la que se encuentra. | - |
| `id_tipo_habitacion`| INT | - | No | FK | Referencia al tipo de habitación. | - |
| `estado_actual` | VARCHAR | 20 | No | - | Disponibilidad en tiempo real. | Valores: 'DISPONIBLE', 'OCUPADA', 'MANTENIMIENTO'. |

**Relaciones:**
- `id_tipo_habitacion` → `tipos_habitacion(id_tipo_habitacion)`

---

### Tabla: reservas
**Descripción:** Entidad central que une clientes con habitaciones en un rango de fechas (RF4, RF5).

| Columna | Tipo de dato | Longitud / Precisión | ¿Nulo? | Clave | Descripción | Restricciones / Validaciones |
|---------|--------------|----------------------|--------|-------|-------------|------------------------------|
| `id_reserva` | INT | - | No | PK | Identificador de la reserva. | Autoincremental. |
| `id_cliente` | INT | - | No | FK | Cliente titular de la reserva. | - |
| `id_habitacion` | INT | - | No | FK | Habitación asignada. | - |
| `fecha_entrada` | DATE | - | No | - | Fecha de check-in programada. | - |
| `fecha_salida` | DATE | - | No | - | Fecha de check-out programada. | `CHECK (fecha_salida > fecha_entrada)` (RNF3) |
| `estado_reserva` | VARCHAR | 20 | No | - | Situación del proceso de reserva. | Valores: 'ACTIVA', 'COMPLETADA', 'CANCELADA'. |
| `fecha_creacion` | TIMESTAMP | - | No | - | Fecha en que se tomó la reserva. | Default: `CURRENT_TIMESTAMP`. |

**Relaciones:**
- `id_cliente` → `clientes(id_cliente)`
- `id_habitacion` → `habitaciones(id_habitacion)`
  *(Nota para RNF3: Requiere un TRIGGER a nivel base de datos que verifique que, si `estado_reserva = 'ACTIVA'`, el rango de fechas `[fecha_entrada, fecha_salida]` no se solape con otras reservas activas para el mismo `id_habitacion`).*

---

### 📋 Tabla: servicios
**Descripción:** Catálogo de servicios adicionales que ofrece el hotel (RF6).

| Columna | Tipo de dato | Longitud / Precisión | ¿Nulo? | Clave | Descripción | Restricciones / Validaciones |
|---------|--------------|----------------------|--------|-------|-------------|------------------------------|
| `id_servicio` | INT | - | No | PK | ID del servicio extra. | Autoincremental. |
| `nombre` | VARCHAR | 100 | No | - | Nombre del servicio (Spa, limpieza).| UNIQUE. |
| `precio_vigente` | DECIMAL | 10,2 | No | - | Costo actual del servicio. | `CHECK (precio_vigente >= 0)`. |

**Relaciones:** Ninguna.

---

### 📋 Tabla: reserva_servicios
**Descripción:** Tabla intermedia (N:M) que asocia servicios específicos consumidos por una reserva (RF6, RF9).

| Columna | Tipo de dato | Longitud / Precisión | ¿Nulo? | Clave | Descripción | Restricciones / Validaciones |
|---------|--------------|----------------------|--------|-------|-------------|------------------------------|
| `id_reserva_srv`| INT | - | No | PK | Identificador del consumo. | Autoincremental. |
| `id_reserva` | INT | - | No | FK | Reserva asociada. | - |
| `id_servicio` | INT | - | No | FK | Servicio consumido. | - |
| `cantidad` | INT | - | No | - | Cuántas unidades se solicitaron. | `CHECK (cantidad > 0)`. Default: 1. |
| `precio_unitario` | DECIMAL | 10,2 | No | - | Costo *en el momento* del consumo. | Evita que cambios futuros afecten facturas antiguas. |
| `fecha_consumo` | TIMESTAMP | - | No | - | Cuándo se entregó el servicio. | Default: `CURRENT_TIMESTAMP`. |

**Relaciones:**
- `id_reserva` → `reservas(id_reserva)`
- `id_servicio` → `servicios(id_servicio)`

---

### Tabla: pagos
**Descripción:** Registro de abonos y liquidaciones realizadas a favor de una reserva (RF7, RNF5).

| Columna | Tipo de dato | Longitud / Precisión | ¿Nulo? | Clave | Descripción | Restricciones / Validaciones |
|---------|--------------|----------------------|--------|-------|-------------|------------------------------|
| `id_pago` | INT | - | No | PK | ID único del pago. | Autoincremental. |
| `id_reserva` | INT | - | No | FK | Reserva que se está pagando. | - |
| `monto` | DECIMAL | 10,2 | No | - | Cantidad monetaria ingresada. | `CHECK (monto > 0)` |
| `fecha_pago` | TIMESTAMP | - | No | - | Fecha y hora exacta del ingreso. | Default: `CURRENT_TIMESTAMP`. |
| `metodo_pago` | VARCHAR | 30 | No | - | Forma en que se liquidó el pago. | Ej: 'EFECTIVO', 'T_CREDITO', 'T_DEBITO'. No almacena tarjeta (RNF5). |

**Relaciones:**
- `id_reserva` → `reservas(id_reserva)`

---

### Tabla: historial_reservas
**Descripción:** Bitácora de auditoría para rastrear quién y cuándo modificó una reserva (RF10).

| Columna | Tipo de dato | Longitud / Precisión | ¿Nulo? | Clave | Descripción | Restricciones / Validaciones |
|---------|--------------|----------------------|--------|-------|-------------|------------------------------|
| `id_historial` | INT | - | No | PK | ID del registro de auditoría. | Autoincremental. |
| `id_reserva` | INT | - | No | FK | Reserva que sufrió el cambio. | - |
| `id_usuario` | INT | - | No | FK | Empleado que hizo el cambio. | - |
| `estado_anterior`| VARCHAR | 20 | Sí | - | Estado antes de la modificación. | Puede ser nulo en su creación. |
| `estado_nuevo` | VARCHAR | 20 | No | - | Estado después de la modificación. | - |
| `fecha_cambio` | TIMESTAMP | - | No | - | Momento exacto del cambio. | Default: `CURRENT_TIMESTAMP`. |
| `detalles_extra`| TEXT | - | Sí | - | Notas como "Cambio de fechas". | - |

**Relaciones:**
- `id_reserva` → `reservas(id_reserva)`
- `id_usuario` → `usuarios(id_usuario)`

---
*Nota sobre cálculos (RF8 y RF9): El cálculo del total a pagar al Checkout y las consultas de disponibilidad no requieren tablas persistentes adicionales; se deben calcular dinámicamente mediante consultas `SELECT` cruzando datos y sumatorias sobre `tarifas_temporada`, `reserva_servicios` y `pagos`.*