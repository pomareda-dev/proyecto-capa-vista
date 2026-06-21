# Distribución del Equipo — Atlantic City CRM

[<- Volver al README](../README.md)

## 1. Composición del Equipo

| Rol | Perfil | Experiencia | Responsabilidad principal |
|-----|--------|-------------|--------------------------|
| **Tech Lead / Arquitecto** | Senior Developer | Mayor experiencia en el stack | Arquitectura, autenticación, revisión de código, despliegue, integración |
| **Desarrollador Frontend / Clientes** | Junior 1 | Conocimientos básicos de Vue/React | Módulo de gestión de clientes, segmentación, perfil de cliente |
| **Desarrollador Fullstack / Promociones** | Junior 2 | Conocimientos básicos de Node.js | Módulo de promociones y reportes exportables |
| **Desarrollador Fullstack / Atención** | Junior 3 | Conocimientos básicos de JavaScript | Módulo de atención al cliente, tickets, dashboards |

> **Nota**: Los nombres específicos se asignarán al inicio del proyecto. Los roles y responsabilidades aquí definidos son vinculantes independientemente de quién ocupe cada posición.

---

## 2. Responsabilidades por Rol

### Tech Lead (Senior)

**Alcance**: Arquitectura del sistema, infraestructura, integración y calidad.

| Área | Responsabilidades |
|------|-------------------|
| **Arquitectura** | Diseñar y documentar la arquitectura en capas; definir convenciones de código, estructura de carpetas y patrones (servicios, stores, componentes) |
| **Infraestructura** | Inicializar el proyecto (Vite + Vue 3, Express, Prisma); configurar CI/CD; gestionar el esquema de base de datos y migraciones |
| **Autenticación** | Implementar login, JWT, middleware RBAC, gestión de roles por gerencia |
| **Revisión de código** | Revisar todos los Pull Requests; asegurar calidad, consistencia y cobertura de tests |
| **Integración** | Resolver conflictos entre módulos; garantizar que la API y el frontend se comuniquen correctamente |
| **Despliegue** | Configurar y ejecutar despliegues a los entornos de prueba y producción |
| **Mentoría** | Pair programming con juniors las primeras 2 semanas; sesiones de dudas diarias (15 min) |
| **Testing E2E** | Diseñar y ejecutar pruebas end-to-end de flujos críticos |

### Junior 1 — Módulo de Clientes

**Alcance**: Toda la funcionalidad de gestión de perfiles y segmentación de clientes.

| Área | Responsabilidades |
|------|-------------------|
| **Frontend** | Componentes Vue: lista de clientes, detalle con tabs, formulario de creación/edición, filtros de segmentación |
| **Backend** | Client Service (CRUD), Client Controller, validación de entrada, lógica de deduplicación |
| **Base de datos** | Modelos Client, Segment, Visit (dentro del esquema Prisma definido por el Senior) |
| **Testing** | Tests unitarios de componentes Client; tests de servicio de Client Service |
| **Estado** | Pinia store de clientes (listado, detalle, filtros) |

### Junior 2 — Módulo de Promociones y Reportes

**Alcance**: CRUD de promociones, asignación a segmentos, y reportes exportables.

| Área | Responsabilidades |
|------|-------------------|
| **Frontend** | Componentes Vue: crear/editar promoción, asignar a segmentos, calendario de vigencia, listado de promociones |
| **Backend** | Promotion Service (CRUD), asignación a segmentos, cálculo de métricas de efectividad |
| **Base de datos** | Modelos Promotion, ClientPromotion, SegmentPromotion |
| **Testing** | Tests unitarios de componentes Promotion; tests de Promotion Service |
| **Reportes** | Endpoint de exportación (CSV/PDF); formulario de filtros de reporte |
| **Estado** | Pinia store de promociones |

### Junior 3 — Módulo de Atención al Cliente y Dashboards

**Alcance**: Sistema de tickets/quejas y visualización de KPIs.

| Área | Responsabilidades |
|------|-------------------|
| **Frontend** | Componentes Vue: lista de tickets, formulario de nuevo ticket, historial de interacciones, dashboard de KPIs, gráficos de actividad |
| **Backend** | Ticket Service (CRUD, comentarios, escalamiento), Report Service (agregaciones para dashboards) |
| **Base de datos** | Modelos Ticket, TicketComment |
| **Testing** | Tests unitarios de componentes Ticket y Dashboard; tests de Ticket Service |
| **Dashboards** | Componentes de gráficos (actividad, efectividad, tiempos de respuesta) |
| **Estado** | Pinia store de tickets y dashboards |

---

## 3. Estructura de Carpetas del Proyecto

