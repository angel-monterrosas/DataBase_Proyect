# Diccionario de Datos – Sistema de Gestión de Citas para Hotel

**Autores:** Angel Uriel Monterrosas Gonzalez, Noe Tlachi Zenteno  
**Matrícula:** 202339872, 202162606  
**Institución:** Benemérita Universidad Autónoma de Puebla (BUAP)  
**Materia:** Base de datos para ingenierías  
**Fecha:** 16 de junio de 2026

---

## 1. Resumen del modelo de datos

El sistema se compone de las siguientes entidades principales:

- **Clientes**: Personas que realizan reservas.
- **Tipos de habitación**: Categorías (simple, doble, suite) con precios base.
- **Tarifas por temporada**: Precios específicos según temporada (alta/baja) para cada tipo de habitación.
- **Habitaciones**: Unidades físicas con número, piso y estado actual.
- **Reservas**: Asociación entre cliente, habitación y fechas, con estado y penalización.
- **Servicios**: Catálogo de servicios adicionales (desayuno, spa, etc.).
- **Reserva_Servicios**: Servicios contratados en cada reserva, con precio congelado.
- **Pagos**: Registro de abonos parciales o totales.
- **Usuarios**: Personal que gestiona reservas.
- **Historial_Reservas**: Auditoría de cambios en el estado de las reservas.



---

## 2. Diccionario de tablas

### Tabla: `clientes`
**Descripción:** Almacena los datos de los huéspedes. Soporta borrado lógico mediante el campo `activo`.

| Columna | Tipo de dato | Longitud / Precisión | ¿Nulo? | Clave | Descripción | Restricciones / Validaciones |
|---------|--------------|----------------------|--------|-------|-------------|------------------------------|
| `id_cliente` | `INT` | - | No | PK | Identificador único del cliente | Auto_increment |
| `nombre` | `VARCHAR` | 100 | No | - | Nombre completo del cliente | - |
| `telefono` | `VARCHAR` | 20 | Sí | - | Número de contacto | - |
| `correo` | `VARCHAR` | 100 | Sí | - | Correo electrónico | Debe ser único (`UNIQUE`) |
| `activo` | `BOOLEAN` | - | No | - | Indica si el cliente está activo (TRUE) o eliminado lógicamente (FALSE) | Por defecto TRUE |

**Relaciones:**
- `id_cliente` → `reservas(id_cliente)`

---

### Tabla: `tipos_habitacion`
**Descripción:** Catálogo de categorías de habitaciones disponibles.

| Columna | Tipo de dato | Longitud / Precisión | ¿Nulo? | Clave | Descripción | Restricciones / Validaciones |
|---------|--------------|----------------------|--------|-------|-------------|------------------------------|
| `id_tipo_habitacion` | `INT` | - | No | PK | Identificador del tipo | Auto_increment |
| `nombre` | `VARCHAR` | 50 | No | - | Nombre (Ej. "Simple", "Doble", "Suite") | - |
| `capacidad` | `INT` | - | No | - | Número de personas | > 0 |
| `descripcion` | `TEXT` | - | Sí | - | Detalles adicionales | - |

**Relaciones:**
- `id_tipo_habitacion` → `habitaciones(id_tipo_habitacion)` y `tarifas_temporada(id_tipo_habitacion)`

---

### Tabla: `tarifas_temporada`
**Descripción:** Define precios por tipo de habitación según temporada (alta/baja). Permite variaciones estacionales.

| Columna | Tipo de dato | Longitud / Precisión | ¿Nulo? | Clave | Descripción | Restricciones / Validaciones |
|---------|--------------|----------------------|--------|-------|-------------|------------------------------|
| `id_tarifa` | `INT` | - | No | PK | Identificador de la tarifa | Auto_increment |
| `id_tipo_habitacion` | `INT` | - | No | FK | Referencia al tipo de habitación | - |
| `temporada` | `ENUM` | ('ALTA','BAJA') | No | - | Indica si es temporada alta o baja | - |
| `precio_noche` | `DECIMAL` | 10,2 | No | - | Precio por noche en esa temporada | > 0 |
| `fecha_inicio` | `DATE` | - | No | - | Fecha de inicio de vigencia | - |
| `fecha_fin` | `DATE` | - | No | - | Fecha de fin de vigencia | `fecha_fin > fecha_inicio` |

**Relaciones:**
- `id_tipo_habitacion` → `tipos_habitacion(id_tipo_habitacion)`

**Restricciones adicionales:**  
`UNIQUE (id_tipo_habitacion, temporada)` para evitar duplicados.

