# Propuesta de Desarrollo — Atlantic City CRM

[<- Volver al README](../README.md)

## 1. Resumen Ejecutivo

Atlantic City enfrenta un problema estructural de fragmentación de información across sus 6 gerencias y más de 1,200 empleados. Los sistemas actuales operan de forma aislada, generando duplicidad de datos, procesos manuales redundantes y una incapacidad para tomar decisiones basadas en datos confiables. Esta propuesta define la arquitectura, tecnologías, requerimientos y plan de entregas para construir un **Sistema Integrado de Gestión de Clientes (CRM)** que centralice la información, automatice procesos y habilite una cultura *data-driven*.

El sistema se construirá con **Vue 3** (Composition API) para la capa de vista, **Node.js + Express** para la API, **MySQL** como motor de datos, y **Vitest + Jest** para pruebas unitarias, distribuyendo el trabajo entre un equipo de 4 desarrolladores (1 senior, 3 juniors) en 3 iteraciones.

---

## 2. Definición del Problema y Causas Raíz

### 2.1 Problema principal

La arquitectura de información de Atlantic City está fragmentada: múltiples sistemas independientes (membresías, promociones, registro de clientes, atención) no se comunican entre sí. Esto no es una percepción aislada sino una **deficiencia estructural verificable** (ver [Análisis del Problema Estructural](analisis_problema_estructural.md)).

### 2.2 Causas raíz identificadas

| # | Causa raíz | Impacto |
|---|-----------|---------|
| 1 | Ausencia de estrategia de integración entre sistemas | Los sistemas evolucionaron de forma independiente sin una visión unificada |
| 2 | Falta de un repositorio central de datos | Un mismo cliente existe con información inconsistente en múltiples sistemas |
| 3 | Dependencia de procesos manuales | Los empleados consultan múltiples sistemas para completar una sola tarea |
| 4 | Crecimiento tecnológico no planificado | Se incorporaron soluciones puntuales sin considerar la arquitectura global |
| 5 | Escasa capacidad analítica transversal | No existe visión 360° del cliente; las decisiones se toman sin datos completos |

### 2.3 Evidencia del problema

- **Duplicidad de datos**: Un mismo cliente puede existir en diferentes sistemas con información inconsistente.
- **Procesos manuales**: Los empleados invierten horas en buscar informaciónentre sistemas, corregir inconsistencias y consolidar reportes manualmente.
- **Atención inconsistente**: Los clientes experimentan respuestas diferentes según el canal o gerencia que los atiende.
- **Reportes poco fiables**: La información recopilada manualmente desde diversas fuentes carece de actualización en tiempo real.

---

## 3. Requerimientos

### 3.1 Requerimientos Funcionales

| ID | Requerimiento | Descripción | Prioridad | Gerencia involucrada |
|----|--------------|-------------|-----------|---------------------|
| RF-01 | Gestión de perfiles de clientes | Registro centralizado con datos de contacto, historial de visitas, preferencias e interacciones previas. Fuente única de verdad. | Alta | Operaciones, Servicio al Cliente |
| RF-02 | Segmentación de clientes | Clasificación por frecuencia de visitas, nivel de gasto, preferencias de juegos y promociones anteriores. Permite campañas dirigidas. | Alta | Marketing |
| RF-03 | Gestión de promociones | CRUD de promociones con asignación por segmento, fechas de vigencia y seguimiento de efectividad. | Alta | Marketing, Desarrollo de Negocios |
| RF-04 | Módulo de atención al cliente | Registro de quejas, problemas y solicitudes con historial completo de incidencias y soluciones. Estados y seguimiento. | Alta | Servicio al Cliente |
| RF-05 | Reportes y dashboards | Visualizaciones en tiempo real: clientes más activos, promociones más exitosas, tiempos de atención. Filtros por variable clave. | Media | Gerencias (todas) |
| RF-06 | Autenticación y roles | Sistema de login con roles por gerencia (Operaciones, Marketing, RRHH, Negocios, Servicio al Cliente, TI) y permisos diferenciados. | Alta | TI |
| RF-07 | Automatización de procesos | Automatización de tareas repetitivas: notificaciones de promociones por segmento, escalamiento de quejas, actualización de estados. | Media | Operaciones, Servicio al Cliente |
| RF-08 | Gestión de membresías | Registro y seguimiento del estado de membresía de cada cliente (activa, vencida,Upgrade). | Media | Operaciones, Marketing |

