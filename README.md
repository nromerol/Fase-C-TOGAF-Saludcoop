# SaludCoop IPS — Arquitectura Empresarial

Proyecto académico de Arquitectura de Sistemas II de la Universidad Central, desarrollado bajo los principios del marco de arquitectura empresarial TOGAF.

## Equipo

- Violeta Sofía Carrasquilla
- Nicolas Steven Romero León
- Diego Ferney Rojas Romero
- Andres Felipe Correcha Meneses

**Profesor:** Oscar Darío Sánchez Pérez  
**Asignatura:** Arquitectura de Sistemas II  
**Proyecto:** Fase C — Caso de estudio SaludCoop IPS  
**Universidad:** Universidad Central — Bogotá D.C.  
**Año:** 2026

---

## Descripción del proyecto

El proyecto propone una arquitectura empresarial objetivo para SaludCoop IPS con el propósito de solucionar los problemas de fragmentación, integración e interoperabilidad existentes entre sus diferentes sistemas de información.

La arquitectura actual presenta múltiples sistemas independientes y conexiones punto a punto que generan problemas de interoperabilidad, concurrencia, disponibilidad y consistencia de los datos.

La arquitectura propuesta busca evolucionar este escenario hacia un modelo centralizado y gobernado mediante una capa de integración basada en API Gateway y estándares de interoperabilidad en salud.

---

## Problema identificado

La arquitectura actual de SaludCoop IPS presenta cuatro sistemas principales:

- Historia Clínica en Línea (HCL)
- Sistema Hospitalario de Emergencias y Consultas (SHEC)
- Telemedicina
- Agendamiento de Citas

Estos sistemas utilizan diferentes tecnologías y modelos de información.

Entre los principales problemas identificados se encuentran:

- Conexiones punto a punto entre aplicaciones.
- Consultas SQL directas entre sistemas.
- Transferencias nocturnas mediante FTP.
- Fragmentación de la información clínica.
- Incompatibilidad entre diferentes formatos de datos.
- Falta de una fuente única de verdad para la identidad del paciente.
- Limitaciones de concurrencia en Telemedicina.
- Falta de una vista clínica unificada.

---

## Arquitectura objetivo

La arquitectura propuesta reemplaza el modelo de múltiples conexiones directas por una arquitectura en estrella gobernada por un **API Gateway**.

```text
                    ┌─────────────────────┐
                    │   API Gateway       │
                    │      APP-07         │
                    └──────────┬──────────┘
                               │
          ┌────────────────────┼────────────────────┐
          │                    │                    │
          ▼                    ▼                    ▼
   Historia Clínica         Telemedicina          SHEC
      APP-01                  APP-03              APP-02
          │
          │
          ▼
        MPI
      APP-05

                    ┌─────────────────────┐
                    │ Visor Clínico Único │
                    │       APP-06       │
                    └─────────────────────┘

                    ┌─────────────────────┐
                    │ Agendamiento APP-04 │
                    └─────────────────────┘
```

El API Gateway actúa como punto central de integración y evita que las aplicaciones establezcan conexiones directas entre ellas.

---

## Componentes principales

### APP-01 — Historia Clínica en Línea

Sistema monolítico existente que será refactorizado para exponer recursos mediante APIs REST/FHIR en lugar de permitir accesos SQL directos.

### APP-02 — SHEC

Sistema de atención domiciliaria que actualmente utiliza almacenamiento local SQLite y sincronización mediante FTP.

La arquitectura objetivo reemplaza este mecanismo por mensajería asíncrona y reconciliación de eventos.

### APP-03 — Telemedicina

Sistema basado en microservicios que será conservado y escalado para solucionar sus limitaciones de conexiones concurrentes.

### APP-04 — Agendamiento

Sistema SaaS encargado de la gestión de citas.

En la arquitectura objetivo consulta al API Gateway para validar el estado del paciente y la disponibilidad real de los servicios.

### APP-05 — Identificador Maestro de Pacientes (MPI)

Nuevo componente encargado de mantener una identidad única para cada paciente.

El MPI constituye la fuente única de verdad para:

- Identidad del paciente.
- Datos demográficos.
- Estado de afiliación.

### APP-06 — Visor Clínico Único