---

### Tabla: `habitaciones`
**Descripción:** Registro de cada habitación física con su estado actual.

| Columna | Tipo de dato | Longitud / Precisión | ¿Nulo? | Clave | Descripción | Restricciones / Validaciones |
|---------|--------------|----------------------|--------|-------|-------------|------------------------------|
| `id_habitacion` | `INT` | - | No | PK | Número de habitación | - |
| `id_tipo_habitacion` | `INT` | - | No | FK | Tipo de habitación | - |
| `piso` | `INT` | - | No | - | Número de piso | - |
| `estado_actual` | `ENUM` | ('DISPONIBLE','OCUPADA','MANTENIMIENTO') | No | - | Estado actual de la habitación | - |
| `activa` | `BOOLEAN` | - | No | - | Si la habitación está operativa | Por defecto TRUE |

**Relaciones:**
- `id_tipo_habitacion` → `tipos_habitacion(id_tipo_habitacion)`
- `id_habitacion` → `reservas(id_habitacion)`

---

### Tabla: `reservas`
**Descripción:** Almacena las reservas realizadas por clientes, incluyendo fechas, estado, penalizaciones y fecha de cancelación.

| Columna | Tipo de dato | Longitud / Precisión | ¿Nulo? | Clave | Descripción | Restricciones / Validaciones |
|---------|--------------|----------------------|--------|-------|-------------|------------------------------|
| `id_reserva` | `INT` | - | No | PK | Identificador de la reserva | Auto_increment |
| `id_cliente` | `INT` | - | No | FK | Cliente que realiza la reserva | - |
| `id_habitacion` | `INT` | - | No | FK | Habitación asignada | - |
| `fecha_entrada` | `DATE` | - | No | - | Fecha de inicio de la estancia | - |
| `fecha_salida` | `DATE` | - | No | - | Fecha de fin de la estancia | `fecha_salida > fecha_entrada` |
| `estado_reserva` | `ENUM` | ('ACTIVA','CONFIRMADA','CANCELADA','COMPLETADA') | No | - | Estado actual de la reserva | Por defecto 'ACTIVA' |
| `fecha_creacion` | `DATETIME` | - | No | - | Fecha y hora de creación | Por defecto `CURRENT_TIMESTAMP` |
| `penalizacion` | `DECIMAL` | 10,2 | Sí | - | Monto cobrado por cancelación tardía | Por defecto 0.00 |
| `fecha_cancelacion` | `DATETIME` | - | Sí | - | Fecha y hora de cancelación (si aplica) | - |

**Relaciones:**
- `id_cliente` → `clientes(id_cliente)`
- `id_habitacion` → `habitaciones(id_habitacion)`
- `id_reserva` → `reserva_servicios(id_reserva)`, `pagos(id_reserva)`, `historial_reservas(id_reserva)`

---

### Tabla: `servicios`
**Descripción:** Catálogo de servicios adicionales que se pueden contratar.

| Columna | Tipo de dato | Longitud / Precisión | ¿Nulo? | Clave | Descripción | Restricciones / Validaciones |
|---------|--------------|----------------------|--------|-------|-------------|------------------------------|
| `id_servicio` | `INT` | - | No | PK | Identificador del servicio | Auto_increment |
| `nombre` | `VARCHAR` | 100 | No | - | Nombre del servicio | - |
| `precio_vigente` | `DECIMAL` | 10,2 | No | - | Precio actual del servicio | > 0 |
| `descripcion` | `TEXT` | - | Sí | - | Detalles adicionales | - |

**Relaciones:**
- `id_servicio` → `reserva_servicios(id_servicio)`

---

### Tabla: `reserva_servicios`
**Descripción:** Servicios contratados en cada reserva, con la cantidad y el precio congelado en el momento de la reserva.

| Columna | Tipo de dato | Longitud / Precisión | ¿Nulo? | Clave | Descripción | Restricciones / Validaciones |
|---------|--------------|----------------------|--------|-------|-------------|------------------------------|
| `id_reserva_servicio` | `INT` | - | No | PK | Identificador único | Auto_increment |
| `id_reserva` | `INT` | - | No | FK | Reserva a la que pertenece | - |
| `id_servicio` | `INT` | - | No | FK | Servicio contratado | - |
| `cantidad` | `INT` | - | No | - | Número de unidades | > 0 |
| `precio_unitario` | `DECIMAL` | 10,2 | No | - | Precio unitario al momento de la contratación | - |

