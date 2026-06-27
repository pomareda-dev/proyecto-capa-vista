# Guía de Trabajo — Junior 2: Módulo de Promociones y Reportes

[<- Volver al README](../README.md)

## Para quién es esta guía

Esta guía es para **Junior 2**, responsable del **Módulo de Promociones y los Reportes Exportables**: CRUD de promociones, asignación de promociones a segmentos, gestión de vigencia y exportación de reportes en CSV/PDF.

Trabajas de extremo a extremo: Vue 3 en el frontend, Express en el backend y Prisma en la base de datos. Tu módulo depende fuertemente de **relaciones muchos-a-muchos** (promociones ↔ segmentos, promociones ↔ clientes), que es el tema técnico más nuevo para ti.

> **Lectura obligatoria antes de empezar**: [Propuesta de Desarrollo](propuesta_desarrollo.md) (secciones 3.1 RF-03, 4.2 y 6) y [Distribución del Equipo](distribucion_equipo.md) (secciones 2 y 4).

---

## 1. ¿Qué vas a construir?

| Iteración | Qué entregas | Semanas |
|-----------|--------------|---------|
| I1 | Segmentación: filtros por frecuencia, gasto, preferencias (en coordinación con Junior 1) | 3 |
| I2 | Promotion CRUD: crear, editar, desactivar promociones con vigencia | 4 |
| I2 | Asignación de promociones a segmentos con validación cruzada | 4 |
| I2 | Vigencia de promociones: fechas de inicio/fin y estados | 5 |
| I2 | Tests unitarios de Promotions (≥ 70%) | Continuo |
| I3 | Reportes exportables (CSV/PDF) | 8 |
| I3 | Refinamiento de UX, documentación y manual de usuario de tu módulo | 7-9 |

**Resultado final**: un usuario de Marketing puede crear promociones, asignarlas a segmentos de clientes, controlar su vigencia, medir efectividad y exportar reportes. Y un usuario de cualquier gerencia puede filtrar clientes por segmento.

---

## 2. Temas que tienes que estudiar o repasar

### 2.1 Fundamentos de Vue 3 (Antes de la Semana 3)