Nueva aplicación web destinada a proporcionar a los profesionales de salud una visión integrada del expediente clínico del paciente.

El visor obtiene la información mediante el API Gateway y no mantiene una copia independiente de los datos clínicos.

### APP-07 — API Gateway

Componente central de integración de la arquitectura objetivo.

Sus principales responsabilidades son:

- Enrutamiento.
- Integración entre sistemas.
- Transformación de información.
- Aplicación de políticas de seguridad.
- Control de tráfico.
- Interoperabilidad mediante HL7/FHIR.

---

## Arquitectura de Datos

La arquitectura de datos propone eliminar la fragmentación entre los diferentes modelos utilizados actualmente.

La arquitectura objetivo utiliza un modelo canónico basado en **HL7/FHIR**.

Los principales recursos contemplados son:

- `Patient`
- `Encounter`
- `Observation`
- `Schedule`

El modelo también incorpora el Identificador Maestro de Pacientes (MPI) como fuente única de verdad.

---

## Seguridad

La arquitectura contempla mecanismos de protección de la información clínica, entre ellos:

- TLS 1.3 para información en tránsito.
- AES-256 para información en reposo.
- Pseudonimización y tokenización para ambientes de prueba.
- Control de acceso.
- Auditoría de transacciones.
- OAuth 2.0 / OpenID Connect.
- Cumplimiento de las políticas de protección de datos aplicables.

---

## Interoperabilidad

La arquitectura utiliza **HL7/FHIR** como estándar para el intercambio de información entre los diferentes sistemas.

Las principales interfaces identificadas son:

### INT-01 — Consulta de paciente

```text
Historia Clínica
      ↓
API Gateway
      ↓
MPI
```

### INT-02 — Validación de agenda

```text
Agendamiento
      ↓
API Gateway
      ↓
Telemedicina / SHEC
```

### INT-03 — Registro de evento

```text
SHEC
 ↓
API Gateway
 ↓
Historia Clínica
```

La utilización de estas interfaces permite eliminar las conexiones directas entre los sistemas.

---

## Principales brechas identificadas

| Elemento | Situación actual | Arquitectura objetivo |
|---|---|---|
| Modelo de datos | Relacional, JSON y SQLite | Modelo canónico HL7/FHIR |
| Identidad | Información fragmentada | MPI |
| Sincronización | FTP nocturno | Mensajería asíncrona |
| Acceso a HCL | SQL directo | REST/FHIR |
| Integración | 6 conexiones punto a punto | API Gateway |
| Gobernanza | Sin matriz formal | Matriz CRUD |
| Visibilidad clínica | Sistemas separados | Visor Clínico Único |
| Seguridad | Controles fragmentados | Cifrado y gobierno centralizado |

---

## Hoja de ruta

La implementación se plantea progresivamente para evitar una migración tipo Big Bang en un entorno de operación 24/7.

### Fase 1 — Capa de integración

Implementación del API Gateway y definición de los contratos de integración.

### Fase 2 — Interoperabilidad

Migración progresiva de las integraciones existentes hacia REST/FHIR.

### Fase 3 — MPI

Implementación del Identificador Maestro de Pacientes.

### Fase 4 — Visor Clínico Único

Desarrollo de una interfaz unificada para los profesionales de salud.

### Fase 5 — Mensajería y sincronización

Migración de los procesos FTP hacia mecanismos de mensajería asíncrona y reconciliación.

### Fase 6 — Escalamiento

Después del piloto inicial, se plantea el escalamiento progresivo de la arquitectura.

---

## Documentación

La carpeta `docs/` contiene la documentación correspondiente a las arquitecturas desarrolladas.

```text
docs/
├── arquitectura-datos/
├── arquitectura-aplicaciones/
└── arquitectura-tecnologica/
```

Los diagramas y matrices utilizados en el proyecto se encuentran organizados en sus respectivas carpetas.

---

## Herramientas

Para la elaboración de los modelos y diagramas se contemplan herramientas como:

- Archi / ArchiMate
- Draw.io
- Lucidchart
- UML
- BPMN

---

## Estado del proyecto

**Estado:** Diseño de arquitectura / Proyecto académico

Este repositorio documenta la arquitectura propuesta para el caso de estudio SaludCoop IPS y no representa una implementación productiva de los sistemas descritos.