### 3.2 Requerimientos No Funcionales

| ID | Requerimiento | Descripción | Justificación |
|----|--------------|-------------|---------------|
| RNF-01 | Rendimiento | Las consultas principales deben responder en menos de 2 segundos con 1,200 usuarios concurrentes. | Atlantic City tiene más de 1,200 empleados que podrían acceder simultáneamente. |
| RNF-02 | Seguridad | Protección de datos personales y financieros siguiendo OWASP Top 10. Autenticación JWT, encriptación de contraseñas con bcrypt, control de acceso basado en roles (RBAC). | El análisis de riesgo identifica brechas de seguridad como riesgo Alto (IDs 2 y 8). Los datos del casino incluyen información financiera sensible. |
| RNF-03 | Usabilidad | Interfaz intuitiva, responsiva, con flujos de trabajo claros por gerencia. Time-to-first-action < 5 minutos sin capacitación formal. | La resistencia al cambio es un riesgo identificado (ID 3). Una interfaz difícil amplificaría este rechazo. |
| RNF-04 | Disponibilidad | El sistema debe estar disponible 99.5% del tiempo (máximo 3.6 horas de caída al mes). | El casino opera de forma continua; caídas afectan la atención directa al cliente. |
| RNF-05 | Escalabilidad | Arquitectura modular que permita agregar módulos sin rediseño. Base de datos preparada para crecimiento del volumen de clientes. | El análisis FODA señala la oportunidad de crecimiento; el sistema no debe ser un cuello de botella. |
| RNF-06 | Mantenibilidad | Código documentado, pruebas unitarias con cobertura ≥ 70%, estructura de carpetas por dominio. | Equipo con 3 juniors; la mantenibilidad reduce dependencia del senior y mejora la curva de aprendizaje. |
| RNF-07 | Compatibilidad | Funcionamiento en navegadores modernos (Chrome, Firefox, Edge, Safari — últimas 2 versiones). | La evaluación de infraestructura (riesgo ID 8) requiere verificar que las PCs del casino tengan navegadores compatibles. |

### 3.3 Limitaciones técnicas identificadas

| Limitación | Impacto | Mitigación propuesta |
|-----------|---------|---------------------|
| Infraestructura local del casino puede ser obsoleta | El sistema web no funcionará correctamente si los navegadores o la red son antiguos | Evaluación previa de las estaciones de trabajo; diseñar para navegadores modernos con fallback progresivo |
| Sistemas existentes pueden no tener APIs de integración | No será posible importar datos históricos automáticamente en la primera versión | Migración manual asistida con scripts ETL en fases posteriores; primera versión como sistema limpio |
| Equipo con 3 juniors y plazo limitado | Curva de aprendizaje puede generar retrasos | Asignar módulos bien delimitados con interfaces claras; el senior actúa como arquitecto y revisor |
| Datos actuales fragmentados y duplicados | La calidad de los datos migrados será baja sin limpieza previa | Incluir módulo de deduplicación y validación como requisito del sprint de cliente |

---

## 4. Arquitectura del Sistema

### 4.1 Enfoque arquitectónico

Se adopta una **arquitectura en capas** con separación de responsabilidades, priorizando la **capa de vista** (foco del proyecto) pero garantizando que cada capa sea independently testeable.

```
┌─────────────────────────────────────────────────────────┐
│                   CAPA DE VISTA (Vue 3)                  │
│  ┌──────────┐ ┌──────────┐ ┌───────────┐ ┌───────────┐  │
│  │ Clientes  │ │ Promoc.  │ │ Atención  │ │ Reportes  │  │
│  │ Module    │ │ Module   │ │ Module    │ │ Module    │  │
│  └─────┬─────┘ └────┬─────┘ └─────┬─────┘ └─────┬─────┘  │
│        │             │             │             │        │
│  ┌─────┴─────────────┴─────────────┴─────────────┴─────┐ │
│  │              Router + State (Pinia)                  │ │
│  └──────────────────────┬──────────────────────────────┘ │
│                         │ HTTP/JSON                       │
├─────────────────────────┼────────────────────────────────┤
│                CAPA DE NEGOCIO (Express)                  │
│  ┌──────────┐ ┌──────────┐ ┌───────────┐ ┌───────────┐  │
│  │ Client  │ │ Promo    │ │ Ticket    │ │ Report   │  │
│  │ Service │ │ Service  │ │ Service   │ │ Service   │  │
│  └─────┬───┘ └────┬─────┘ └─────┬─────┘ └─────┬─────┘  │
│        │           │             │             │         │
│  ┌─────┴───────────┴─────────────┴─────────────┴───────┐ │
│  │           Auth Middleware (JWT + RBAC)               │ │
│  └──────────────────────┬──────────────────────────────┘ │
│                         │ ORM (Prisma)                    │
├─────────────────────────┼────────────────────────────────┤
│                  CAPA DE DATOS (MySQL)                    │
│  ┌──────────┐ ┌──────────┐ ┌───────────┐ ┌───────────┐ │
│  │ clients  │ │ promos   │ │ tickets   │ │ users     │ │
│  │ segments │ │ assign.  │ │ comments  │ │ roles     │ │
│  └──────────┘ └──────────┘ └───────────┘ └───────────┘ │
└─────────────────────────────────────────────────────────┘
```

