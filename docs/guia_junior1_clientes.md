# Guía de Trabajo — Junior 1: Módulo de Clientes

[<- Volver al README](../README.md)

## Para quién es esta guía

Esta guía es para **Junior 1**, responsable del **Módulo de Clientes**: toda la funcionalidad de gestión de perfiles, segmentación y perfil 360° del cliente en Atlantic City CRM.

Tu trabajo es de extremo a extremo (fullstack dentro de tu módulo): tocas Vue 3 en el frontend, Express en el backend y Prisma en la base de datos. El Senior hará la arquitectura base y te dará el esquema `schema.prisma`, pero tú implementas la lógica de tu módulo sobre ese esquema.

> **Lectura obligatoria antes de empezar**: [Propuesta de Desarrollo](propuesta_desarrollo.md) (secciones 3, 4 y 6) y [Distribución del Equipo](distribucion_equipo.md) (secciones 2 y 4).

---

## 1. ¿Qué vas a construir?

Tu módulo aparece en tres momentos del proyecto:

| Iteración | Qué entregas | Semanas |
|-----------|--------------|---------|
| I1 | App Shell (Layout, Sidebar, Header, Router con guards) | 1 |
| I1 | Client CRUD (crear, listar, editar, buscar clientes) | 2 |
| I1 | Client Detail con tabs (datos, visitas, promociones, tickets) | 3 |
| I1 | Deduplicación: validación de DNI/email duplicados | 3 |
| I1 | Segmentación: filtros por frecuencia, gasto, preferencias | 3 (con Junior 2) |
| I2 | Integración en Client Detail: tabs de promociones y tickets | 6 |
| I3 | Refinamiento de UX, documentación de tu módulo, manual de usuario | 7-9 |
| I3 | Tests unitarios de tu módulo ≥ 70% | Continuo |

**Resultado final**: un usuario autenticado (Operaciones o Servicio al Cliente) puede crear, buscar, editar y segmentar clientes, ver su perfil 360° con sus promociones y tickets, y el dato queda persistido en MySQL sin duplicados.

---

## 2. Temas que tienes que estudiar o repasar

No es opcional: sin estos temas no vas a poder completar tus tareas. Estudia en orden, porque cada tema se apoya en el anterior.

### 2.1 Fundamentos de Vue 3 (Antes de la Semana 1)

