# Guía de Trabajo — Junior 3: Módulo de Atención al Cliente y Dashboards

[<- Volver al README](../README.md)

## Para quién es esta guía

Esta guía es para **Junior 3**, responsable del **Módulo de Atención al Cliente (tickets)** y los **Dashboards de KPIs**: sistema de quejas y solicitudes con escalamiento, y visualización de métricas del CRM.

Trabajas de extremo a extremo: Vue 3 en el frontend, Express en el backend y Prisma en la base de datos. Tu módulo tiene dos retos técnicos: **reglas de negocio de escalamiento** y **agregaciones y gráficos** para dashboards.

> **Lectura obligatoria antes de empezar**: [Propuesta de Desarrollo](propuesta_desarrollo.md) (secciones 3.1 RF-04 y RF-05, 4.2 y 6) y [Distribución del Equipo](distribucion_equipo.md) (secciones 2 y 4).

---

## 1. ¿Qué vas a construir?

| Iteración | Qué entregas | Semanas |
|-----------|--------------|---------|
| I2 | Ticket CRUD: crear quejas/solicitudes con comentarios | 5 |
| I2 | Escalamiento de tickets por reglas de negocio (monto, prioridad) | 5 |
| I2 | Línea de tiempo de interacciones por ticket | 5 |
| I2 | Tests unitarios de Tickets (≥ 70%) | Continuo |
| I3 | Dashboard de KPIs: clientes activos, conversiones, tiempos de respuesta | 7 |
| I3 | Gráficos de actividad por gerencia y tendencias de visitas | 7 |
| I3 | Report Service compartido con Junior 2 (agregaciones) | 7 |
| I3 | Refinamiento de UX, documentación y manual de usuario de tu módulo | 8-9 |

**Resultado final**: un usuario de Servicio al Cliente puede registrar quejas, comentarlas, escalarlas según reglas y ver su historial. Y un usuario de cualquier gerencia puede ver un dashboard con KPIs del CRM en tiempo real.

---

## 2. Temas que tienes que estudiar o repasar

### 2.1 Fundamentos de Vue 3 (Antes de la Semana 5)