### 4.2 Componentes por capa

#### Capa de Vista (Vue 3 + Composition API)

| Componente | Responsabilidad | Detalle |
|-----------|-----------------|---------|
| **Layout principal** | Shell de la aplicación | Sidebar con navegación por gerencia, header con datos del usuario, área de contenido dinámica |
| **Módulo Clientes** | Gestión de perfiles y segmentación | Lista paginada, detalle con tabs (datos, visitas, promociones, tickets), formulario de edición, filtros de segmentación |
| **Módulo Promociones** | CRUD de promociones y asignación | Crear/editar promoción, asignar a segmentos, calendario de vigencia, métricas de efectividad |
| **Módulo Atención al Cliente** | Quejas y solicitudes | Lista de tickets por estado, formulario de nuevo ticket, historial de interacciones, escalamiento |
| **Módulo Reportes** | Dashboards y exportación | Gráficos de actividad, efectividad de promociones, KPIs (# clientes activos, tiempo respuesta, conversión) |
| **Módulo Auth** | Login y gestión de sesión | Login, logout, cambio de contraseña, visualización de permisos según rol |
| **Router + Pinia** | Navegación y estado global | Rutas protegidas por rol, store centralizado para estado compartido |
| **Servicios API** | Comunicación con backend | Capa de servicios HTTP (axios) con interceptores para auth y errores |

#### Capa de Negocio (Node.js + Express)

| Componente | Responsabilidad |
|-----------|-----------------|
| **Auth Service** | Login, emisión de JWT, validación de tokens, gestión de roles |
| **Client Service** | CRUD de clientes, búsqueda, segmentación, deduplicación |
| **Promotion Service** | CRUD de promociones, asignación a segmentos, métricas de efectividad |
| **Ticket Service** | CRUD de tickets, comentarios, escalamiento, cambio de estado |
| **Report Service** | Generación de reportes agregados, estadísticas, datos para dashboards |
| **RBAC Middleware** | Control de acceso basado en roles por gerencia |
| **Error Handler** | Manejo centralizado de errores con logging |
| **Validation Layer** | Validación de entrada con esquemas (Joi/Zod) |

#### Capa de Datos (MySQL)

| Entidad principal | Campos clave | Relaciones |
|------------------|-------------|------------|
| `users` | id, email, password_hash, role_id, gerency | Pertenece a role |
| `roles` | id, name, permissions (JSON) | Tiene muchos users |
| `clients` | id, name, email, phone, dni, segment_id | Pertenece a segment |
| `segments` | id, name, criteria (JSON) | Tiene muchos clients |
| `promotions` | id, title, description, start_date, end_date, type | Pertenece a muchos segments (M:N) |
| `client_promotions` | client_id, promotion_id, status, redeemed_at | M:N entre clients y promotions |
| `tickets` | id, client_id, subject, status, priority, assigned_to | Pertenece a client, assigned_to user |
| `ticket_comments` | id, ticket_id, user_id, comment, created_at | Pertenece a ticket y user |
| `visits` | id, client_id, date, amount_spent, preferences (JSON) | Pertenece a client |

### 4.3 Flujo de comunicación

```
Usuario → [Vue Component] → [Pinia Store] → [API Service (axios)]
                                                      ↓
                                              [Express Route]
                                                      ↓
                                              [Auth Middleware (JWT)]
                                                      ↓
                                              [Validation Layer]
                                                      ↓
                                              [Service Layer]
                                                      ↓
                                              [Prisma ORM]
                                                      ↓
                                              [MySQL]
```

### 4.4 Decisiones de arquitectura

| Decisión | Justificación |
|----------|---------------|
| **Aplicación monolítica** (no microservicios) | El equipo de 4 personas y el plazo limitado no justifican la complejidad operativa de microservicios. Un monolito bien modularizado permite separación lógica sin overhead de red. |
| **API REST** (no GraphQL) | La simplicidad de REST se alinea con el nivel del equipo y los patrones de acceso son predecibles (CRUD por entidad). GraphQL agregaría complejidad innecesaria. |
| **Prisma como ORM** | Genera tipos TypeScript automáticamente, maneja migraciones y reduce errores SQL manuales. Para juniors, es más seguro que escribir queries a mano. |
| **JWT para autenticación** | Stateless, escalable, compatible con la arquitectura REST. Los tokens llevan el rol del usuario para que el middleware RBAC valide permisos sin consultas adicionales a la base de datos. |
| **Pinia para estado** | Pinia es el store oficial de Vue 3 (reemplaza Vuex). Es más simple, tiene mejor soporte para TypeScript, y su API es más directa para juniors. |
| **Separación por dominio** | La estructura de carpetas sigue el dominio de negocio (clientes, promociones, tickets), no el tipo técnico (controllers, models). Esto facilita que cada desarrollador trabaje en un módulo de extremo a extremo. |

---

## 5. Selección de Tecnologías

### 5.1 Framework de vista: Vue 3 (Composition API)

| Criterio | Vue 3 | React 18 | Decisión |
|----------|-------|----------|----------|
| **Curva de aprendizaje** | Menor: template declarativo, reactividad integrada, convenciones claras | Mayor: JSX, hooks, decisiones de estado (useState vs useReducer) | **Vue** — Con 3 juniors, la curva de aprendizaje es crítica para la productividad |
| **Estructura por convención** | Single-File Components (SFC) con template + script + style en un archivo | Libre: exige decisiones arquitectónicas (CSS modules, styled-components, etc.) | **Vue** — Las convenciones reducen la fricción de decisión |
| **Ecosistema oficial** | Router oficial, Pinia (store oficial), Vue Test Utils | React Router (terceros), múltiples stores (Redux, Zustand, Jotai) | **Vue** — Menos fragmentación significa menos tiempo en decisiones |
| **Tooling** | Vite (creado por el equipo de Vue), Vue DevTools | CRA esprecated, Vite también disponible | Empate — Ambos funcionan bien con Vite |
| **Mercado laboral** | Mayor demanda en el mercado global | Demanda significativamente mayor | React gana en mercado, pero Vue tiene presencia sólida en LATAM |
| **Testing** | Vitest + Vue Test Utils + @vue/compiler-sfc | Jest + React Testing Library | Empate — Ambos tienen ecosistemas de testing maduros |
| **Mantenibilidad a largo plazo** | Bueno con Composition API y TypeScript | Bueno con hooks y TypeScript | Empate — Ambos escalan bien con buenas prácticas |

**Decisión: Vue 3 con Composition API.** La justificación principal es el perfil del equipo (3 juniors). Vue ofrece una curva de aprendizaje más corta, convenciones más claras que reducen la fricción de decisión, y un ecosistema menos fragmentado donde "la forma oficial" es la forma correcta. Esto acelera la productividad del equipo y reduce errores arquitectónicos por inexperiencia.

### 5.2 Stack completo

| Capa | Tecnología | Versión | Justificación |
|------|-----------|---------|---------------|
| **Vista** | Vue 3 + Composition API | 3.4+ | Ver análisis arriba |
| **Estado** | Pinia | 2.x | Store oficial para Vue 3; API simple, TypeScript-first |
| **Routing** | Vue Router | 4.x | Router oficial; guards nativos para RBAC |
| **HTTP Client** | Axios | 1.x | Interceptors para auth y errores; ampliamente documentado |
| **Estilos** | Tailwind CSS | 3.x | Utility-first acelera el desarrollo de UI sin escribir CSS personalizado; consistencia visual por defecto |
| **Componentes** | Headless UI (o similares) | — | Componentes accesibles sin estilos predefinidos; se integra con Tailwind |
| **Backend** | Node.js + Express | Node 20 LTS, Express 4.x | Stack conocido por el equipo; ecosistema npm más amplio |
| **Validación** | Zod | 3.x | Inferencia de tipos TypeScript desde esquemas; validación compartida entre frontend y backend |
| **ORM** | Prisma | 5.x | Tipos generados automáticamente, migraciones integradas, previene SQL manual error-prone |
| **Autenticación** | JWT + bcrypt | jsonwebtoken, bcrypt | Estándar para APIs REST; bcrypt para hash de contraseñas |
| **Base de datos** | MySQL | 8.x | Requerimiento del proyecto; soporta transacciones ACID, relaciones complejas |
| **Testing Frontend** | Vitest + Vue Test Utils | Vitest 1.x | Nativo para Vite; compatible con la API de Jest |
| **Testing Backend** | Jest + Supertest | Jest 29.x | Estándar para testing en Node.js; Supertest para pruebas de integración de API |
| **Linting** | ESLint + Prettier | — | Formato consistente; reduce fricción en code review |

### 5.3 Alternativas evaluadas y descartadas

| Alternativa | Descartada por |
|-------------|---------------|
| React 18 | Mayor curva de aprendizaje; ecosistema más fragmentado ( más decisiones para juniors) |
| GraphQL | Complejidad innecesaria para patrones CRUD predecibles; overhead de aprendizaje |
| MongoDB | Los datos del CRM son inherentemente relacionales (clientes → visitas → tickets → promociones); una base relacional es más natural |
| Sequelize | API más verbosa que Prisma; sin generación automática de tipos |
| Vuex | Deprecado para Vue 3; Pinia es el reemplazo oficial |
| Bootstrap/Material | Estilos predefinidos chocan con la identidad visual de Atlantic City; Tailwind ofrece más control |

---

## 6. Plan de Iteraciones

La estrategia de entrega es **iterativa incremental**: cada iteración produce software funcional que puede ser demostrado.

### Iteración 1 — Fundación y Módulo de Clientes (Semanas 1-3)

**Objetivo**: Infraestructura base + autenticación + gestión de perfiles de clientes

| Semana | Entregable | Responsable |
|--------|-----------|-------------|
| 1 | Proyecto inicializado (Vite + Vue 3, Express, Prisma, MySQL). Esquema de base de datos. Pipeline de CI con linting y tests. | Senior (Tech Lead) |
| 1 | App Shell: Layout principal, sidebar por gerencia, router con guards de auth | Junior 1 (con supervisión del Senior) |
| 2 | Auth: Login, logout, JWT, middleware RBAC, roles por gerencia | Senior |
| 2 | Client CRUD: Crear, listar, editar, buscar clientes | Junior 1 |
| 3 | Client Detail: Vista de detalle con tabs (datos, visitas, promociones, tickets) | Junior 1 |
| 3 | Deduplicación: Validación de DNI/email duplicados al registrar cliente | Junior 1 |
| 3 | Segmentación: Filtros por frecuencia de visitas, gasto, preferencias | Junior 2 |

**Criterio de aceptación de la iteración**: Un usuario autenticado puede crear, buscar, editar y segmentar clientes. Los datos se persisten en MySQL. Tests unitarios cobren los servicios de Auth y Client.

### Iteración 2 — Promociones y Atención al Cliente (Semanas 4-6)

**Objetivo**: Módulos de promociones y tickets de atención

| Semana | Entregable | Responsable |
|--------|-----------|-------------|
| 4 | Promotion CRUD: Crear, editar, desactivar promociones con vigencia | Junior 2 |
| 4 | Asignación de promociones a segmentos | Junior 2 |
| 5 | Ticket CRUD: Crear quejas/solicitudes, comentarios, cambio de estado | Junior 3 |
| 5 | Escalamiento de tickets: reglas de negocio (monto, prioridad) | Junior 3 |
| 6 | Client Detail integrado: tabs de promociones y tickets visibles desde el perfil del cliente | Junior 1 + Junior 2 + Junior 3 |
| 6 | Validación cruzada: al asignar promoción, verificar que el segmento existe y está activo | Junior 2 |

**Criterio de aceptación de la iteración**: Un usuario de Marketing puede crear y asignar promociones a segmentos. Un usuario de Servicio al Cliente puede crear y dar seguimiento a tickets. Los tickets y promociones se ven desde el perfil del cliente.

### Iteración 3 — Reportes, Dashboards y Refinamiento (Semanas 7-9)

**Objetivo**: Visualización de datos, reports automáticos, corrección de hallazgos

| Semana | Entregable | Responsable |
|--------|-----------|-------------|
| 7 | Dashboard de KPIs: clientes activos, promociones más efectivas, tiempo promedio de respuesta | Junior 3 |
| 7 | Gráficos de actividad: uso por gerencia, tendencias de visitas | Junior 3 |
| 8 | Reportes exportables (CSV/PDF) | Junior 2 |
| 8 | Refinamiento de UX: feedback de pruebas con usuario, ajustes de flujo | Todos (integración) |
| 9 | Pruebas de integración end-to-end | Senior |
| 9 | Documentación técnica y manual de usuario | Todos |
| 9 | Despliegue a entorno de producción | Senior |

**Criterio de aceptación de la iteración**: Los dashboards muestran datos en tiempo real. Los reportes se exportan correctamente. El sistema está desplegado y accesible desde las estaciones de trabajo del casino.

---

## 7. Estrategia de Pruebas

| Nivel | Herramienta | Cobertura objetivo | Responsabilidad |
|-------|------------|-------------------|-----------------|
| **Unitarias - Frontend** | Vitest + Vue Test Utils | ≥ 70% de componentes y stores | Cada desarrollador en su módulo |
| **Unitarias - Backend** | Jest | ≥ 70% de servicios y middlewares | Cada desarrollador en su módulo |
| **Integración de API** | Jest + Supertest | Endpoints principales (auth, CRUD) | Senior (revisión) + responsable del módulo |
| **E2E** | Cypress o Playwright | Flujos críticos (login, crear cliente, crear ticket) | Senior |

### Flujo de trabajo con tests

1. Desarrollador escribe tests unitarios **antes** o **junto con** el código (TDD pragmático).
2. Pull Requests sin tests que cubran la lógica nueva no se aprueban.
3. CI ejecuta todos los tests en cada PR; si falla, no se mergea.
4. El Senior revisa tanto la lógica como la cobertura de tests en cada PR.

---

## 8. Gestión de Riesgos del Proyecto

Referencia cruzada con el [Análisis de Riesgo](analisis_de_riesgo.md) existente. Aquí se detallan los riesgos **técnicos del desarrollo**:

| ID | Riesgo | Probabilidad | Impacto | Mitigación |
|----|--------|-------------|---------|-----------|
| RP-01 | Curva de aprendizaje de juniors en Vue 3/Prisma | Alta | Moderado | Pair programming las primeras 2 semanas; el Senior lidera la configuración inicial y crea ejemplos de referencia |
| RP-02 | Scope creep por requerimientos no contemplados | Moderada | Alto | Definir alcance cerrado por iteración; cualquier cambio pasa por aprobación del equipo y se evalúa impacto |
| RP-03 | Dependencia del Senior como cuello de botella | Moderada | Alto | Documentar decisiones arquitectónicas; Code Review express (máximo 24h); los juniors proponen soluciones antes de esperar instrucciones |
| RP-04 | DatosSucios heredados afecten la calidad del sistema | Alta | Moderado | Implementar deduplicación en la creación de clientes; validación estricta en formularios |
| RP-05 | Infraestructura del casino incompatible | Baja | Alto | Evaluación previa de navegadores y red; diseño responsive y progresive enhancement |

---

## 9. Entregables por Iteración

| Iteración | Entregable | Formato |
|-----------|-----------|---------|
| I1 | Software funcional: Auth + Clientes + Segmentación | Repositorio Git + Demo desplegada |
| I1 | Esquema de base de datos (migraciones Prisma) | Archivo `schema.prisma` |
| I1 | Tests unitarios de Auth y Client | Ejecutables via `npm test` |
| I2 | Software funcional: Promociones + Tickets de atención | Repositorio Git + Demo desplegada |
| I2 | Tests unitarios de Promotions y Tickets | Ejecutables via `npm test` |
| I3 | Software funcional: Dashboards + Reportes exportables | Repositorio Git + Demo desplegada |
| I3 | Tests de integración E2E | Ejecutables via Cypress/Playwright |
| I3 | Documentación técnica | Archivos Markdown en el repo |
| I3 | Manual de usuario | Archivo Markdown en el repo |

---

## 10. Referencias

- [Problema de Atlantic City](problema_atlantic_city.md)
- [Análisis del Problema Estructural](analisis_problema_estructural.md)
- [FODA de Atlantic City](foda_atlantic_city.md)
- [Análisis de Riesgo](analisis_de_riesgo.md)
- [Rúbrica de Evaluación](rubrica_evaluacion_parcial_proyecto_capa_vista.md)