```
atlantic-city-crm/
├── client/                          # Frontend - Vue 3
│   ├── src/
│   │   ├── assets/                  # Estilos, imágenes, íconos
│   │   ├── components/              # Componentes compartidos
│   │   │   ├── ui/                  # Botones, inputs, modales, etc.
│   │   │   └── layout/              # Sidebar, Header, AppShell
│   │   ├── modules/                 # Módulos por dominio
│   │   │   ├── auth/                # — Senior
│   │   │   │   ├── components/
│   │   │   │   ├── views/
│   │   │   │   └── composables/
│   │   │   ├── clients/             # — Junior 1
│   │   │   │   ├── components/
│   │   │   │   ├── views/
│   │   │   │   └── composables/
│   │   │   ├── promotions/          # — Junior 2
│   │   │   │   ├── components/
│   │   │   │   ├── views/
│   │   │   │   └── composables/
│   │   │   ├── tickets/             # — Junior 3
│   │   │   │   ├── components/
│   │   │   │   ├── views/
│   │   │   │   └── composables/
│   │   │   └── reports/             # — Junior 2 + Junior 3
│   │   │       ├── components/
│   │   │       ├── views/
│   │   │       └── composables/
│   │   ├── router/                  # Rutas protegidas por rol
│   │   ├── stores/                  # Pinia stores
│   │   ├── services/                # Llamadas HTTP a la API
│   │   └── utils/                   # Helpers compartidos
│   ├── tests/                       # Tests unitarios del frontend
│   └── ...                          # vite.config, package.json, etc.
├── server/                          # Backend - Express
│   ├── src/
│   │   ├── config/                  # DB, JWT, env
│   │   ├── middleware/               # Auth, RBAC, error handler
│   │   ├── modules/                  # Módulos por dominio
│   │   │   ├── auth/                # — Senior
│   │   │   ├── clients/             # — Junior 1
│   │   │   ├── promotions/          # — Junior 2
│   │   │   ├── tickets/             # — Junior 3
│   │   │   └── reports/             # — Junior 2 + Junior 3
│   │   ├── shared/                  # Validaciones, helpers, tipos
│   │   └── app.js                   # Entry point Express
│   ├── prisma/
│   │   └── schema.prisma           # Esquema de base de datos
│   ├── tests/                       # Tests del backend
│   └── ...                          # package.json, etc.
├── docs/                            # Documentación del proyecto
└── README.md
```

Cada desarrollador trabaja dentro de sus módulos asignados. La carpeta `shared/` (componentes UI compartidos, validaciones, helpers) es responsabilidad del Senior y se coordina mediante PRs.

---

## 4. Distribución de Tareas por Iteración

### Iteración 1 — Fundación y Módulo de Clientes (Semanas 1-3)

| Tarea | Responsable | Detalle |
|-------|-------------|---------|
| Inicializar proyecto (Vite + Vue 3, Express, Prisma, MySQL) | Senior | Scaffold completo, configuración de linting, CI/CD |
| Diseñar esquema de base de datos | Senior | Archivo `schema.prisma` con todas las entidades |
| App Shell: Layout, Sidebar, Header, Router | Junior 1 | Con supervisión del Senior en la primera semana |
| Auth: Login, logout, JWT, middleware RBAC | Senior | Backend: auth service, middleware. Frontend: login view, router guards |
| Client CRUD: creación, listado, edición, búsqueda | Junior 1 | Frontend: formularios, tabla paginada. Backend: service + controller |
| Client Detail: vista con tabs (datos, visitas) | Junior 1 | Frontend: componente de detalle con navegación por tabs |
| Validación de deduplicación (DNI/email) | Junior 1 | Backend: validación en el service; Frontend: feedback en el formulario |
| Segmentación: filtros por frecuencia, gasto, preferencias | Junior 2 | Frontend: componente de filtros; Backend: query params en el endpoint |
| Tests unitarios de Auth | Senior | Jest para backend; Vitest para frontend |
| Tests unitarios de Clients | Junior 1 | Cubrir servicios y componentes principales |

### Iteración 2 — Promociones y Atención al Cliente (Semanas 4-6)

| Tarea | Responsable | Detalle |
|-------|-------------|---------|
| Promotion CRUD: crear, editar, desactivar | Junior 2 | Frontend completo + Backend service |
| Asignación de promociones a segmentos | Junior 2 | Interfaz de asignación + lógica de validación cruzada |
| Vigencia de promociones (fechas, estados) | Junior 2 | Lógica de inicio/fin de promociones |
| Ticket CRUD: crear queja/solicitud | Junior 3 | Frontend: formulario + lista. Backend: service + controller |
| Comentarios en tickets | Junior 3 | Línea de tiempo de interacciones por ticket |
| Escalamiento de tickets por reglas de negocio | Junior 3 | Reglas: monto, prioridad → asignación automática de responsable |
| Integración en Client Detail: tabs de promociones y tickets | Junior 1 | Conectar componentes de Junior 2 y Junior 3 en la vista de detalle |
| Tests unitarios de Promotions | Junior 2 | Cubrir servicios y componentes |
| Tests unitarios de Tickets | Junior 3 | Cubrir servicios y componentes |

### Iteración 3 — Reportes, Dashboards y Refinamiento (Semanas 7-9)

