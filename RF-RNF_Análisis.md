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

## 1. Requerimientos funcionales (RF)

Los requerimientos funcionales describen **qué** debe hacer el sistema de gestión de citas (reservas) del hotel.

| ID  | Requerimiento funcional | Descripción|
|-----|------------------------|--------------------------------------|
| RF1 | Gestión de clientes | El sistema debe permitir registrar nuevos clientes, modificar sus datos (nombre, teléfono, correo) y eliminarlos de forma lógica (que no aparezcan en nuevas reservas pero se conserve su historial). |
| RF2 | Catálogo de tipos de habitación | Se debe poder definir diferentes tipos de habitación (ej. simple, doble, suite) y asignar un precio base por temporada (por ejemplo, precio en temporada alta y en temporada baja). |
| RF3 | Inventario de habitaciones | Cada habitación individual (número, piso) debe tener un estado actual: disponible, ocupada o en mantenimiento. El sistema debe poder actualizar ese estado. |
| RF4 | Creación de reservas | El sistema debe permitir a un recepcionista crear una reserva indicando: cliente, habitación específica, fechas de entrada y salida, y servicios adicionales opcionales. No se deben poder crear dos reservas que solapen fechas para la misma habitación. |
| RF5 | Cancelación de reservas | Se debe poder cancelar una reserva. Al cancelar, la habitación debe quedar disponible nuevamente para futuras reservas. |
| RF6 | Servicios adicionales | El sistema debe permitir asociar servicios extra (ej. desayuno, spa, limpieza especial) a una reserva, registrando cantidad y precio vigente al momento de la reserva. |
| RF7 | Registro de pagos | Se deben poder registrar pagos parciales o totales de una reserva, indicando el monto, la fecha y el método de pago (efectivo, tarjeta, transferencia). |
| RF8 | Consulta de disponibilidad | El sistema debe ofrecer una función que, dado un rango de fechas y un tipo de habitación, muestre qué habitaciones están libres en ese período. |
| RF9 | Facturación al checkout | Al finalizar la estancia (checkout), el sistema debe calcular el total a pagar sumando el costo de las noches más los servicios consumidos, restando los pagos ya realizados, y mostrar un desglose. |
| RF10 | Historial de cambios de reserva | Cada vez que una reserva cambie de estado (creada, confirmada, cancelada, completada), el sistema debe registrar quién hizo el cambio, cuándo y el estado anterior. |

---

## 2. Requerimientos no funcionales (RNF)

Los requerimientos no funcionales describen **cómo** debe ser el sistema en términos de calidad, rendimiento, seguridad, etc. (sin especificar tecnologías).

| ID  | Requerimiento no funcional | Descripción |
|-----|----------------------------|--------------|
| RNF1 | **Rendimiento** | La consulta de disponibilidad para un rango de 7 días debe mostrar resultados en menos de medio segundo, considerando un volumen académico de hasta 100 habitaciones y 1000 reservas históricas. |
| RNF2 | **Concurrencia** | El sistema debe soportar el uso simultáneo de al menos 5 usuarios (por ejemplo, dos recepcionistas y tres consultas web de disponibilidad) sin conflictos ni pérdida de datos. |
| RNF3 | **Consistencia** | No se deben generar situaciones donde una misma habitación aparezca reservada a dos clientes diferentes para las mismas fechas. Tampoco se debe permitir una reserva con fecha de salida anterior a la fecha de entrada. |
| RNF4 | **Disponibilidad** | Para efectos académicos, se espera que el sistema esté disponible durante los horarios de práctica. Se realizarán respaldos de información diaria. |
| RNF5 | **Seguridad** | Solo los usuarios autorizados (recepcionistas y administrador) pueden acceder al sistema. No se almacenarán números completos de tarjetas de crédito; solo el método de pago y el monto. |
| RNF6 | **Escalabilidad** | El diseño de la base de datos debe permitir agregar más habitaciones o nuevos tipos de servicio sin necesidad de rediseñar completamente la estructura. |
| RNF7 | **Mantenibilidad** | Se debe entregar documentación clara: descripción de entidades y relaciones, reglas de negocio, y un diccionario de datos. El código SQL debe estar comentado. |
| RNF8 | **Usabilidad para consultas** | Las operaciones más frecuentes (consultar disponibilidad, crear reserva, generar factura) deben ser fáciles de ejecutar mediante procedimientos o funciones predefinidas, con nombres intuitivos. |

---

## 3. Manejo de errores y casos borde (descripción general)

Se describen situaciones anómalas y cómo el sistema debe reaccionar.

| Caso | Comportamiento esperado del sistema |
|------|--------------------------------------|
| Intentar reservar una habitación ya ocupada en las fechas deseadas | El sistema debe mostrar un mensaje claro: "La habitación [número] no está disponible en esas fechas" y evitar la creación de la reserva. |
| Cancelar una reserva cuando faltan menos de 24 horas para la entrada | El sistema debe permitir la cancelación, pero registrar una penalización simbólica (por ejemplo, cargar el 10% del costo de la primera noche) a modo de política del hotel. |
| Registrar un pago por un monto mayor al saldo pendiente de la reserva | El sistema debe rechazar el pago y mostrar un mensaje: "El monto excede el saldo pendiente. Saldo actual: $X". |
| Realizar el checkout sin haber pagado el total | El sistema debe generar la factura con el monto pendiente, marcar la reserva como "con adeudo" y permitir imprimir un comprobante de deuda. |
| Intentar eliminar un tipo de habitación que aún tiene reservas futuras | El sistema debe impedir la eliminación y notificar: "No se puede eliminar este tipo de habitación porque existen reservas activas asociadas". En su lugar, se puede desactivar el tipo para nuevas reservas. |
| Ingresar fechas de reserva inválidas (salida antes que entrada) | El sistema debe detectar el error y solicitar corregir las fechas antes de guardar. |
| Fallo de conexión mientras se está haciendo una reserva | El sistema debe cancelar la operación incompleta y notificar al usuario. Se debe poder reintentar la operación sin duplicar datos. |

---

**Nota:** Este documento es una especificación previa al diseño técnico. En la siguiente fase se definirán las entidades, atributos, relaciones y restricciones según el gestor de base de datos que se elija.
