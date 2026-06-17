# Documento de Análisis de Requerimientos Funcionales y No Funcionales
## Sistema de Gestión Universitaria Multi-Inquilino (SaaS Auto-Servicios)

---

### 1. Introducción y Alcance del Proyecto

El proyecto seleccionado consiste en el diseño e implementación de una base de datos para un **sistema multi-inquilino (multi-tenant)** bajo el modelo de **esquema compartido**. La plataforma está orientada a la gestión de procesos académicos y administrativos de múltiples instituciones de educación superior de forma simultánea.

El núcleo funcional emula las operaciones de un sistema de **auto-servicios universitario** (gestión de matrículas, inscripciones, asignación de materias, consulta de horarios y calificaciones). Garantiza que el acceso masivo de usuarios finales ocurra bajo un entorno aislado, seguro y de alta disponibilidad.

---

### 2. Requerimientos Funcionales del Sistema (RF-SIS)

Estos requerimientos definen las acciones y servicios que el sistema de software proporcionará a los usuarios finales (estudiantes, docentes y administradores) a través de la interfaz.

#### RF-SIS01: Autenticación y Enrutamiento Multi-Tenant
* **Descripción:** El sistema debe autenticar a los usuarios y enrutarlos automáticamente al entorno de su institución.
* **Criterio de aceptación:** Al ingresar el correo institucional, el sistema debe identificar el dominio, resolver el identificador de la universidad y cargar la configuración visual y académica correspondiente.

#### RF-SIS02: Auto-Servicios Estudiantiles (Kárdex y Horarios)
* **Descripción:** El sistema debe permitir a los estudiantes consultar su avance académico (kárdex) y su horario de clases activo.
* **Criterio de aceptación:** El estudiante podrá visualizar un listado de materias aprobadas, reprobadas y en curso, así como descargar un documento PDF con su horario semanal.

#### RF-SIS03: Proceso de Inscripción de Asignaturas
* **Descripción:** El sistema debe habilitar una interfaz de selección de materias durante los periodos de alta y baja académica.
* **Criterio de aceptación:** El usuario podrá buscar materias, verificar cupos disponibles en tiempo real y registrarse en las secciones habilitadas para su programa de estudios.

#### RF-SIS04: Administración Institucional Delegada
* **Descripción:** Cada universidad debe contar con un panel de control exclusivo para sus administradores.
* **Criterio de aceptación:** Los administradores podrán gestionar (CRUD) catálogos de materias, docentes y periodos escolares, afectando únicamente a su institución.

---

### 3. Requerimientos No Funcionales del Sistema (RNF-SIS)

Estos requerimientos establecen los atributos de calidad, rendimiento de la aplicación y restricciones tecnológicas a nivel de la arquitectura de software.

#### RNF-SIS01: Interfaz de Usuario Responsiva (Compatibilidad)
* **Criterio técnico:** El cliente web debe ser completamente responsivo para garantizar el acceso desde dispositivos móviles.
* **Métrica:** La interfaz gráfica debe renderizarse correctamente en resoluciones desde 320px (móviles) hasta pantallas de escritorio, cumpliendo con los estándares de accesibilidad WCAG.

#### RNF-SIS02: Tiempo de Respuesta de la Interfaz (Rendimiento)
* **Criterio técnico:** Las peticiones desde el cliente hacia el servidor backend no deben superar los tiempos de espera aceptables para una experiencia fluida.
* **Métrica:** El tiempo máximo de respuesta para operaciones de lectura (ej. cargar el kárdex) debe ser menor a **2 segundos** bajo una carga normal de usuarios.

#### RNF-SIS03: Disponibilidad del Servicio (Fiabilidad)
* **Criterio técnico:** La plataforma web y los servicios de backend deben mantener una operatividad constante, especialmente durante periodos críticos de inscripción.
* **Métrica:** El sistema debe garantizar un *Uptime* (tiempo de actividad) del **99.9%** mensual, utilizando estrategias de balanceo de carga en la arquitectura del servidor.