| Tarea | Responsable | Detalle |
|-------|-------------|---------|
| Dashboard de KPIs: clientes activos, conversiones, tiempos | Junior 3 | Frontend: gráficos y tarjetas de métricas; Backend: endpoints de agregación |
| Gráficos de actividad por gerencia y tendencias | Junior 3 | Librería de gráficos (Chart.js o similar) |
| Reportes exportables (CSV/PDF) | Junior 2 | Backend: generación de archivos; Frontend: botón de exportación |
| Refinamiento de UX basado en pruebas con usuario | Todos | Ajustes de flujo, validaciones, mensajes de error |
| Pruebas de integración E2E (Cypress/Playwright) | Senior | Flujos críticos: login, crear cliente, crear ticket, asignar promoción |
| Documentación técnica | Todos | Cada quien documenta su módulo; Senior revisa |
| Manual de usuario | Todos | Capturas de pantalla y flujos para cada gerencia |
| Despliegue a producción | Senior | Configuración final del servidor, variables de entorno, migraciones |

---

## 5. Convenciones de Trabajo

### 5.1 Flujo de desarrollo (Git)

```
main ─────────────────────────────────── (producción)
  └── develop ────────────────────────── (integración)
       ├── feature/auth-login ────────── (Senior)
       ├── feature/client-crud ───────── (Junior 1)
       ├── feature/promotion-crud ────── (Junior 2)
       └── feature/ticket-crud ────────── (Junior 3)
```

- **Rama base**: `develop`. Todas las features se mergean primero a `develop`.
- **Nomenclatura**: `feature/<módulo>-<descripción>` (ej: `feature/client-search-filters`).
- **Pull Requests**: Todo cambio va por PR. El Senior revisa todos los PRs. Los juniors revisan al menos 1 PR por semana de otro compañero (para practicar lectura de código).
- **Commits**: Seguir [Conventional Commits](https://www.conventionalcommits.org/) (ej: `feat(clients): add search filters`, `fix(auth): prevent token expiration`).

### 5.2 Definición de Hecho (Definition of Done)

Una tarea se considera **hecha** cuando:

1. ✅ El código está en una rama `feature/` y el PR está creado.
2. ✅ Todos los tests unitarios pasan (cobertura ≥ 70% del módulo).
3. ✅ El Senior aprobó el PR (al menos 1 aprobación).
4. ✅ No hay errores de linting.
5. ✅ La funcionalidad fue verificada manualmente en el navegador.
6. ✅ Los casos borde discutidos en la tarea están cubiertos.

### 5.3 Comunicación del equipo

| Práctica | Frecuencia | Participantes |
|----------|-----------|---------------|
| **Daily standup** | Diaria (15 min) | Todos |
| **Sprint planning** | Inicio de cada iteración | Todos |
| **Sprint review + retrospective** | Fin de cada iteración | Todos |
| **Pair programming** | Primeras 2 semanas (según necesidad) | Senior + Junior |
| **Code review** | Continuo (máximo 24h de respuesta) | Senior (revisa todos), Juniors (revisan 1+ por semana) |

---

## 6. Matriz de Responsabilidades RACI

Leyenda: **R** = Responsable (hace), **A** = Aprobador (decide), **C** = Consultado, **I** = Informado

| Actividad | Senior | Junior 1 | Junior 2 | Junior 3 |
|-----------|--------|----------|----------|----------|
| Arquitectura del sistema | **R/A** | C | C | C |
| Inicialización del proyecto | **R/A** | I | I | I |
| Diseño de esquema de base de datos | **R/A** | C | C | C |
| Módulo Auth | **R/A** | I | I | I |
| Módulo Clientes | A | **R** | I | I |
| Módulo Promociones | A | I | **R** | I |
| Módulo Tickets/Atención | A | I | I | **R** |
| Dashboards y Reportes | A | I | R | **R** |
| Tests E2E | **R/A** | C | C | C |
| Revisión de PRs | **R/A** | C | C | C |
| Despliegue | **R/A** | I | I | I |
| Documentación | A | **R** | **R** | **R** |

---

## 7. Criterios de Éxito del Equipo

| Criterio | Métrica | Objetivo |
|----------|---------|----------|
| Cobertura de tests | % de líneas cubiertas por tests unitarios | ≥ 70% por módulo |
| Tiempo de respuesta en PRs | Horas desde la creación hasta la aprobación | ≤ 24 horas |
| Defectos por iteración | Bugs encontrados en review | < 5 por iteración |
| Adopción de convenciones | Commits siguiendo Conventional Commits | 100% |
| Completitud de módulos | Funcionalidades implementadas vs. planificadas | ≥ 90% por iteración |

---

## 8. Referencias

- [Propuesta de Desarrollo](propuesta_desarrollo.md)
- [Problema de Atlantic City](problema_atlantic_city.md)
- [Análisis del Problema Estructural](analisis_problema_estructural.md)
- [FODA de Atlantic City](foda_atlantic_city.md)
- [Análisis de Riesgo](analisis_de_riesgo.md)
- [Rúbrica de Evaluación](rubrica_evaluacion_parcial_proyecto_capa_vista.md)