| Tema | Para qué lo necesitas | Recursos |
|------|----------------------|----------|
| **Composition API**: `setup`, `ref`, `reactive`, `computed`, `watch` | Todos tus componentes | [Vue 3 Guide](https://vuejs.org/guide/extras/composition-api-faq.html) |
| **SFC** | Estructura base | [Vue SFC](https://vuejs.org/guide/scaling-up/sfc.html) |
| **Props y emits** | Línea de tiempo recibe `ticketId`, formulario emite `submit` | [Props](https://vuejs.org/guide/components/props.html), [Events](https://vuejs.org/guide/components/events.html) |
| **v-if / v-for con `key`** | Renderizar lista de comentarios, estados de tickets | [List Rendering](https://vuejs.org/guide/essentials/list.html) |
| **v-model en componentes** | Formulario de nuevo ticket | [Form Input Bindings](https://vuejs.org/guide/elements/forms.html) |
| **Lifecycle**: `onMounted`, `onUnmounted` | Cargar ticket y comentarios al montar el detalle | [Lifecycle](https://vuejs.org/guide/essentials/lifecycle.html) |

**Ejercicio práctico antes de tu primera tarea**: crea un componente `TicketStatusBadge.vue` que reciba `status` por prop ('open', 'in_progress', 'escalated', 'closed') y muestre un badge de color distinto según el estado. Esto valida props + computed.

### 2.2 Pinia (Semana 5)

| Tema | Para qué lo necesitas |
|------|----------------------|
| **Definir store** con setup syntax | Store de tickets (listado, ticket seleccionado, comentarios) |
| **State, getters, actions** | Cargar tickets, filtrar por estado, agregar comentario |
| **`storeToRefs`** | Destructurar manteniendo reactividad |
| **Acciones asíncronas** | Llamar al servicio y manejar loading/error |
| **Store de dashboard separado** | Store de KPIs diferente al de tickets |

### 2.3 Listas y líneas de tiempo en Vue (Semana 5)

| Tema | Para qué lo necesitas |
|------|----------------------|
| **Renderizado de listas con `v-for` y `:key`** | Lista de tickets, comentarios en línea de tiempo |
| **Componentes de lista** (item como componente hijo) | `TicketListItem.vue`, `CommentItem.vue` |
| **Scroll virtual** (opcional) | Si la lista de tickets crece, considera `vue-virtual-scroller` |
| **Estados vacíos y de carga** | "Sin tickets", "Cargando..." — no dejes vistas en blanco |

> **Gotcha**: nunca uses el `index` del `v-for` como `:key` si la lista puede reordenarse o eliminar items. Usa `ticket.id` como key.

### 2.4 HTTP con Axios (Semana 5)

| Tema | Para qué lo necesitas |
|------|----------------------|
| **Instancia axios con interceptors** | Auth JWT, manejo de errores |
| **Métodos CRUD** | Endpoints de Ticket y Comments |
| **Paginación** | Lista de tickets puede ser larga |
| **Polling opcional**: `setInterval` para refrescar | Ver nuevos tickets sin recargar (mejora opcional) |

### 2.5 Backend: Express + Prisma (Semana 5)

| Tema | Para qué lo necesitas |
|------|----------------------|
| **Express routers y controllers** | `ticket.routes.js`, `ticket.controller.js` |
| **Prisma relaciones**: `Ticket → Client`, `Ticket → User (assigned_to)`, `Ticket → TicketComment` | Traer ticket con su cliente y comentarios |
| **Prisma**: `create` con relaciones anidadas | Crear ticket con comentario inicial |
| **Prisma transacciones**: `prisma.$transaction` | Escalamiento: cambiar `assigned_to` y `status` atómicamente |
| **Zod para validación** | Esquema de ticket: `subject` no vacío, `priority` enum (low, medium, high) |

> **Patrón a replicar**: estudia `client.service.js` de Junior 1. Replica el mismo patrón (controller → service → prisma).

### 2.6 Reglas de negocio de escalamiento (Semana 5)

Este es el tema más interesante de tu módulo. Las reglas de escalamiento están derivadas del [Análisis del Problema Estructural](analisis_problema_estructural.md) y de RF-07.

| Regla | Trigger | Acción |
|-------|---------|--------|
| E1 | `priority === 'high'` al crear el ticket | Asignar automáticamente al usuario de TI con rol de supervisor |
| E2 | `amount_at_stake >= 5000` (monto involucrado) | Escalar a gerencia de Operaciones |
| E3 | Ticket sin respuesta en 48h | Subir `priority` un nivel y marcar `escalated: true` |

**Cómo implementarlas**:
1. Crea un método `applyEscalationRules(ticket)` en `ticket.service.js` que reciba el ticket y devuelva la asignación sugerida.
2. Llámalo dentro de la transacción de creación (`prisma.$transaction`).
3. Para la regla E3 (tiempo), necesitarás un job programado. En la primera versión, impleméntala como endpoint manual `/api/tickets/check-escalations` que el Senior convertirá en cron job.

**Criterios de aceptación**:
- Crear un ticket con `priority: 'high'` lo asigna automáticamente al supervisor de TI.
- Crear un ticket con `amount_at_stake >= 5000` lo escala a Operaciones.
- El endpoint `/check-escalations` sube prioridad de tickets > 48h sin respuesta.

**Temas a estudiar antes**: 2.5 (transacciones Prisma), reglas de negocio (este apartado).

### 2.7 Dashboards y visualización (Semana 7)

| Tema | Para qué lo necesitas |
|------|----------------------|
| **Librería de gráficos**: Chart.js (recomendada) o ApexCharts | Gráficos de actividad y tendencias |
| **Integración de Chart.js con Vue 3**: `vue-chartjs` wrapper o uso directo con `ref` al canvas | Componentes de gráfico reutilizables |
| **Tipos de gráfico**: barras, líneas, doughnut, KPI card | Métricas distintas requieren grafos distintos |
| **Actualización en tiempo real**: refresco periódico o WebSocket (avanzado) | KPIs "en tiempo real" del RF-05 |
| **Layout de dashboard**: grid responsive con Tailwind | Distribución de tarjetas y gráficos |

> **Recomendación**: empieza con un refresco vía `setInterval` cada 30 segundos. Es más simple que WebSocket y suficiente para la primera versión.

### 2.8 Agregaciones con Prisma (Semana 7)

| Tema | Para qué lo necesitas |
|------|----------------------|
| **`prisma.client.count`** | KPI: clientes activos |
| **`prisma.promotion.count`** con `where` | KPI: promociones activas |
| **`prisma.ticket.aggregate`**: `_avg`, `_count`, `_max`, `_min` | KPI: tiempo promedio de respuesta |
| **`groupBy` en Prisma** | Tickets por estado, promociones por tipo |
| **Queries con `include` y `select`** | Traer datos agregados para gráficos |

Ejemplo para "tiempo promedio de respuesta":
```js
const avgResponseTime = await prisma.ticket.aggregate({
  _avg: { response_time_minutes: true },
  where: { status: 'closed' }
});
```

### 2.9 Report Service compartido (Semana 7)

La carpeta `server/src/modules/reports/` la compartes con Junior 2 (ver [Distribución del Equipo](distribucion_equipo.md) sección 3). En I3:
- **Tú**: endpoints de agregaciones para dashboards (`/api/reports/kpis`, `/api/reports/activity`).
- **Junior 2**: endpoints de exportación CSV/PDF (`/api/reports/clients.csv`, etc.).
- **Coordinación**: acuerden prefijos de rutas para no pisarse (`/kpis` vs `/export/`).

### 2.10 Testing (Continuo)

| Tema | Para qué lo necesitas |
|------|----------------------|
| **Vitest** para componentes Vue | Tests de `TicketForm`, `CommentTimeline`, `Dashboard` |
| **Vue Test Utils**: `mount`, `setProps`, `trigger` | Interacción con formularios |
| **Jest** para backend | Tests de `ticket.service.js` |
| **Mock de Prisma** | Testear service sin BD real |
| **Mock de Chart.js** | En tests de componentes de dashboard, mockea el chart |
| **Cobertura ≥ 70%** | Requisito |

---

## 3. Tareas detalladas por iteración

### Iteración 2 — Lo que construyes tú

#### Tarea 2.1 — Ticket CRUD (Semana 5)

**Objetivo**: Crear, listar, ver detalle de tickets de quejas y solicitudes.

**Pasos sugeridos**:

Backend (en `server/src/modules/tickets/`):
1. Crea `ticket.routes.js` con `GET /`, `GET /:id`, `POST /`, `PATCH /:id/status`, `POST /:id/comments`.
2. Crea `ticket.controller.js` con validación Zod.
3. Crea `ticket.service.js` con CRUD usando Prisma. Estados: `open`, `in_progress`, `escalated`, `closed`.
4. Al crear un ticket, permite incluir un comentario inicial en la misma transacción (`prisma.ticket.create({ data: { ...ticket, comments: { create: { ...initialComment } } } })`).

Frontend (en `client/src/modules/tickets/`):
1. Crea `services/TicketService.js`.
2. Crea `stores/tickets.js` con Pinia.
3. Crea `views/TicketListView.vue`: lista de tickets con filtro por estado.
4. Crea `components/TicketForm.vue`: formulario de nuevo ticket (subject, description, priority, client).
5. Crea `views/TicketDetailView.vue`: detalle con datos y línea de tiempo de comentarios.

**Criterios de aceptación**:
- Crear un ticket lo persiste con estado `open`.
- Listar tickets filtra por estado correctamente.
- Ver el detalle de un ticket muestra sus comentarios en orden cronológico.

**Temas a estudiar antes**: 2.1, 2.2, 2.3, 2.4, 2.5.

#### Tarea 2.2 — Escalamiento de tickets (Semana 5)

**Objetivo**: Aplicar las reglas E1, E2, E3 del análisis.

**Pasos sugeridos**:
1. Implementa `applyEscalationRules(ticket)` en `ticket.service.js`.
2. Llámalo dentro de `prisma.$transaction` en la creación del ticket.
3. Crea endpoint `POST /api/tickets/check-escalations` que suba prioridad de tickets > 48h sin respuesta.
4. En el frontend, muestra visualmente cuando un ticket está escalado (badge `escalated`).
5. Registra en `ticket_comments` un comentario automático del sistema cuando se escala ("Escalado a Operaciones por monto: USD 5000").

**Criterios de aceptación**:
- Ticket con `priority: 'high'` se asigna automáticamente al supervisor de TI.
- Ticket con `amount_at_stake >= 5000` se escala a Operaciones con comentario automático.
- El endpoint `/check-escalations` sube prioridad y marca `escalated: true` en tickets > 48h.

**Temas a estudiar antes**: 2.5 (transacciones), 2.6.

#### Tarea 2.3 — Comentarios en tickets (Semana 5)

**Objetivo**: Línea de tiempo de interacciones por ticket.

**Pasos sugeridos**:
1. Crea `components/CommentTimeline.vue`: lista cronológica de comentarios con autor, fecha y contenido.
2. Crea `components/CommentForm.vue`: input para agregar comentario.
3. Al agregar comentario, si el ticket estaba `open`, cambiar a `in_progress` automáticamente.
4. Muestra visualmente quién escribió cada comentario (avatar o iniciales del usuario).

**Criterios de aceptación**:
- Agregar un comentario a un ticket `open` lo mueve a `in_progress`.
- El comentario queda al final de la línea de tiempo en orden.
- Se muestra el autor de cada comentario.

**Temas a estudiar antes**: 2.1, 2.3, 2.5.

#### Tarea 2.4 — Tests unitarios de Tickets (Continuo, cierra fin de I2)

**Mínimos a cubrir**:
- `TicketService.create` con reglas de escalamiento (E1, E2).
- `TicketService.addComment` que cambia estado a `in_progress`.
- `TicketForm.vue`: validación de campos, envío correcto.
- `CommentTimeline.vue`: renderiza comentarios en orden.

### Iteración 3 — Lo que construyes tú

#### Tarea 3.1 — Dashboard de KPIs (Semana 7)

**Objetivo**: Dashboard con tarjetas de KPIs: clientes activos, promociones más efectivas, tiempo promedio de respuesta.

**Pasos sugeridos**:

Backend (en `server/src/modules/reports/`):
1. Crea `report.routes.js` con `GET /api/reports/kpis`.
2. Crea `report.service.js` con métodos `getKpis()` que use `prisma.client.count`, `prisma.ticket.aggregate`, etc.
3. Devuelve un objeto `{ activeClients, activePromotions, avgResponseTime, totalTickets }`.

Frontend:
1. Crea `views/DashboardView.vue` con grid responsive de Tailwind.
2. Crea `components/KpiCard.vue`: tarjeta reutilizable con `label`, `value`, `icon`, `trend`.
3. Carga los KPIs en `onMounted` y refresca con `setInterval` cada 30 segundos.

**Criterios de aceptación**:
- La vista muestra 4 KPI cards con datos reales del backend.
- Los KPIs refrescan cada 30 segundos sin recargar la página.
- Las métricas reflejan el estado actual de la BD.

**Temas a estudiar antes**: 2.7, 2.8.

#### Tarea 3.2 — Gráficos de actividad (Semana 7)

**Objetivo**: Gráficos de uso por gerencia, tendencias de visitas, efectividad de promociones.

**Pasos sugeridos**:

Backend:
1. Endpoint `GET /api/reports/activity`: tickets por gerencia (agrupar por `assigned_to.gerency`).
2. Endpoint `GET /api/reports/visits-trend`: visitas agrupadas por mes (`prisma.visits.groupBy`).
3. Endpoint `GET /api/reports/promotion-effectiveness`: ratio `redeemed / assigned` por promoción.

Frontend:
1. Crea `components/ActivityByGerencyChart.vue`: barras con Chart.js.
2. Crea `components/VisitsTrendChart.vue`: línea con Chart.js.
3. Crea `components/PromotionEffectivenessChart.vue`: doughnut o barras horizontales.
4. Integra los tres en `DashboardView.vue`.

**Criterios de aceptación**:
- El gráfico de barras muestra tickets por gerencia.
- El gráfico de líneas muestra la tendencia mensual de visitas.
- El gráfico de efectividad muestra el ratio de cada promoción.

**Temas a estudiar antes**: 2.7 (Chart.js con Vue), 2.8 (agregaciones).

#### Tarea 3.3 — Refinamiento de UX (Semana 8)

1. Prueba con un usuario real el flujo: crear ticket → comentar → escalar.
2. Ajusta al menos 3 fricciones detectadas.

#### Tarea 3.4 — Documentación y manual (Semana 9)

1. Documenta tus endpoints en `docs/`: `/api/tickets/*`, `/api/reports/kpis`, `/api/reports/activity`, etc.
2. Escribe la sección del manual de usuario de Atención al Cliente y Dashboards: capturas y pasos.

---

## 4. Definition of Done para tus tareas

1. ✅ Código en rama `feature/tickets-<descripción>` (o `feature/dashboard-`, `feature/reports-`), PR creado.
2. ✅ Tests unitarios pasan (cobertura ≥ 70% de tu módulo).
3. ✅ Senior aprobó el PR.
4. ✅ `npm run lint` sin errores.
5. ✅ Funcionalidad verificada en navegador.
6. ✅ Casos borde cubiertos (especialmente las reglas de escalamiento).

---

## 5. Reglas de comunicación

| Práctica | Frecuencia | Quién |
|----------|-----------|-------|
| Daily standup | Diaria 15 min | Todos |
| Coordinación con Junior 1 | En I2 (tab de tickets en Client Detail) | Junior 1 — acuerda el contrato del componente |
| Coordinación con Junior 2 | En I3 (carpeta `reports/` compartida) | Junior 2 — prefijos de rutas |
| Revisión de PR | ≤ 24h | Senior revisa tus PRs; tú revisas ≥ 1 PR/semana |

> **Regla de oro**: las reglas de escalamiento involucran información de otras gerencias (TI, Operaciones). Antes de codificar, valida con el Senior los IDs reales de los usuarios supervisores en la BD sembrada.

---

## 6. Recursos de referencia

- [Vue 3 Documentation](https://vuejs.org/)
- [Pinia Documentation](https://pinia.vuejs.org/)
- [Prisma Documentation — Aggregations](https://www.prisma.io/docs/concepts/components/prisma-client/aggregations-grouping)
- [Prisma Documentation — Transactions](https://www.prisma.io/docs/concepts/components/prisma-client/transactions)
- [Chart.js Documentation](https://www.chartjs.org/docs/latest/)
- [Vue Chart.js](https://vue-chartjs.org/)
- [Vitest](https://vitest.dev/)
- [Vue Test Utils](https://test-utils.vuejs.org/)

**Documentos del proyecto**:
- [Propuesta de Desarrollo](propuesta_desarrollo.md) — secciones 3.1 RF-04, RF-05, RF-07, 4.2, 6
- [Distribución del Equipo](distribucion_equipo.md) — secciones 2, 4, 5
- [Análisis del Problema Estructural](analisis_problema_estructural.md) — base para las reglas de escalamiento
- [Análisis de Riesgo](analisis_de_riesgo.md) — contexto de por qué la atención inconsistente es un dolor