# Análisis de Riesgo - Atlantic City

[<- Volver al README](../README.md)

| ID | RIESGO IDENTIFICADO | PLAN DE ACCIÓN PARA MITIGARLO | CRITICIDAD |
|----|---------------------|-------------------------------|------------|
|1|**Pérdida o corrupción** de datos durante la migración del sistema centralizado|Crear una estrategia de migración granulada para poder migrar la data de a pocos para poder ir verificando|Alto|
|2|**Brecha de seguridad** en información personal y financiera de los clientes debido a sistemas fragmentados.|Seguir estándares de seguridad conocidos como por ejemplo OWASP Top10:2025.|Alto|
|3|**Resistencia al cambio** por parte de los 1200 empleados y 6 gerencias, causando una baja adopción.|Dar charlas a los empleados encargados del sistema para concientizar la importancia de tener un sistema centralizado. Se presentará los beneficios y una guía de uso adecuada.|Leve|
|4|**Retraso en el desarrollo** por la curva de aprendizaje de nuevas tecnologías.|Coordinar capacitaciones frecuentes para el equipo de desarrollo y priorizar las tecnologías ya conocidas.|Moderado|
|5|**Mala definicion de las caracteristicas y funciones del sistema** que ocasionean cambios no contemplados por Atlantic City.|Realizar un plan de trabajo detallado que indique todas las características y funciones que se estarán entrendo, así como el procedimiento para abordar solicitudes fuera del plan.|Moderado|
|6|**Que el proveedor de infraestructura en la nube cese sus operaciones repentinamente** y se tenga que migrar toda la data a un nuevo proveedor.|Preparar un plan de contingengia para el escenario en el que se tenga que migrar o cambiar de infraestructura, ya sea a un nuevo proveedor o a una infraestructura local.|Alto|
|7|**Caída en el rendimiento operativo** por obsolescencia de la infraestructura local.|Restringir el acceso a los datos según el puesto de cada trabajador (roles estrictos).|Moderado|
|8|**Brechas de seguridad** y cumplimiento normativo al centralizar datos sensibles.|Revisar el internet y las computadoras de las salas de Atlantic City antes de lanzar el sistema para identificar cuáles necesitan mejora o cambiar.|Alto|
|9|**Falla en el sistema operativo** por errores en la configuracion del sistema.|Mantener actualizados los sistemas, los servidores y los frameworks utilizados para evitar problemas futuros.|Moderado|
