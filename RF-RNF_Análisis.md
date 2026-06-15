# Análisis de Requerimientos – Sistema de Gestión de Citas para Hotel (RF, RNF, Errores)

**Autor:** Angel Uriel Monterrosas Gonzalez  
**Matrícula:** 202339872  
**Institución:** Benemérita Universidad Autónoma de Puebla (BUAP)  
**Estudios:** Licenciatura en Ingeniería en Ciencias de la Computación  
**Proyecto:** Base de datos para gestión de reservas de hotel  
**Fecha:** 2026-06-14  

---

## 1. Requerimientos funcionales (RF) – Base de datos

| ID  | Requerimiento                                                                 | Criterio de aceptación                                                                 |
|-----|-------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------|
| RF1 | Registrar, actualizar y eliminar lógicamente (soft delete) clientes.          | Tabla `Cliente` con campos: id_cliente, nombre, email, teléfono, activo (boolean).      |
| RF2 | Mantener catálogo de tipos de habitación (simple, doble, suite) con precios base por temporada. | Tabla `TipoHabitacion` y `PrecioTemporada` (fecha_inicio, fecha_fin, precio).           |
| RF3 | Gestionar inventario de habitaciones individuales (número, piso, estado: disponible, mantenimiento, ocupada). | Tabla `Habitacion` con FK a `TipoHabitacion`. Estado actual.                            |
| RF4 | Crear una reserva con fechas de entrada/salida, cliente, habitación(s) y servicios opcionales. | Tabla `Reserva` que no permita fechas solapadas para la misma habitación (trigger o restricción de exclusión). |
| RF5 | Cancelar reserva (cambio de estado a 'cancelada') y liberar habitaciones automáticamente. | Trigger `after update` que reactive disponibilidad si estado pasa a cancelada.          |
| RF6 | Asociar servicios adicionales (masajes, cenas, tours) a una reserva, con precio y cantidad. | Tabla `ReservaServicio` (id_reserva, id_servicio, cantidad, precio_momento).            |
| RF7 | Registrar pagos parciales o totales de una reserva, vinculando cada pago a un método (tarjeta, efectivo, transferencia). | Tabla `Pago` con monto, fecha, id_reserva, método.                                      |
| RF8 | Consultar disponibilidad de habitaciones por rango de fechas y tipo (función o procedimiento almacenado). | Función `disponibilidad(fecha_inicio, fecha_fin, id_tipo)` que devuelve lista de habitaciones libres. |
| RF9 | Generar factura final al checkout (suma de noches + servicios consumidos – pagos). | Procedimiento almacenado `calcular_factura(id_reserva)` que retorna JSON con desglose.   |
| RF10 | Mantener historial de cambios de estado de reserva (creada, confirmada, cancelada, completada). | Tabla `HistorialReserva` con trigger automático que registre usuario, timestamp y estado anterior. |

---

## 2. Requerimientos no funcionales (RNF)

| ID  | Requerimiento no funcional | Especificación técnica                                                                 |
|-----|----------------------------|-----------------------------------------------------------------------------------------|
| RNF1 | **Rendimiento**            | Consulta de disponibilidad para 7 días debe resolverse en < 200 ms con 10,000 habitaciones y 500,000 reservas. Índices en `Reserva(fecha_entrada, fecha_salida, id_habitacion)`. |
| RNF2 | **Concurrencia**           | Soporte mínimo para 50 usuarios simultáneos. Nivel de aislamiento `READ COMMITTED`.    |
| RNF3 | **Consistencia**           | CHECK: fecha_salida > fecha_entrada. No permitir reservas superpuestas (índice de exclusión o trigger serializable). |
| RNF4 | **Disponibilidad**         | Respaldo automático diario + replicación de solo lectura. RTO = 4 horas, RPO = 1 hora. |
| RNF5 | **Seguridad**              | Roles: `recepcionista` (CRUD en reservas, clientes, pagos), `gerente` (catálogos), `admin` (estructura). Encriptar columna `tarjeta_pago` si se almacena. |
| RNF6 | **Escalabilidad**          | Particionar `Reserva` por rango de fechas (mensual). Soporte para multitenant por `id_hotel` en futura versión. |
| RNF7 | **Mantenibilidad**         | Diccionario de datos, diagrama ER y scripts versionados (Flyway/Liquibase).            |
| RNF8 | **Usabilidad para desarrolladores** | Exponer consultas complejas como vistas materializadas o funciones con parámetros claros. |

---

## 3. Manejo de errores y casos borde

| Caso | Estrategia |
|------|-------------|
| Intento de reserva en habitación ya ocupada | Función de disponibilidad filtra habitaciones libres. Si se intenta insertar igual, trigger `check_overlap` lanza SQLSTATE `P0001` con mensaje "Habitación no disponible para las fechas indicadas". |
| Cancelación fuera de plazo (menos de 24h antes de entrada) | Regla de negocio: no cancelación gratuita. La app valida antes del UPDATE. La BD acepta cambio pero se registra penalización en tabla `Penalizacion`. |
| Pago mayor al saldo de la reserva | Trigger `before insert` verifica y emite advertencia; requiere autorización de gerente (campo `autorizado_por`). |
| Checkout sin haber pagado todo | Se permite pero estado de reserva pasa a 'adeudo' y se genera registro en `CuentaPorCobrar`. |
| Eliminación de un tipo de habitación con reservas futuras | FK `ON DELETE RESTRICT`. Para desactivar tipo, se usa columna `activo = false` y la app impide nuevas reservas. |
| Falla de conexión durante transacción | `retry logic` en app con 3 intentos y timeout exponencial. BD con `autocommit = off` y rollback explícito en errores. |
| Inserción de fechas inválidas (salida <= entrada) | Restricción `CHECK (fecha_salida > fecha_entrada)` en tabla `Reserva`. La BD rechaza la operación con error claro. |
