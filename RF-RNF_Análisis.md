# Documento de Análisis de Requerimientos Funcionales y No Funcionales
## Sistema de Gestión Universitaria Multi-Inquilino (SaaS Auto-Servicios)

---

### 1. Introducción y Alcance del Proyecto

El proyecto seleccionado consiste en el diseño e implementación de una base de datos para un **sistema multi-inquilino (multi-tenant)** bajo el modelo de **esquema compartido**. La plataforma está orientada a la gestión de procesos académicos y administrativos de múltiples instituciones de educación superior de forma simultánea.

El núcleo funcional emula las operaciones de un sistema de **auto-servicios universitario** (gestión de matrículas, inscripciones, asignación de materias, consulta de horarios y calificaciones), garantizando que el acceso masivo de usuarios finales (estudiantes y docentes) ocurra bajo un entorno aislado, seguro y de alta disponibilidad.

---

### 2. Requerimientos Funcionales de la Base de Datos (RF-BD)

Estos requerimientos definen las funciones implícitas, restricciones de integridad y comportamiento operativo que el motor de base de datos (SGBD) debe ejecutar y garantizar de manera nativa.

#### RF-BD01: Aislamiento Estricto de Inquilinos
* **Descripción:** El SGBD debe segregar la información de cada institución de forma transparente. Toda consulta, inserción o modificación de datos realizada por un usuario final debe restringirse automáticamente al identificador de su respectiva universidad (`id_universidad`).
* **Regla de integridad:** Ninguna sentencia de manipulación de datos (DML) ejecutada por un inquilino debe tener visibilidad sobre los registros de otro.

#### RF-BD02: Consistencia Relacional Inter-Entidad
* **Descripción:** El esquema debe impedir cruces de información inválidos entre distintas instituciones a través de restricciones de llave foránea (`FOREIGN KEY`).
* **Regla de integridad:** El sistema debe rechazar cualquier intento de inscribir a un estudiante de la *Universidad A* en una sección académica o materia perteneciente a la *Universidad B*.

#### RF-BD03: Preservación Histórica mediante Borrado Lógico
* **Descripción:** Los registros transaccionales críticos (como actas de calificaciones o historiales de reinscripción) no deben eliminarse físicamente del almacenamiento.
* **Regla de integridad:** El motor debe interceptar las sentencias `DELETE` sobre tablas nucleares y transformarlas en operaciones `UPDATE` que modifiquen un estado lógico (`activo = FALSE`), preservando la integridad referencial histórica.

#### RF-BD04: Automatización de Auditoría y Trazabilidad
* **Descripción:** Toda transacción ejecutada en el auto-servicios debe registrar de forma obligatoria su metadato de creación y modificación para fines de auditoría interna.
* **Regla de integridad:** El SGBD debe poblar de manera automática los campos `fecha_registro`, `ultima_modificacion` y `usuario_auditoria` utilizando funciones nativas de tiempo del servidor (`CURRENT_TIMESTAMP`).

---

### 3. Requerimientos No Funcionales de la Base de Datos (RNF-BD)

Estos requerimientos determinan las restricciones técnicas, atributos de calidad, rendimiento y mecanismos de seguridad estructural del motor de datos.

#### RNF-BD01: Seguridad Dinámica a Nivel de Fila (Row-Level Security)
* **Criterio técnico:** El aislamiento multi-tenant no debe depender exclusivamente de las cláusulas `WHERE` del backend. El motor de datos (PostgreSQL) debe tener habilitadas políticas de **Row-Level Security (RLS)** activas en todas las tablas operativas.
* **Métrica:** El acceso no autorizado a filas de otro inquilino mediante consola directa debe ser bloqueado a nivel de núcleo del SGBD.

#### RNF-BD02: Eficiencia de Consulta bajo Alta Concurrencia
* **Criterio técnico:** Durante los periodos de reinscripción (fase crítica de auto-servicios), el esquema debe responder eficientemente ante miles de peticiones de lectura simultáneas.
* **Métrica:** Se deben implementar **índices B-Tree compuestos** en donde la primera columna de indexación sea el `id_universidad`. Las consultas de búsqueda de estudiantes o materias deben resolverse en un tiempo menor o igual a **200 milisegundos**.

#### RNF-BD03: Garantía de Propiedades ACID en Inscripciones
* **Criterio técnico:** El proceso de selección de asignaturas y asignación de cupos debe ser completamente atómico para evitar la sobreventa de lugares en secciones académicas.
* **Métrica:** Toda transacción de inscripción debe ejecutarse dentro de bloques de control de concurrencia estrictos (`BEGIN TRANSACTION ... COMMIT`), asegurando un aislamiento de tipo *Serializable* o *Read Committed* según la criticidad del proceso.

#### RNF-BD04: Escalabilidad Estructural y Particionamiento
* **Criterio técnico:** El volumen de datos generado por el historial académico de múltiples universidades puede degradar el rendimiento a largo plazo.
* **Métrica:** Las tablas que almacenen registros históricos de accesos, kárdex o logs transaccionales que superen el millón de filas deben implementar **particionamiento nativo** por rangos de fechas o por el método *Hash* del inquilino.

---

### 4. Estructura Base del Modelo Relacional (Snippet DDL)

Como demostración del cumplimiento de los requerimientos de aislamiento (`id_universidad`) y auditoría, se define el siguiente estándar de estructura lógica para PostgreSQL:

```sql
-- Catálogo de Universidades (Tenants)
CREATE TABLE universidad (
    id_universidad SERIAL PRIMARY KEY,
    nombre_institucion VARCHAR(150) NOT NULL,
    codigo_interno VARCHAR(20) UNIQUE NOT NULL,
    creado_el TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Entidad Estudiante con Atributos Multi-Tenant y Auditoría
CREATE TABLE estudiante (
    id_estudiante SERIAL PRIMARY KEY,
    id_universidad INT NOT NULL,
    matricula VARCHAR(30) NOT NULL,
    nombre_completo VARCHAR(150) NOT NULL,
    correo_institucional VARCHAR(100) NOT NULL,
    estatus_logico BOOLEAN DEFAULT TRUE NOT NULL,
    actualizado_el TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Restricciones de Integridad
    CONSTRAINT fk_universidad_estudiante 
        FOREIGN KEY (id_universidad) 
        REFERENCES universidad(id_universidad) 
        ON DELETE RESTRICT,
    CONSTRAINT uq_matricula_por_universidad 
        UNIQUE (id_universidad, matricula)
);