**Relaciones:**
- `id_reserva` → `reservas(id_reserva)`
- `id_servicio` → `servicios(id_servicio)`

---

### Tabla: `pagos`
**Descripción:** Registro de pagos realizados por el cliente para una reserva.

| Columna | Tipo de dato | Longitud / Precisión | ¿Nulo? | Clave | Descripción | Restricciones / Validaciones |
|---------|--------------|----------------------|--------|-------|-------------|------------------------------|
| `id_pago` | `INT` | - | No | PK | Identificador del pago | Auto_increment |
| `id_reserva` | `INT` | - | No | FK | Reserva a la que corresponde | - |
| `monto` | `DECIMAL` | 10,2 | No | - | Monto del pago | > 0 |
| `fecha_pago` | `DATETIME` | - | No | - | Fecha y hora del pago | Por defecto `CURRENT_TIMESTAMP` |
| `metodo_pago` | `ENUM` | ('EFECTIVO','TARJETA','TRANSFERENCIA') | No | - | Método de pago | - |

**Relaciones:**
- `id_reserva` → `reservas(id_reserva)`

**Observación de seguridad (RNF5):** No se almacenan datos sensibles como números de tarjeta; solo el método y monto.

---

### Tabla: `usuarios`
**Descripción:** Personal que puede gestionar reservas. Necesario para el historial de cambios.

| Columna | Tipo de dato | Longitud / Precisión | ¿Nulo? | Clave | Descripción | Restricciones / Validaciones |
|---------|--------------|----------------------|--------|-------|-------------|------------------------------|
| `id_usuario` | `INT` | - | No | PK | Identificador del usuario | Auto_increment |
| `nombre_usuario` | `VARCHAR` | 50 | No | - | Nombre de usuario para login | - |
| `nombre_completo` | `VARCHAR` | 100 | No | - | Nombre real del empleado | - |
| `rol` | `ENUM` | ('ADMIN','RECEPCIONISTA') | No | - | Rol dentro del sistema | - |

**Relaciones:**
- `id_usuario` → `historial_reservas(id_usuario)`

---

### Tabla: `historial_reservas`
**Descripción:** Auditoría de cambios en el estado de las reservas (RF10). Registra quién, cuándo y qué estado cambió.

| Columna | Tipo de dato | Longitud / Precisión | ¿Nulo? | Clave | Descripción | Restricciones / Validaciones |
|---------|--------------|----------------------|--------|-------|-------------|------------------------------|
| `id_historial` | `INT` | - | No | PK | Identificador del registro | Auto_increment |
| `id_reserva` | `INT` | - | No | FK | Reserva afectada | - |
| `id_usuario` | `INT` | - | No | FK | Usuario que realizó el cambio | - |
| `estado_anterior` | `VARCHAR` | 20 | No | - | Estado antes del cambio | - |
| `estado_nuevo` | `VARCHAR` | 20 | No | - | Estado después del cambio | - |
| `fecha_cambio` | `DATETIME` | - | No | - | Fecha y hora del cambio | Por defecto `CURRENT_TIMESTAMP` |
| `comentario` | `TEXT` | - | Sí | - | Observaciones adicionales | - |

**Relaciones:**
- `id_reserva` → `reservas(id_reserva)`
- `id_usuario` → `usuarios(id_usuario)`

---

## 3. Relaciones entre tablas (Claves foráneas)

| Clave foránea | Tabla destino | Columna destino | Regla ON DELETE | Regla ON UPDATE |
|---------------|---------------|-----------------|-----------------|-----------------|
| `reservas.id_cliente` | `clientes` | `id_cliente` | RESTRICT | CASCADE |
| `reservas.id_habitacion` | `habitaciones` | `id_habitacion` | RESTRICT | CASCADE |
| `habitaciones.id_tipo_habitacion` | `tipos_habitacion` | `id_tipo_habitacion` | RESTRICT | CASCADE |
| `tarifas_temporada.id_tipo_habitacion` | `tipos_habitacion` | `id_tipo_habitacion` | CASCADE | CASCADE |
| `reserva_servicios.id_reserva` | `reservas` | `id_reserva` | CASCADE | CASCADE |
| `reserva_servicios.id_servicio` | `servicios` | `id_servicio` | RESTRICT | CASCADE |
| `pagos.id_reserva` | `reservas` | `id_reserva` | CASCADE | CASCADE |
| `historial_reservas.id_reserva` | `reservas` | `id_reserva` | CASCADE | CASCADE |
| `historial_reservas.id_usuario` | `usuarios` | `id_usuario` | RESTRICT | CASCADE |

---

