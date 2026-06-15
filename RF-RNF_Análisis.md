# Análisis de Requerimientos – Sistema de Gestión de Citas para Hotel (RF, RNF, Errores)
**Enfoque académico – Proyecto base de datos**

**Autor:** Angel Uriel Monterrosas Gonzalez  
**Matrícula:** 202339872  
**Institución:** Benemérita Universidad Autónoma de Puebla (BUAP)  
**Estudios:** Licenciatura en Ingeniería en Ciencias de la Computación  
**Proyecto:** Base de datos para gestión de reservas de hotel (escala académica)  
**Fecha:** 2026-06-14  

---

## 1. Requerimientos funcionales (RF) – Base de datos

| ID  | Requerimiento                                                                 | Criterio de aceptación                                                                 |
|-----|-------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------|
| RF1 | Registrar, actualizar y eliminar lógicamente (soft delete) clientes.          | Tabla `Cliente` con campos: id_cliente, nombre, email, teléfono, activo (boolean).      |
| RF2 | Mantener catálogo de tipos de habitación (simple, doble, suite) con precio base por temporada (baja/alta). | Tabla `TipoHabitacion` y `PrecioTemporada` (nombre_temporada, precio).                  |
| RF3 | Gestionar inventario de habitaciones individuales (número, piso, estado: disponible, mantenimiento, ocupada). | Tabla `Habitacion` con FK a `TipoHabitacion`. Estado actual.                             |
| RF4 | Crear una reserva con fechas de entrada/salida, cliente, habitación y servicios opcionales. | Tabla `Reserva` que no permita fechas solapadas para la misma habitación (mediante restricción o trigger). |
| RF5 | Cancelar reserva (cambio de estado a 'cancelada') y liberar habitación automáticamente. | Trigger `after update` que reactive disponibilidad si estado pasa a cancelada.          |
| RF6 | Asociar servicios adicionales (ej. desayuno, limpieza especial) a una reserva, con precio y cantidad. | Tabla `ReservaServicio` (id_reserva, id_servicio, cantidad, precio_momento).            |
| RF7 | Registrar pagos parciales o totales de una reserva, vinculando cada pago a un método (tarjeta, efectivo). | Tabla `Pago` con monto, fecha, id_reserva, método.                                      |
| RF8 | Consultar disponibilidad de habitaciones por rango de fechas y tipo (vista o función). | Función o procedimiento `disponibilidad(fecha_inicio, fecha_fin, id_tipo)` que devuelve lista de habitaciones libres. |
| RF9 | Generar factura final al checkout (suma de noches + servicios – pagos).        | Procedimiento almacenado `calcular_factura(id_reserva)` que retorna desglose.           |
| RF10 | Mantener historial de cambios de estado de reserva (creada, confirmada, cancelada, completada). | Tabla `HistorialReserva` con trigger que registre usuario, timestamp y estado anterior. |

---

## 2. Requerimientos no funcionales (RNF)

| ID  | Requerimiento no funcional | Especificación técnica (escala académica)                                             |
|-----|----------------------------|----------------------------------------------------------------------------------------|
| RNF1 | **Rendimiento**            | Consulta de disponibilidad para 7 días debe resolverse en < 500 ms con hasta 100 habitaciones y 1,000 reservas registradas. Índices simples en `Reserva(fecha_entrada, fecha_salida, id_habitacion)`. |
| RNF2 | **Concurrencia**           | Soporte para 5-10 usuarios simultáneos (simulación de recepción y consultas web). Nivel de aislamiento `READ COMMITTED`. |
| RNF3 | **Consistencia**           | Restricción `CHECK (fecha_salida > fecha_entrada)`. Evitar reservas superpuestas mediante trigger o restricción de exclusión (según SGBD). |
| RNF4 | **Disponibilidad**         | Respaldo manual o programado semanal. No se exige alta disponibilidad (entorno académico). |
| RNF5 | **Seguridad**              | Roles básicos: `recepcionista` (CRUD en reservas, clientes, pagos), `admin` (estructura). No se almacenan datos sensibles de tarjetas (solo método y monto). |
| RNF6 | **Escalabilidad**          | Diseño que permita agregar más habitaciones o servicios sin reestructurar tablas. No se requiere particionamiento. |
| RNF7 | **Mantenibilidad**         | Entregar diagrama ER, diccionario de datos y script SQL comentado.                    |
| RNF8 | **Usabilidad**             | Las consultas complejas deben empaquetarse en funciones o vistas con nombres autoexplicativos. |

---

## 3. Manejo de errores y casos borde (nivel académico)

| Caso | Estrategia |
|------|-------------|
| Intento de reserva en habitación ya ocupada | La función de disponibilidad solo devuelve habitaciones libres. Si se intenta insertar manualmente una reserva conflictiva, un trigger `before insert` verifica solapamiento y lanza un error con mensaje claro. |
| Cancelación fuera de plazo (menos de 24h antes de entrada) | Regla académica: se permite cancelación pero se registra una penalización simbólica (ej. cargo del 10%). La tabla `Reserva` almacena la fecha de cancelación. |
| Pago mayor al saldo de la reserva | El sistema de pago debe rechazarlo con mensaje "El monto excede el saldo pendiente". Se valida en la aplicación o mediante un trigger `before insert` en `Pago`. |
| Checkout sin haber pagado todo | El sistema permite generar factura con saldo pendiente y marcar la reserva como 'adeudo'. Opcional: generar reporte de adeudos. |
| Eliminación de un tipo de habitación con reservas futuras | La FK desde `Habitacion` a `TipoHabitacion` debe ser `ON DELETE RESTRICT`. Para desactivar un tipo, se utiliza un campo `activo` (booleano) y se impiden nuevas reservas por lógica de aplicación. |
| Inserción de fechas inválidas (salida <= entrada) | Restricción `CHECK (fecha_salida > fecha_entrada)` en la tabla `Reserva`. La base de datos rechaza la inserción. |
| Fallo de conexión durante una transacción | En scripts académicos se recomienda usar `BEGIN TRANSACTION` y `ROLLBACK` en caso de error. La aplicación puede reintentar una vez. |