| Tema | Para qué lo necesitas | Recursos sugeridos |
|------|----------------------|-------------------|
| **Composition API**: `setup`, `ref`, `reactive`, `computed`, `watch` | Todos tus componentes los usan | [Vue 3 Guide — Composition API](https://vuejs.org/guide/extras/composition-api-faq.html) |
| **Single-File Components (SFC)**: `<template>`, `<script setup>`, `<style scoped>` | Estructura base de cada componente | [Vue SFC](https://vuejs.org/guide/scaling-up/sfc.html) |
| **Props y emits**: comunicación padre-hijo | Tabla → formulario de edición, filtros → listado | [Component Props](https://vuejs.org/guide/components/props.html), [Component Events](https://vuejs.org/guide/components/events.html) |
| **Slots**: contenido proyectado | Layout y tabs reutilizables | [Slots](https://vuejs.org/guide/components/slots.html) |
| **v-model en componentes** | Formularios de creación/edición de cliente | [Form Input Bindings](https://vuejs.org/guide/elements/forms.html) |
| **Lifecycle hooks**: `onMounted`, `onUnmounted` | Cargar datos iniciales, limpiar listeners | [Lifecycle](https://vuejs.org/guide/essentials/lifecycle.html) |

**Ejercicio práctico antes de tu primera tarea**: crea un componente `HelloUser.vue` que reciba un `name` por prop, muestre un input con `v-model` y un contador con `ref`. Publícalo en tu rama `feature/<tu-nombre>-practice`. Esto valida que tu entorno Vue funciona.

### 2.2 Vue Router y guards (Semana 1)

| Tema | Para qué lo necesitas |
|------|----------------------|
| **Router básico**: `createRouter`, `routes`, `router-view`, `router-link` | App Shell y navegación |
| **Route params y query**: `useRoute`, `useRouter` | Detalle de cliente por ID, filtros por URL |
| **Navigation guards**: `beforeEach`, `meta` por ruta | Proteger rutas por rol (RBAC) |
| **Lazy loading de vistas**: `defineAsyncComponent` o `() => import()` | Performance del dashboard |

### 2.3 Estado global con Pinia (Semana 1-2)

| Tema | Para qué lo necesitas |
|------|----------------------|
| **Definir un store**: `defineStore` con setup syntax | Store de clientes (listado, detalle, filtros) |
| **State, getters, actions** | Cargar clientes, filtrar, guardar el cliente seleccionado |
| **Uso del store en componentes**: `storeToRefs` para reactividad | Conectar el store con la vista |
| **Store compartido entre vistas** | Listado y detalle comparten el mismo store |

> **Gotcha que vas a tropezar**: si haces `const { clients } = store` pierdes reactividad. Tienes que usar `storeToRefs(store)` para destructurar manteniendo reactividad.

### 2.4 HTTP con Axios (Semana 2)

| Tema | Para qué lo necesitas |
|------|----------------------|
| **Instancia de axios**: `axios.create`, base URL, headers | Capa de servicios HTTP |
| **Interceptors**: request (atachar JWT), response (manejar errores 401) | Auth en cada petición |
| **Métodos CRUD**: `get`, `post`, `put`, `delete` | Todos los endpoints de Client |
| **Manejo de errores**: `try/catch` + respuesta del backend | Feedback al usuario |

### 2.5 Backend: Express + Prisma (Semana 2-3)

| Tema | Para qué lo necesitas |
|------|----------------------|
| **Express**: routers, controllers, middleware | Tu `client.controller.js` y `client.routes.js` |
| **Prisma**: modelos, `prisma.client.findMany`, `findUnique`, `create`, `update`, `delete` | Acceso a la base de datos |
| **Prisma relations**: `include`, `select` | Traer clientes con sus visitas, segmentos, tickets |
| **Prisma filters**: `where` con operadores (`contains`, `gt`, `in`) | Búsqueda y segmentación |
| **Paginación con Prisma**: `skip`, `take` | Lista paginada de clientes |
| **Zod para validación**: esquema de cliente, tipos inferidos | Validación en controller |

> **Patrón a replicar**: el Senior creará el módulo Auth como referencia. Estudia su código y replica la misma estructura (controller → service → prisma) en Client. No inventes tu propio patrón.

### 2.6 Testing (Continuo)

| Tema | Para qué lo necesitas |
|------|----------------------|
| **Vitest**: `describe`, `it`, `expect`, `vi.mock` | Tests de componentes Vue |
| **Vue Test Utils**: `mount`, `shallowMount`, `setProps`, `trigger` | Tests de tus componentes |
| **Jest**: API idéntica a Vitest | Tests de tu Client Service en backend |
| **Mock de Prisma**: `vi.mock('@prisma/client')` | Testear service sin BD real |
| **Cobertura**: `--coverage` | Verificar ≥ 70% |

---

## 3. Tareas detalladas por iteración

### Iteración 1 — Lo que construyes tú

#### Tarea 1.1 — App Shell (Semana 1, con supervisión del Senior)

**Objetivo**: Layout principal con sidebar por gerencia, header con datos del usuario y área de contenido dinámica. Router con guards de auth.

**Pasos sugeridos**:
1. Estudia el componente `auth/views/LoginView.vue` que el Senior entregará primero.
2. Crea `components/layout/AppShell.vue` con tres slots: sidebar, header, content.
3. Crea `components/layout/SidebarNav.vue` leyendo el rol del usuario desde el store de Auth para mostrar solo las rutas permitidas.
4. Crea `components/layout/AppHeader.vue` mostrando usuario logueado (desde store de Auth) y botón de logout.
5. Define las rutas en `router/index.js` con `meta: { requiresAuth: true, roles: ['Operaciones', ...] }`.
6. Implementa el guard `beforeEach` que valide token + rol contra el meta de la ruta.

**Criterios de aceptación**:
- Un usuario sin token es redirigido a `/login`.
- Un usuario de Marketing no ve la ruta de Clientes en el sidebar...ERR ÓË ëîría que la viera. Revisa la matriz RBAC con el Senior.
- El logout limpia el token y redirige a `/login`.

**Temas a estudiar antes**: 2.1, 2.2.

#### Tarea 1.2 — Client CRUD (Semana 2)

**Objetivo**: Crear, listar, editar y buscar clientes. End-to-end: frontend, backend y BD.

**Pasos sugeridos**:

Backend (en `server/src/modules/clients/`):
1. Crea `client.routes.js` con `GET /api/clients`, `POST /api/clients`, `GET /api/clients/:id`, `PUT /api/clients/:id`, `DELETE /api/clients/:id`.
2. Crea `client.controller.js` que parsea la request y llama al service.
3. Crea `client.service.js` con la lógica CRUD usando Prisma (`prisma.client.findMany`, `create`, etc.).
4. Define un esquema Zod `clientSchema` y úsalo para validar entrada en el controller.
5. Implementa paginación con `skip` y `take` en `GET /api/clients`.

Frontend (en `client/src/modules/clients/`):
1. Crea `services/ClientService.js` con métodos `list`, `get`, `create`, `update`, `remove` usando axios.
2. Crea `stores/clients.js` con Pinia (setup syntax): estado `clients`, `selectedClient`, getters `totalPages`, action `fetchClients`.
3. Crea `views/ClientListView.vue`: tabla paginada con búsqueda por nombre/DNI.
4. Crea `components/ClientForm.vue`: formulario de creación/edición con validación visual.
5. Crea `views/ClientEditView.vue` que usa `ClientForm` en modo edición.

**Criterios de aceptación**:
- Crear un cliente desde el frontend lo persiste en MySQL.
- Buscar por nombre filtra la lista en tiempo real.
- La paginación funciona al cambiar de página.
- Los errores de validación (DNI sin formato, email inválido) muestran mensaje en el formulario.

**Temas a estudiar antes**: 2.3, 2.4, 2.5.

#### Tarea 1.3 — Client Detail con tabs (Semana 3)

**Objetivo**: Vista de detalle del cliente con navegación por tabs: datos, visitas, promociones, tickets.

**Pasos sugeridos**:
1. Crea `views/ClientDetailView.vue` que recibe `:id` por route param y carga el cliente desde el store.
2. Cada tab es un componente hijo que recibe el `clientId` por prop:
   - `ClientDataTab.vue`: datos básicos (reutiliza `ClientForm` en modo lectura).
   - `ClientVisitsTab.vue`: lista las visitas del cliente (consulta Prisma `include: { visits: true }`).
   - `ClientPromotionsTab.vue`: placeholder inicial; se llena en I2 cuando Junior 2 entregue promociones.
   - `ClientTicketsTab.vue`: placeholder inicial; se llena en I2 cuando Junior 3 entregue tickets.
3. Implementa navegación con `<RouterLink>` o `v-if`/`v-show` para alternar tabs.

**Criterios de aceptación**:
- Navegar a `/clients/:id` muestra el detalle del cliente.
- Los tabs de datos y visitas muestran información real persistida.
- Los tabs de promociones y tickets muestran un estado vacío elegante ("Sin promociones asignadas") hasta que I2 los llene.

**Temas a estudiar antes**: 2.1, 2.3.

#### Tarea 1.4 — Deduplicación (Semana 3)

**Objetivo**: Validar que un cliente no se registre dos veces con el mismo DNI o email.

**Pasos sugeridos**:
1. En `client.service.js`, antes de `prisma.client.create`, consulta `findUnique` por `dni` y por `email`.
2. Si existe, lanza un error tipado `DuplicateClientError` con el campo conflictivo.
3. El controller mapea el error a HTTP 409 con un mensaje claro.
4. En el frontend, captura el 409 en `ClientForm` y muestra el error junto al campo correspondiente.

**Criterios de aceptación**:
- Intentar registrar un DNI existente es rechazado con 409.
- Intentar registrar un email existente es rechazado con 409.
- El mensaje del frontend indica exactamente qué campo está duplicado.

**Temas a estudiar antes**: 2.5 (manejo de errores y códigos HTTP), 2.4.

#### Tarea 1.5 — Segmentación (Semana 3, en coordinación con Junior 2)

**Objetivo**: Filtros por frecuencia de visitas, nivel de gasto y preferencias.

**Pasos sugeridos**:
1. Coordina con Junior 2: él hará el componente de filtros del frontend; tú implementas la query en el backend.
2. En `GET /api/clients`, acepta query params `minVisits`, `minSpend`, `preference` y construye el `where` de Prisma.
3. Usando la relacion `visits`, agrega condiciones: `where: { visits: { some: { amount_spent: { gte: minSpend } } } }`.
4. Documenta el endpoint con ejemplos en `docs/` (sección de tu módulo).

> **Coordinación crucial**: Junior 2 necesita saber el contrato exacto de query params. Escríbelo en un documento antes de codificar y compártelo con él.

**Criterios de aceptación**:
- Filtrar "clientes con ≥ 5 visitas" devuelve solo esos.
- Filtrar "clientes con gasto ≥ USD 1000" devuelve los correctos.
- Combinar filtros funciona (AND lógico).

**Temas a estudiar antes**: 2.5 (filtros avanzados de Prisma, relations).

### Iteración 2 — Integración

#### Tarea 2.1 — Tabs de promociones y tickets en Client Detail (Semana 6)

**Objetivo**: Conectar los componentes de Junior 2 (promociones) y Junior 3 (tickets) dentro de tu `ClientDetailView`.

**Pasos sugeridos**:
1. Acuerda con Junior 2 y Junior 3 los props que sus componentes esperan (`clientId`, eventos emitidos).
2. Sustituye los placeholders del Tarea 1.3 por los componentes reales de ellos.
3. Verifica que cada tab carga al montarse (lazy load) y no todos de golpe.

**Criterios de aceptación**:
- El tab de promociones muestra las promociones asignadas a ese cliente.
- El tab de tickets muestra las quejas/solicitudes de ese cliente.
- Cambiar de tab no recarga la página entera.

**Temas a estudiar antes**: 2.1 (componentes, props, slots), lazy loading.

### Iteración 3 — Refinamiento

#### Tarea 3.1 — Refinamiento de UX (Semana 8)

1. Haz una prueba con un usuario real (puede ser un compañero) usando el flujo: login → crear cliente → ver detalle → filtrar.
2. Anota fricciones: mensajes confusos, flujos cortados, validaciones que no ayudan.
3. Ajusta al menos 3 puntos de fricción detectados.

#### Tarea 3.2 — Tests unitarios ≥ 70% (Continuo, cierra en I3)

**Mínimos a cubrir**:
- `ClientService.list` con paginación y filtros.
- `ClientService.create` con duplicado rechazado.
- `ClientForm.vue`: validación de campos, envío correcto, manejo de error 409.
- `client.service.js` (backend): CRUD con Prisma mockeado.

#### Tarea 3.3 — Documentación y manual (Semana 9)

1. Documenta tu módulo en `docs/`: endpoints, esquemas Zod, flujo de creación de cliente.
2. Escribe la sección del manual de usuario correspondiente a Clientes: capturas y pasos.

---

## 4. Definition of Done para tus tareas

Una tarea tuya se considera **hecha** cuando (copiado de la [Distribución del Equipo](distribucion_equipo.md) sección 5.2):

1. ✅ El código está en una rama `feature/clients-<descripción>` y el PR está creado.
2. ✅ Todos tus tests unitarios pasan (cobertura ≥ 70% del módulo).
3. ✅ El Senior aprobó el PR (al menos 1 aprobación).
4. ✅ No hay errores de linting (`npm run lint`).
5. ✅ La funcionalidad fue verificada manualmente en el navegador.
6. ✅ Los casos borde discutidos en la tarea están cubiertos.

---

## 5. Reglas de comunicación

| Práctica | Frecuencia | Quién |
|----------|-----------|-------|
| Daily standup | Diaria 15 min | Todos |
| Dudas puntuales | Cuando aparezcan | Primero al Slack del equipo; si no responden en 2h, al Senior en directo |
| Revisión de PR | ≤ 24h de respuesta | Senior revisa todos tus PRs; tú revisas ≥ 1 PR/semana de Junior 2 o Junior 3 |
| Pair programming | Semanas 1-2 con Senior | Para desbloquearte con Vue/Prisma |

> **Regla de oro**: si estás bloqueado más de 2 horas en algo, pide ayuda. Bloquearte más tiempo no demuestra autonomía, demuestra falta de comunicación.

---

## 6. Recursos de referencia

- [Vue 3 Documentation](https://vuejs.org/) — lectura inicial obligatoria
- [Pinia Documentation](https://pinia.vuejs.org/)
- [Vue Router Documentation](https://router.vuejs.org/)
- [Prisma Documentation](https://www.prisma.io/docs)
- [Vitest Documentation](https://vitest.dev/)
- [Vue Test Utils](https://test-utils.vuejs.org/)

**Documentos del proyecto**:
- [Propuesta de Desarrollo](propuesta_desarrollo.md) — secciones 3, 4, 6
- [Distribución del Equipo](distribucion_equipo.md) — secciones 2, 4, 5
- [Análisis del Problema Estructural](analisis_problema_estructural.md) — para entender por qué tu módulo es la fuente única de verdad
- [Análisis de Riesgo](analisis_de_riesgo.md) — RP-04 (datos sucios) justifica tu tarea de deduplicación