# Análisis de Requerimientos – Sistema de Gestión de Citas para Hotel

**Autor:** Angel Uriel Monterrosas Gonzalez  
**Matrícula:** 202339872  
**Institución:** Benemérita Universidad Autónoma de Puebla (BUAP)  
**Estudios:** Licenciatura en Ingeniería en Ciencias de la Computación  
**Proyecto:** Base de datos para gestión de citas (reservas de habitaciones y servicios) en hotel.  
**Fecha:** 2026-06-14  

---

## 1. Contexto y objetivo de negocio

El hotel requiere un sistema centralizado que permita gestionar reservas de habitaciones, servicios adicionales (spa, restaurante, tours) y facturación. Actualmente usan hojas de cálculo y hay problemas de doble reserva, cancelaciones manuales e inventario inconsistente de habitaciones.

**Objetivo:** Diseñar e implementar una base de datos relacional que garantice integridad referencial, evite conflictos de horario y permita consultas de disponibilidad en tiempo real.

**Alcance inicial:**  
- Módulo de clientes (registro, historial)  
- Módulo de habitaciones (tipos, precios dinámicos, estados)  
- Módulo de reservas (crear, modificar, cancelar, check-in/out)  
- Módulo de servicios adicionales (asociación a reserva)  
- Módulo de pagos (depósitos, saldos, facturación básica)

---

## 2. Requerimientos funcionales (RF) – Enfoque en base de datos

| ID  | Requerimiento                                                                 | Criterio de aceptación                                                                 |
|-----|-------------------------------------------------------------------------------|---------------------------------------------------------------------------------------|
| RF1 | Registrar, actualizar y eliminar lógicamente (soft delete) clientes.          | Tabla `Cliente` con campos: id_cliente, nombre, email, teléfono, activo (boolean).    |
| RF2 | Mantener catálogo de tipos de habitación (simple, doble, suite) con precios base por temporada. | Tabla `TipoHabitacion` y `PrecioTemporada` (fecha_inicio, fecha_fin, precio).        |
| RF3 | Gestionar inventario de habitaciones individuales (número, piso, estado: disponible, mantenimiento, ocupada). | Tabla `Habitacion` con FK a `TipoHabitacion`. Estado actual y último cambio.         |
| RF4 | Crear una reserva con fechas de entrada/salida, cliente, habitación(s) y servicios opcionales. | Tabla `Reserva` que no permita fechas solapadas para la misma habitación (vía trigger o restricción exclusión en PostgreSQL). |
| RF5 | Cancelar reserva (cambio de estado a 'cancelada') y liberar habitaciones automáticamente. | Trigger `after update` que reactive disponibilidad si estado pasa a cancelada.        |
| RF6 | Asociar servicios adicionales (masajes, cenas, tours) a una reserva, con precio y cantidad. | Tabla `ReservaServicio` (id_reserva, id_servicio, cantidad, precio_momento).          |
| RF7 | Registrar pagos parciales o totales de una reserva, vinculando cada pago a un método (tarjeta, efectivo, transferencia). | Tabla `Pago` con monto, fecha, id_reserva, método.                                    |
| RF8 | Consultar disponibilidad de habitaciones por rango de fechas y tipo (vista o procedimiento almacenado). | Función `disponibilidad(fecha_inicio, fecha_fin, id_tipo)` que devuelve lista de habitaciones libres. |
| RF9 | Generar factura final al checkout (suma de noches + servicios consumidos – pagos). | Procedimiento almacenado `calcular_factura(id_reserva)` que retorna JSON con desglose. |
| RF10 | Mantener historial de cambios de estado de reserva (creada, confirmada, cancelada, completada). | Tabla `HistorialReserva` con trigger automático que registre usuario, timestamp y estado anterior. |

---

## 3. Requerimientos no funcionales (RNF)

