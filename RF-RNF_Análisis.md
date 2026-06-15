markdown_content = """# Documento de Análisis de Requerimientos Funcionales y No Funcionales
## Sistema Centralizado de Gestión para Consultorio Dental

**Autor:** Angel Uriel Monterrosas Gonzalez  
**Matrícula:** 202339872  
**Institución:** Benemérita Universidad Autónoma de Puebla (BUAP)  
**Estudios:** Licenciatura en Ingeniería en Ciencias de la Computación  

---

### 1. Introducción y Alcance del Proyecto

El proyecto consiste en el diseño e implementación de una base de datos relacional orientada a la **gestión integral de un consultorio dental de uso exclusivo (single-tenant)**. El sistema centraliza la información para un único entorno administrativo, eliminando la complejidad de esquemas compartidos.

El núcleo funcional abarca el control de la agenda diaria, la administración de expedientes clínicos, el catálogo de tratamientos y el registro de pagos. El objetivo es garantizar un **acceso rápido, consistente y seguro** a los datos de los pacientes, optimizando los tiempos de atención y asegurando la integridad del historial médico.

---

### 2. Requerimientos Funcionales de la Base de Datos (RF-BD)

Estos requerimientos definen las funciones implícitas y reglas de negocio que el Sistema Gestor de Base de Datos (SGBD) debe ejecutar de forma nativa para mantener la coherencia del consultorio.

| ID | Nombre | Descripción | Regla de Integridad |
| :--- | :--- | :--- | :--- |
| **RF-BD01** | Prevención de Solapamiento de Citas | El SGBD debe evitar que se agenden dos pacientes en el mismo horario. | Restricciones a nivel de tabla (`EXCLUDE` o triggers) que rechacen la inserción si hay superposición de `hora_inicio` y `hora_fin` en una misma fecha. |
| **RF-BD02** | Unicidad y Validación de Pacientes | El sistema debe evitar la creación de expedientes duplicados para una misma persona. | Validación obligatoria de unicidad mediante restricciones `UNIQUE` en campos clave como `telefono` o `email`. |
| **RF-BD03** | Preservación Histórica mediante Borrado Lógico | Los expedientes médicos y registros de citas pasadas o canceladas no deben eliminarse físicamente por normatividad legal. | El motor requiere el uso de un campo de estado (`estatus` o `activo`) para ocultar registros de las vistas operativas sin romper la integridad referencial. |
| **RF-BD04** | Automatización de Trazabilidad | Todo registro de evolución clínica o agendamiento debe documentar su creación de forma automática. | El SGBD debe poblar automáticamente campos como `fecha_creacion` y `ultima_actualizacion` usando la función nativa `CURRENT_TIMESTAMP`. |

---

### 3. Requerimientos No Funcionales de la Base de Datos (RNF-BD)

Estos requerimientos determinan las restricciones técnicas, la eficiencia operativa y los estándares de seguridad que la estructura debe cumplir.

| ID | Nombre | Criterio Técnico | Métrica |
| :--- | :--- | :--- | :--- |
| **RNF-BD01** | Eficiencia en Consultas de Agenda Diaria | La recuperación de citas para el día en curso debe ser inmediata por alta frecuencia de uso. | Implementación de un índice B-Tree sobre `fecha_cita`. Las consultas de búsqueda diaria deben resolverse en un tiempo menor o igual a **50 milisegundos**. |
| **RNF-BD02** | Propiedades ACID en Transacciones | La creación de una cita y su pago o anticipo asociado deben tratarse como una única unidad lógica de trabajo atómica. | Uso estricto de bloques `BEGIN TRANSACTION ... COMMIT`. Si falla el registro del pago, la cita no se confirma. El aislamiento mínimo debe ser *Read Committed*. |
| **RNF-BD03** | Integridad Relacional Estricta | El esquema debe impedir por completo la existencia de registros "huérfanos" en el sistema. | Forzar llaves foráneas (`FOREIGN KEY`) con políticas `ON DELETE RESTRICT` para evitar eliminaciones accidentales de catálogos base (como tratamientos). |

---

