**Autor:** Angel Uriel Monterrosas Gonzalez  
**Matrícula:** 202339872  
**Institución:** Benemérita Universidad Autónoma de Puebla (BUAP)  
**Estudios:** Licenciatura en Ingeniería en Ciencias de la Computación  

# Documento Exhaustivo de Análisis de Requerimientos
## Sistema Integral de Gestión Clínica para Consultorio Dental

---

### 1. Introducción y Alcance del Proyecto

Este documento detalla los requerimientos para el desarrollo de un **sistema de gestión de uso exclusivo (single-tenant)** orientado a un consultorio dental. El objetivo es digitalizar y centralizar el flujo de trabajo clínico, administrativo y financiero de la clínica. 

El alcance del sistema abarca desde la programación de citas y control de acceso, hasta la **gestión avanzada del expediente clínico electrónico (ECE)**, incluyendo la representación de odontogramas. Se prioriza la integridad de los datos médicos y la eficiencia operativa.

La arquitectura debe soportar un entorno local o de nube privada, garantizando la confidencialidad del paciente bajo marcos normativos de salud (como la NOM-024-SSA3-2012 en México para expedientes electrónicos).

---

### 2. Desarrollo: Requerimientos Funcionales (RF)

Los Requerimientos Funcionales definen las acciones específicas, módulos y comportamientos que el sistema debe ejecutar para satisfacer las necesidades del flujo de trabajo dental.

| ID | Módulo | Nombre | Descripción | Criterio de Aceptación |
| :--- | :--- | :--- | :--- | :--- |
| **RF-01** | **Agenda** | Control de Solapamiento | El sistema debe gestionar el calendario bloqueando horarios ya ocupados. | Al intentar agendar en un bloque de tiempo ocupado, el sistema lanzará un error y sugerirá el horario libre más cercano. |
| **RF-02** | **Clínica** | Odontograma Digital | Representación gráfica interactiva de la dentadura del paciente (adulto e infantil). | El odontólogo podrá marcar estados (caries, extracciones, implantes) por cada pieza dental y guardar el histórico visual. |
| **RF-03** | **Clínica** | Expediente Médico | Registro de antecedentes, alergias, diagnósticos y notas de evolución. | El expediente no permitirá la eliminación de notas previas (solo anexos o correcciones con marca de tiempo). |
| **RF-04** | **Finanzas** | Gestión de Presupuestos | Generación de cotizaciones desglosadas por tratamiento y su conversión a cuentas por cobrar. | El sistema calculará el total automáticamente y permitirá registrar pagos parciales (abonos) hasta liquidar el saldo. |
| **RF-05** | **Inventario** | Control de Insumos | Registro de entrada y salida de materiales de uso clínico (ej. resinas, anestesia). | Descuento automático del stock al registrar un tratamiento completado y alertas visuales de inventario bajo. |
| **RF-06** | **Seguridad** | Gestión de Roles (RBAC) | Control de acceso basado en tres perfiles: Administrador, Odontólogo y Recepcionista. | Un usuario "Recepcionista" podrá agendar citas y cobrar, pero tendrá bloqueado el acceso a la edición del diagnóstico clínico. |

---

### 3. Desarrollo: Requerimientos No Funcionales (RNF)

Los Requerimientos No Funcionales establecen los atributos de calidad, rendimiento, restricciones de seguridad y estándares técnicos que la arquitectura del sistema debe cumplir.

| ID | Atributo | Nombre | Descripción | Métrica / Estándar |
| :--- | :--- | :--- | :--- | :--- |
| **RNF-01** | **Seguridad** | Cifrado de Datos Sensibles | La información personal y diagnóstica debe almacenarse de forma segura para prevenir fugas de información. | Cifrado **AES-256** en reposo para datos médicos e implementación de protocolo **TLS 1.3** para datos en tránsito. |
| **RNF-02** | **Rendimiento** | Tiempos de Respuesta | La interfaz de usuario debe reaccionar de manera fluida frente a operaciones cotidianas de lectura. | Las consultas al catálogo de pacientes o carga de la agenda diaria deben resolverse en **menos de 1.5 segundos**. |
| **RNF-03** | **Auditoría** | Trazabilidad Inmutable | Toda acción crítica (cobros, edición de expediente, cancelación de citas) debe quedar registrada. | Existencia de una tabla *Log* transaccional que guarde `ID_Usuario`, `Acción`, `Fecha_Hora` y `Dirección_IP`, sin permisos de borrado. |
| **RNF-04** | **Usabilidad** | Diseño Responsivo | El sistema debe ser operable desde distintos dispositivos dentro del consultorio. | La interfaz debe adaptarse sin pérdida de funcionalidad a monitores de escritorio (1080p) y tablets (mínimo 768px de ancho). |
| **RNF-05** | **Disponibilidad** | Tolerancia a Fallos locales | La clínica no puede detener sus operaciones ante caídas temporales del servidor principal. | Implementación de **backups automáticos incrementales** cada 12 horas hacia un almacenamiento físico secundario o en la nube. |
| **RNF-06** | **Legal** | Cumplimiento Normativo | El software debe alinearse con las leyes vigentes de protección de datos de salud en el país. | Cumplimiento de estándares de confidencialidad médica e interoperabilidad (ej. estructura de ECE compatible con NOM-024). |

---

### 4. Conclusión

El análisis detallado demuestra que el sistema trasciende un simple catálogo de citas. Al integrar herramientas clínicas como el **odontograma digital** y controles estrictos de auditoría y roles, se garantiza una gestión segura y eficiente. 

La implementación rigurosa de los requerimientos funcionales (RF) asegura que la clínica opere sin fricciones, mientras que los requerimientos no funcionales (RNF) protegen la infraestructura contra vulnerabilidades, garantizando un rendimiento óptimo bajo alta concurrencia en la recepción.