#### RNF-SIS04: Arquitectura Basada en API REST (Interoperabilidad)
* **Criterio técnico:** La comunicación entre el *frontend* y el *backend* debe estar completamente desacoplada.
* **Métrica:** Todo intercambio de datos se realizará exclusivamente a través de una **API RESTful**, retornando las respuestas en formato JSON estándar para permitir futuras integraciones con aplicaciones móviles.

---

### 4. Requerimientos Funcionales de la Base de Datos (RF-BD)

Estos requerimientos definen las funciones implícitas, restricciones de integridad y comportamiento operativo que el motor de base de datos (SGBD) debe ejecutar y garantizar de manera nativa.

#### RF-BD01: Aislamiento Estricto de Inquilinos
* **Descripción:** El SGBD debe segregar la información de cada institución de forma transparente. Toda transacción debe restringirse automáticamente al identificador de su respectiva universidad (`id_universidad`).
* **Regla de integridad:** Ninguna sentencia de manipulación de datos (DML) ejecutada por un inquilino debe tener visibilidad sobre los registros de otro.

#### RF-BD02: Consistencia Relacional Inter-Entidad
* **Descripción:** El esquema debe impedir cruces de información inválidos entre distintas instituciones a través de restricciones de llave foránea (`FOREIGN KEY`).
* **Regla de integridad:** El sistema debe rechazar cualquier intento de inscribir a un estudiante de la *Universidad A* en una sección académica perteneciente a la *Universidad B*.

#### RF-BD03: Preservación Histórica mediante Borrado Lógico
* **Descripción:** Los registros transaccionales críticos no deben eliminarse físicamente del almacenamiento.
* **Regla de integridad:** El motor debe interceptar las sentencias `DELETE` sobre tablas nucleares y transformarlas en operaciones `UPDATE` que modifiquen un estado lógico (`activo = FALSE`).

#### RF-BD04: Automatización de Auditoría y Trazabilidad
* **Descripción:** Toda transacción ejecutada debe registrar de forma obligatoria su metadato de creación y modificación para auditoría.
* **Regla de integridad:** El SGBD debe poblar de manera automática los campos `fecha_registro` y `ultima_modificacion` utilizando funciones nativas (`CURRENT_TIMESTAMP`).

---

### 5. Requerimientos No Funcionales de la Base de Datos (RNF-BD)

Estos requerimientos determinan las restricciones técnicas, atributos de calidad, rendimiento y mecanismos de seguridad estructural del motor de datos.

#### RNF-BD01: Seguridad Dinámica a Nivel de Fila (Row-Level Security)
* **Criterio técnico:** El aislamiento multi-tenant no debe depender exclusivamente del backend. PostgreSQL debe tener políticas de **Row-Level Security (RLS)** activas.
* **Métrica:** El acceso no autorizado a filas de otro inquilino mediante consola directa debe ser bloqueado a nivel de núcleo del SGBD.

#### RNF-BD02: Eficiencia de Consulta bajo Alta Concurrencia
* **Criterio técnico:** El esquema debe responder eficientemente ante miles de peticiones de lectura simultáneas durante inscripciones.
* **Métrica:** Se deben implementar **índices B-Tree compuestos** iniciando con el `id_universidad`. Las búsquedas deben resolverse en un tiempo menor o igual a **200 milisegundos**.

#### RNF-BD03: Garantía de Propiedades ACID en Inscripciones
* **Criterio técnico:** El proceso de selección de asignaturas y asignación de cupos debe ser completamente atómico para evitar la sobreventa.
* **Métrica:** Toda inscripción debe ejecutarse dentro de bloques de concurrencia estrictos (`BEGIN ... COMMIT`), asegurando un aislamiento *Read Committed*.

#### RNF-BD04: Escalabilidad Estructural y Particionamiento
* **Criterio técnico:** El volumen de datos del historial académico de múltiples universidades puede degradar el rendimiento a largo plazo.
* **Métrica:** Las tablas que superen el millón de filas (ej. logs transaccionales) deben implementar **particionamiento nativo** por rangos de fechas o por método *Hash*.

---

### 6. Estructura Base del Modelo Relacional (Snippet DDL)

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