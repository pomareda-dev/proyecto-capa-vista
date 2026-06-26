# Resumen de Propuestas de Solución y Elevator Speech

[<- Volver al README](../README.md)

## 1. Propuestas de Solución — Resumen

**Problema:** 6 gerencias y +1,200 empleados sobre sistemas aislados que no se comunican → datos duplicados, procesos manuales redundantes, atención inconsistente y decisiones sin datos confiables.

**Solución:** Un **CRM integrado** que centraliza la información del cliente en una sola fuente de verdad, automatiza procesos repetitivos y entrega dashboards en tiempo real.

**Arquitectura (en capas):**

| Capa | Tecnología |
|------|-----------|
| Vista (foco del proyecto) | Vue 3 (Composition API), Pinia, Tailwind CSS |
| Negocio | Node.js + Express, JWT + bcrypt, RBAC |
| Datos | MySQL 8 + Prisma ORM |

**Decisiones clave:**
- Monolito modular sobre microservicios — el equipo de 4 y el plazo no justifican el overhead.
- Vue 3 sobre React — menor curva de aprendizaje para 3 juniors.
- API REST sobre GraphQL — patrones CRUD predecibles, menor complejidad.

**Módulos:** Clientes · Segmentación · Promociones · Atención al Cliente (tickets) · Reportes y Dashboards · Auth y Roles (RBAC por gerencia).

**Requerimientos no funcionales:** consultas < 2s con 1,200 usuarios · OWASP Top 10 · disponibilidad 99.5% · cobertura de tests ≥ 70%.

**Plan de entregas (3 iteraciones · 9 semanas):**

1. **I1 (semanas 1-3):** Auth + RBAC + gestión y segmentación de clientes.
2. **I2 (semanas 4-6):** Promociones + tickets de atención integrados al perfil del cliente.
3. **I3 (semanas 7-9):** Dashboards en tiempo real + reportes exportables + despliegue a producción.

**Equipo y mitigación:** 1 senior (Tech Lead) + 3 juniors. TDD pragmático con CI que bloquea merges sin cobertura. Mitigaciones clave: pair programming las primeras 2 semanas, Code Review express (máx. 24h) y alcance cerrado por iteración. Ver [Análisis de Riesgo](analisis_de_riesgo.md).

---

## 1B. Propuestas de Solución — Versión Ultra Resumida

- Centralizar toda la información del cliente en un único CRM integrado para eliminar la duplicidad de datos entre las 6 gerencias.
- Construir una capa de vista en Vue 3 con módulos por gerencia (clientes, promociones, atención, reportes).
- Implementar autenticación con JWT y control de acceso por roles (RBAC) para diferenciar permisos por gerencia.
- Automatizar la segmentación de clientes por frecuencia, gasto y preferencias para campañas de marketing dirigidas.
- Digitalizar la atención al cliente con tickets de quejas y solicitudes que permitan escalamiento y seguimiento.
- Construir dashboards en tiempo real y reportes exportables para que las decisiones gerenciales se basen en datos confiables.
- Entregar el sistema en 3 iteraciones de 3 semanas: clientes, luego promociones y atención, finalmente reportes y despliegue.
- Asegurar la calidad con pruebas unitarias (cobertura ≥ 70%) y CI que bloquea merges sin cobertura.

---

## 2. Elevator Speech

> Entiendo lo frustrante que debe ser saber que cada día su equipo pierde horas valiosas buscando información entre sistemas que no se hablan, que un mismo cliente reciba respuestas distintas según quién lo atienda, y que las decisiones que usted debe tomar se apoyen en datos que no terminan de ser confiables. No es un problema de su gente, es un problema de herramientas. Por eso proponemos construir un CRM integrado que centraliza toda la información del cliente en un solo lugar, automatiza las tareas que hoy se hacen a mano y le entrega dashboards en tiempo real para que usted pueda decidir con datos reales, no con intuiciones. Lo hacemos en 3 iteraciones de 3 semanas, con un equipo de 4 personas y un senior que garantiza la calidad: en la primera ya tendrán autenticación y gestión de clientes funcionando, en la segunda vendrán las promociones y la atención al cliente, y en la tercera los reportes y el despliegue a producción. El resultado concreto es menos trabajo manual, mejor atención al cliente y, por primera vez, decisiones gerenciales basadas en datos confiables.

---

## 3. Referencias

- [Propuesta de Desarrollo](propuesta_desarrollo.md) — detalle completo de arquitectura, tecnologías y plan de iteraciones
- [Distribución del Equipo](distribucion_equipo.md) — roles, tareas por iteración y convenciones de trabajo
- [Problema de Atlantic City](problema_atlantic_city.md)
- [Análisis del Problema Estructural](analisis_problema_estructural.md)
- [Análisis de Riesgo](analisis_de_riesgo.md)