| Tema | Para qué lo necesitas | Recursos |
|------|----------------------|----------|
| **Composition API**: `setup`, `ref`, `reactive`, `computed`, `watch` | Todos tus componentes | [Vue 3 Guide](https://vuejs.org/guide/extras/composition-api-faq.html) |
| **SFC**: `<template>`, `<script setup>`, `<style scoped>` | Estructura base | [Vue SFC](https://vuejs.org/guide/scaling-up/sfc.html) |
| **Props y emits** | Componente de filtros → listado de clientes; formulario → calendario | [Props](https://vuejs.org/guide/components/props.html), [Events](https://vuejs.org/guide/components/events.html) |
| **v-model en componentes** | Formulario de promoción, inputs de fecha | [Form Input Bindings](https://vuejs.org/guide/elements/forms.html) |
| ** watchers**: `watch`, `watchEffect` | Reaccionar a cambios en filtros de segmentación | [Watchers](https://vuejs.org/guide/essentials/watchers.html) |

**Ejercicio práctico antes de tu primera tarea**: crea un componente `FilterPanel.vue` que emita un evento `filter-change` con `{ minVisits, minSpend }` cuando cambien los inputs. Verifica que el padre lo recibe. Esto valida que dominas props/emits.

### 2.2 Pinia (Semana 3-4)

| Tema | Para qué lo necesitas |
|------|----------------------|
| **Definir store** con setup syntax | Store de promociones (listado, promoción seleccionada) |
| **State, getters, actions** | Cargar promociones, filtrar por estado, asignar segmento |
| **`storeToRefs`** para destructurar manteniendo reactividad | Conectar store con la vista |
| **Acciones asíncronas** | Llamar al servicio y manejar loading/error |

> **Gotcha**: no llames a las acciones del store dentro de `computed`. Hazlo en `onMounted` o en un handler de evento.

### 2.3 Formularios y fechas en Vue (Semana 4)

| Tema | Para qué lo necesitas |
|------|----------------------|
| **Validación de formularios**: estados por campo, mensajes de error | Formulario de promoción (título, fechas, descripción) |
| **Datepickers**: integrar librería tipo `v-calendar` o HTML5 `<input type="date">` | Vigencia de promociones |
| **Manipulación de fechas**: `Date`, formatos ISO, librería `date-fns` | Comparar fechas de inicio y fin |
| **Desactivar fechas pasadas** en el datepicker | Validar que `start_date` no sea anterior a hoy |

> **Gotcha crítico**: las fechas se desalinean entre frontend y backend si envías la hora local. Siempre opera con fechas en formato ISO (`YYYY-MM-DD`) sin hora, o usa UTC consistentemente.

### 2.4 HTTP con Axios (Semana 4)

| Tema | Para qué lo necesitas |
|------|----------------------|
| **Instancia axios con interceptors** | Auth JWT en cada petición, manejo de errores |
| **Métodos CRUD**: `get`, `post`, `put`, `delete` | Endpoints de Promotion |
| **Query params** | Listar promociones por estado o segmento |
| **Manejo de errores HTTP** | 400 validación, 409 conflicto, 403 RBAC |

### 2.5 Backend: Express + Prisma (Semana 4-5)

| Tema | Para qué lo necesitas |
|------|----------------------|
| **Express routers y controllers** | `promotion.routes.js`, `promotion.controller.js` |
| **Prisma: relaciones M:N** | `promotions ↔ segments` (tabla de unión `SegmentPromotion`) |
| **Prisma: `include` y `select`** | Traer promoción con sus segmentos asignados |
| **Prisma: crear con relaciones**: `prisma.promotion.create({ data: { segments: { connect: [{ id: 1 }, { id: 2 }] } } })` | Asignación de promociones a segmentos |
| **Prisma: filtros con operadores relacionales** | Buscar promociones activas (`status: 'active'`), vigentes (`start_date <= now <= end_date`) |
| **Zod para validación** | Esquema de promoción: título no vacío, fechas válidas, `end_date > start_date` |

> **Patrón a replicar**: estudia el `client.service.js` de Junior 1 y replica el mismo patrón (controller → service → prisma). La consistencia entre módulos es un criterio de evaluación.

### 2.6 Validación cruzada (Semana 5-6)

Este es un tema específico de tu módulo. Validación cruzada significa: antes de asignar una promoción a un segmento, verificar que **el segmento existe y está activo**.

| Concepto | Detalle |
|----------|---------|
| Qué validar | El `segment_id` recibido existe en la tabla `segments` |
| Cuándo validar | En el service, antes de `prisma.segmentPromotion.create` |
| Cómo fallar | Lanza `SegmentNotFoundError` o `SegmentInactiveError` |
| HTTP status | 404 si no existe, 409 si está inactivo |

> **Coordinación con Junior 1**: él es el dueño del modelo `Segment`. Antes de codificar, acuerda qué campos tiene `Segment` (`id`, `name`, `status`, `criteria`) y si `status` puede ser `active|inactive`.

### 2.7 Generación de reportes (Semana 8)

| Tema | Para qué lo necesitas |
|------|----------------------|
| **Generación de CSV en Node.js**: `csv-stringify` o construcción manual | Reporte CSV |
| **Generación de PDF en Node.js**: `pdfkit` o `puppeteer` | Reporte PDF |
| **Content-Disposition header**: `res.setHeader('Content-Disposition', 'attachment; filename="..."` | Forzar descarga del archivo |
| **Streaming de archivos**: `res.pipe` | Archivos grandes sin saturar memoria |
| **Filtros aplicables al reporte**: query params (segmento, rango de fechas) | Reportes por criterios |

> **Recomendación**: empieza por CSV (más simple). Deja PDF para el final; si te falta tiempo, entrega solo CSV y documenta PDF como mejora.

### 2.8 Testing (Continuo)

| Tema | Para qué lo necesitas |
|------|----------------------|
| **Vitest** para componentes Vue | Tests de formulario de promoción, filtros |
| **Vue Test Utils**: `mount`, `setProps`, `trigger` | Tests de interacción |
| **Jest** para backend | Tests de `promotion.service.js` |
| **Mock de Prisma** | Testear service sin BD real |
| **Cobertura ≥ 70%** | Requisito de la iteración |

---

## 3. Tareas detalladas por iteración

### Iteración 1 — Lo que construyes tú

#### Tarea 1.1 — Componente de filtros de segmentación (Semana 3)

**Objetivo**: Frontend de los filtros de segmentación que se conecta con el endpoint de Junior 1.

**Pasos sugeridos**:
1. Coordina con Junior 1 el contrato de query params: `minVisits`, `minSpend`, `preference`. **Antes de codificar**, escríbanlo en un documento compartido.
2. Crea `components/ClientFilters.vue` con inputs para los tres filtros.
3. Emite eventos `filter-change` con el objeto `{ minVisits, minSpend, preference }`.
4. En la vista `ClientListView.vue` de Junior 1, este componente entra como hijo y le pasa los filtros al store.
5. Maneja el estado de "filtros activos" para permitir limpiarlos.

**Criterios de aceptación**:
- Cambiar `minVisits` actualiza la lista de clientes en tiempo real.
- Combinar los tres filtros funciona como AND.
- El botón "Limpiar filtros" restaura la lista completa.

**Temas a estudiar antes**: 2.1 (props, emits, watch), 2.2.

### Iteración 2 — Lo que construyes tú

#### Tarea 2.1 — Promotion CRUD (Semana 4)

**Objetivo**: Crear, editar, desactivar promociones con título, descripción, fechas de vigencia y tipo.

**Pasos sugeridos**:

Backend (en `server/src/modules/promotions/`):
1. Crea `promotion.routes.js` con `GET /`, `GET /:id`, `POST /`, `PUT /:id`, `PATCH /:id/deactivate`.
2. Crea `promotion.controller.js` que parsea y valida con Zod.
3. Crea `promotion.service.js` con CRUD usando Prisma. Estado por defecto: `active`.
4. El esquema Zod valida: `title` no vacío, `start_date < end_date`, `type` enum (descuento, bono, etc.).

Frontend (en `client/src/modules/promotions/`):
1. Crea `services/PromotionService.js`.
2. Crea `stores/promotions.js` con Pinia.
3. Crea `views/PromotionListView.vue`: tabla con promociones y estado activo/inactivo.
4. Crea `components/PromotionForm.vue`: formulario con datepickers.
5. Crea `views/PromotionEditView.vue`.

**Criterios de aceptación**:
- Crear una promoción la persiste con estado `active`.
- Desactivar (no borrar) lleva el estado a `inactive` y la quita de listas activas.
- Validar `start_date < end_date` rechaza promociones con fechas inválidas.

**Temas a estudiar antes**: 2.3 (formularios, fechas), 2.4, 2.5.

#### Tarea 2.2 — Asignación de promociones a segmentos (Semana 4)

**Objetivo**: Asignar una promoción a uno o varios segmentos, con validación cruzada.

**Pasos sugeridos**:
1. Crea endpoint `POST /api/promotions/:id/segments` con body `{ segmentIds: number[] }`.
2. En el service, antes de conectar, valida cada `segmentId` con `prisma.segment.findUnique` y que su `status` sea `active`.
3. Usa `prisma.promotion.update({ data: { segments: { connect: segmentIds.map(id => ({ id })) } } }`.
4. En el frontend, crea `components/PromotionAssignForm.vue`: multiselect de segmentos activos.
5. Maneja errores: 404 si un segmento no existe, 409 si está inactivo.

**Criterios de aceptación**:
- Asignar 3 segmentos a una promoción queda persistido.
- Intentar asignar un `segmentId` inexistente es rechazado con 404.
- Intentar asignar un segmento `inactive` es rechazado con 409.

**Temas a estudiar antes**: 2.5 (Prisma M:N, connect), 2.6.

#### Tarea 2.3 — Vigencia de promociones (Semana 5)

**Objetivo**: Lógica de inicio/fin de promociones basada en fechas.

**Pasos sugeridos**:
1. Agrega un getter en el store `activePromotions` que filtre promociones donde `now >= start_date && now <= end_date && status === 'active'`.
2. En el backend, agrega un endpoint `GET /api/promotions/active` que aplique el mismo filtro en la query de Prisma (`where: { AND: [...] }`).
3. Documenta el comportamiento: una promoción con `end_date` en el pasado y `status: 'active'` NO aparece en `active`, pero sí en el listado general.

**Criterios de aceptación**:
- Una promoción cuya `end_date` ya pasado no aparece en `/api/promotions/active`.
- Una promoción futura (`start_date` en el futuro) tampoco aparece en `/active`.
- Una promoción vigente (entre `start_date` y `end_date`) aparece.

**Temas a estudiar antes**: 2.3 (manipulación de fechas), 2.5.

#### Tarea 2.4 — Tests unitarios de Promotions (Continuo, cierra fin de I2)

**Mínimos a cubrir**:
- `PromotionService.create` con fechas inválidas rechazadas.
- `PromotionService.assignSegments` con segmento inexistente / inactivo.
- `PromotionForm.vue`: validación de campos, envío correcto, manejo de errores.
- Store de promociones: getters `activePromotions`.

### Iteración 3 — Lo que construyes tú

#### Tarea 3.1 — Reportes exportables (Semana 8)

**Objetivo**: Endpoint de exportación de reportes en CSV y PDF.

**Pasos sugeridos**:

Backend (en `server/src/modules/reports/`):
1. Crea `report.routes.js` con `GET /api/reports/clients.csv`, `GET /api/reports/promotions.pdf`.
2. Acepta query params para filtrar (`segmentId`, `startDate`, `endDate`).
3. Para CSV: usa `csv-stringify` con `res.setHeader('Content-Disposition', 'attachment; filename="reporte.csv"')` y haz pipe al response.
4. Para PDF: usa `pdfkit`; crea un documento con título, fecha de generación y tabla de datos.
5. Reutiliza las queries de `client.service.js` y `promotion.service.js`: no dupliques lógica, importa el service.

Frontend:
1. En `views/ReportsView.vue`, agrega botones "Exportar CSV" y "Exportar PDF".
2. Los botones abren `window.open('/api/reports/clients.csv?...')` o hacen fetch con blob.

**Criterios de aceptación**:
- El CSV se descarga y se abre en Excel con columnas correctas.
- El PDF se descarga con los datos y cabecera visible.
- Los filtros aplicados viajan al backend y el reporte respeta los filtros.

**Temas a estudiar antes**: 2.7.

#### Tarea 3.2 — Refinamiento de UX (Semana 8)

1. Prueba con un usuario real el flujo: crear promoción → asignar a segmento → verla en cliente.
2. Ajusta al menos 3 fricciones detectadas.

#### Tarea 3.3 — Documentación y manual (Semana 9)

1. Documenta tus endpoints en `docs/`: `/api/promotions/*`, `/api/reports/*`.
2. Escribe la sección del manual de usuario de Promociones y Reportes.

---

## 4. Definition of Done para tus tareas

1. ✅ Código en rama `feature/promotions-<descripción>` (o `feature/segmentation-`, `feature/reports-`), PR creado.
2. ✅ Tests unitarios pasan (cobertura ≥ 70% de tu módulo).
3. ✅ Senior aprobó el PR.
4. ✅ `npm run lint` sin errores.
5. ✅ Funcionalidad verificada en navegador.
6. ✅ Casos borde cubiertos.

---

## 5. Reglas de comunicación

| Práctica | Frecuencia | Quién |
|----------|-----------|-------|
| Daily standup | Diaria 15 min | Todos |
| Coordinación con Junior 1 | Inicio de I1 (contrato de segmentación) e I2 (modelo de Segment) | Junior 1 |
| Coordinación con Junior 3 | En I3 (Report Service compartido en `reports/`) | Junior 3 |
| Revisión de PR | ≤ 24h | Senior revisa tus PRs; tú revisas ≥ 1 PR/semana |

> **Regla de oro**: tu módulo tiene dos dependencias externas (Segment de Junior 1; Report Service compartido con Junior 3). Antes de codificar nada que las toque, acuerda el contrato por escrito (un mini-doc de 5 líneas) y compártelo.

---

## 6. Recursos de referencia

- [Vue 3 Documentation](https://vuejs.org/)
- [Pinia Documentation](https://pinia.vuejs.org/)
- [Prisma Documentation — Relations](https://www.prisma.io/docs/concepts/components/prisma-schema/relations)
- [csv-stringify](https://csv.js.org/stringify/)
- [PDFKit](http://pdfkit.org/)
- [Vitest](https://vitest.dev/)
- [Vue Test Utils](https://test-utils.vuejs.org/)

**Documentos del proyecto**:
- [Propuesta de Desarrollo](propuesta_desarrollo.md) — secciones 3.1 RF-03 y RF-05, 4.2, 6
- [Distribución del Equipo](distribucion_equipo.md) — secciones 2, 4, 5
- [Análisis del Problema Estructural](analisis_problema_estructural.md) — para entender el dolor de "promociones desorganizadas"