| ID  | Requerimiento no funcional | Especificación técnica                                                                 |
|-----|----------------------------|----------------------------------------------------------------------------------------|
| RNF1 | **Rendimiento**            | Consulta de disponibilidad para un rango de 7 días debe resolverse en < 200 ms con hasta 10,000 habitaciones y 500,000 reservas históricas. Índices en `Reserva(fecha_entrada, fecha_salida, id_habitacion)`. |
| RNF2 | **Concurrencia**           | Soporte mínimo para 50 usuarios simultáneos (recepcionistas y sistema web) sin bloqueos pesimistas excesivos. Nivel de aislamiento `READ COMMITTED` en transacciones de creación de reserva. |
| RNF3 | **Consistencia**           | Restricción CHECK: fecha_salida > fecha_entrada. No se permiten dos reservas activas superpuestas en misma habitación (índice de exclusión o trigger con serializable). |
| RNF4 | **Disponibilidad**         | Base de datos debe tener respaldo automático diario + replicación de solo lectura para consultas de disponibilidad. RTO = 4 horas, RPO = 1 hora. |
| RNF5 | **Seguridad**              | Los roles: `recepcionista` (CRUD en reservas, clientes, pagos), `gerente` (acceso a facturación y catálogos), `admin` (estructura de BD, backups). Usar encriptación AES-256 para columna `tarjeta_pago` (si se almacena). |
| RNF6 | **Escalabilidad**          | Particionar tabla `Reserva` por rango de fechas (mensual). Soporte para añadir hoteles adicionales (multitenant por `id_hotel`) en futura versión. |
| RNF7 | **Mantenibilidad**         | Diccionario de datos actualizado, diagrama ER y scripts de migración versionados (Flyway/Liquibase). Cobertura de triggers y procedimientos documentada. |
| RNF8 | **Usabilidad para desarrolladores** | Las consultas complejas (disponibilidad) deben exponerse como vistas materializadas (actualización nocturna) o funciones con parámetros claros. |

---

## 4. Restricciones tecnológicas

- **SGBD sugerido:** PostgreSQL 15+ (por soporte nativo de rangos de fechas `tsrange` y exclusión constraints) o MySQL 8+ con triggers.  
- **Tamaño estimado:** 5 GB después de 2 años (si se manejan 200 reservas/día, 5 servicios por reserva).  
- **Concurrencia pico:** 150 transacciones/segundo (fin de semana festivo).  
- **Idioma del código:** PL/pgSQL (si PostgreSQL) o SQL estándar.  
- **Entorno:** Servidor Linux con 8 GB RAM, SSD, red interna <1ms latencia.

---

## 5. Métricas de éxito (KPIs)

| KPI | Meta | Método de validación |
|-----|------|----------------------|
| Tiempo medio de consulta de disponibilidad | < 300 ms (p95) | Monitoreo con `pg_stat_statements` |
| Errores por doble reserva | 0 por mes | Registro de excepciones de aplicación |
| Tiempo de generación de factura | < 50 ms por reserva | Prueba de carga con 500 facturas simultáneas |
| Integridad referencial | 100% (sin huérfanos) | Verificación diaria mediante consultas `LEFT JOIN` con `IS NULL` |
| Backups exitosos | 99.9% (máximo 1 fallo/mes) | Log del scheduler de backups |

---

## 6. Manejo de errores y casos borde

| Caso | Estrategia |
|------|-------------|
| Intento de reserva en habitación ya ocupada | La función de disponibilidad debe devolver solo habitaciones libres; si aun así se intenta insertar, el trigger `check_overlap` lanza SQLSTATE `P0001` con mensaje "Habitación no disponible para las fechas indicadas". |
| Cancelación fuera de plazo (menos de 24h antes de entrada) | Regla de negocio: no se permite cancelación gratuita. La aplicación debe validar antes de ejecutar UPDATE. Base de datos puede aceptar cambio pero se registra penalización en tabla `Penalizacion`. |
| Pago mayor al saldo de la reserva | La tabla `Pago` permite exceso, pero un trigger `before insert` verifica y emite advertencia; se requiere autorización de gerente (campo `autorizado_por`). |
| Checkout sin haber pagado todo | Se permite pero el estado de la reserva pasa a 'adeudo' y se genera un registro en `CuentaPorCobrar`. |
| Eliminación de un tipo de habitación que tiene reservas futuras | Restricción `ON DELETE RESTRICT` en FK desde `Habitacion` hacia `TipoHabitacion`. Para desactivar un tipo, se usa columna `activo = false` y se impide nuevas reservas por lógica de aplicación. |
| Falla de conexión durante transacción | Se implementa `retry logic` en la capa de aplicación con máximo 3 intentos y timeout exponencial. La BD debe tener `autocommit = off` y rollback explícito en errores. |

---

