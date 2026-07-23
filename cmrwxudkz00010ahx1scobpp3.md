---
title: "El Camino APEX #3: Pensar a futuro desde el primer día — arquitectura para crecer sin caos"
seoTitle: "El Camino APEX #3: Arquitectura para crecer"
seoDescription: "Multi-tenant desde el día uno y la lógica pesada en la base de datos. Decisiones que sostienen un SaaS."
datePublished: 2026-07-23T03:14:55.436Z
cuid: cmrwxudkz00010ahx1scobpp3
slug: el-camino-apex-3-arquitectura-para-crecer
cover: https://cdn.hashnode.com/uploads/covers/69dc626abe21b1bd01263ced/42b21dfd-3b12-4c14-af19-d364ec42efb4.png
ogImage: https://cdn.hashnode.com/uploads/og-images/69dc626abe21b1bd01263ced/924b7209-9f0e-4715-8d96-529b621e049b.png
tags: saas, oracle-database, multi-tenant, ci-cd, oracle-apex, arquitectura

---

Hay una trampa silenciosa en el desarrollo de software: la urgencia de hacer que algo funcione hoy te puede robar la capacidad de que ese algo sirva mañana o que escale de manera fluida.

En este capítulo quiero hablar de las decisiones que tomé — algunas a tiempo, otras después de entender lo que hubiera costado no tomarlas — sobre cómo estructurar GESTIONA+ para que crezca sin volverse caótico.

![](https://cdn.hashnode.com/uploads/covers/69dc626abe21b1bd01263ced/3f22d5e3-b5f6-470a-8146-ebe4d06688d6.png align="center")

* * *

## La decisión de multi-compañía y multi-sucursal

Desde el inicio supe que GESTIONA+ no podía ser una aplicación de un solo cliente. El modelo de negocio es suscripción — lo que significa múltiples clientes en el mismo sistema, cada uno con sus datos aislados, pero todos beneficiándose de las mismas mejoras.

La pregunta era: ¿cómo organizas eso?

La alternativa obvia sería un schema por cliente, un workspace por cliente, una aplicación por cliente. Técnicamente funciona. Pero hagamos el cálculo mental: si tienes 20 clientes y necesitas corregir un bug, haces ese cambio 20 veces. Si agregas una funcionalidad nueva, la despliega 20 veces. Si cambias algo en la base de datos, ejecutas el DDL 20 veces. El mantenimiento se convierte en el trabajo principal, no el desarrollo.

Elegí el camino contrario: **una sola aplicación, multi-compañía, multi-sucursal**.

La estructura base es simple:

```plaintext
COMPANIA
  └── SUCURSAL
        └── Todos los registros del negocio
```

Cada registro en el sistema lleva su `COMPANY_ID` y su `BRANCH_ID`. Cada consulta filtra por ambos. El usuario solo ve lo que pertenece a su compañía y sucursal — sin excepciones.

Esto significa que cuando mejoro algo en GESTIONA+, todos los clientes se benefician al mismo tiempo. Un solo deployment, impacto para todos. Eso es lo que hace sostenible un SaaS siendo un equipo de uno.

¿Hay complejidad en este modelo? Sí. A futuro habrá decisiones difíciles — permisos entre sucursales, reportes consolidados por compañía, casos donde un usuario necesita ver más de una sucursal. Soy honesto en que aún tengo camino por labrar y retos que no he enfrentado todavía. Pero estoy convencido de que esta base es más manejable que la alternativa.

* * *

## Los estándares que nadie menciona pero todos necesitan

Hay un aspecto del desarrollo que se aprende tarde si nadie te lo enseña explícitamente: los estándares de base de datos importan más de lo que parecen al inicio.

Cuando tienes cinco tablas, la nomenclatura es un detalle menor. Cuando tienes cincuenta tablas con relaciones cruzadas, la nomenclatura es lo que te permite entender el sistema sin tener que leerlo completo cada vez que tocas algo.

Estas son las decisiones que fui tomando y que hoy son parte de mi cultura de desarrollo:

### Nomenclatura consistente

Todos los nombres de tablas siguen el mismo patrón. Las columnas de llaves foráneas reflejan el nombre de la tabla que referencian. Si existe `COMPANIA` con llave `COMPANY_ID`, cualquier tabla que la referencie usa `COMPANY_ID` — nunca `ID_EMPRESA`, nunca `COD_CIA`, nunca algo inventado en el momento.

La consistencia no es estética. Es lo que te permite navegar el modelo de datos sin un mapa.

### Secuencias y colisiones

En una aplicación SaaS con múltiples usuarios activos, las secuencias de base de datos necesitan pensarse con cuidado. Si dos usuarios acceden a la misma pantalla al mismo tiempo y ambos generan un nuevo registro, ¿qué pasa con los IDs?

Oracle maneja esto bien con secuencias correctamente configuradas pero tienes que saber que el problema existe para diseñar alrededor de él. Una secuencia con `CACHE` configurado adecuadamente y sin dependencias de aplicación en el valor exacto del siguiente número te evita problemas de concurrencia que son difíciles de diagnosticar en producción.

### Documentación como hábito

Mantener documentación del sistema no es algo que hago porque alguien me lo exige. Lo hago porque yo mismo soy el que vuelve tres meses después a entender qué hace ese proceso almacenado que escribí a las 11 de la noche y se podría pensar que como soy el desarrollador me acordare, pues eso no siempre es así y es mejor anticiparse, cuando tengas múltiples tablas, paquetes, procesos, clientes viene bien tener una documentación que ayude a recordar de manera rápida que fue implementado y como.

Un comentario en el código explicando el *por qué* de una decisión, un README actualizado con el modelo de datos, notas sobre las decisiones que se tomaron y las que se descartaron — todo eso es tiempo invertido que se recupera multiplicado.

* * *

## La lógica vive en la base de datos, no en las páginas

Este es uno de los principios que más defiendo y que más diferencia hace en proyectos Oracle APEX de largo plazo: **la lógica pesada pertenece a la base de datos**.

Cuando empiezas a desarrollar en APEX, la tentación es poner toda la lógica directamente en la página — validaciones en procesos de página, cálculos en items computados, reglas de negocio en Dynamic Actions encadenadas. Funciona. Pero a medida que el sistema y la data crece, esa estrategia se convierte en un laberinto.

El principio que aplico en GESTIONA+ es claro: si algo es complejo, va a la base de datos.

**Procedimientos almacenados** para los procesos de negocio que tienen múltiples pasos — una venta, un cierre de caja, una transferencia entre sucursales. La página llama al procedimiento, el procedimiento hace el trabajo, la página muestra el resultado.

**Funciones** para los cálculos que se reutilizan en múltiples lugares — descuentos, impuestos, totales. Una sola definición, usada desde donde sea necesario.

**Paquetes** para agrupar la lógica relacionada. Un paquete por módulo funcional mantiene el código organizado y fácil de encontrar.

¿Qué queda en la página? La presentación. La interacción con el usuario. El flujo de navegación. Nada más.

Las ventajas de este enfoque son múltiples:

*   **Las páginas se vuelven simples.** Fáciles de entender, fáciles de mantener, fáciles de modificar sin miedo de romper algo.
    
*   **La lógica es reutilizable.** Si necesitas el mismo proceso desde dos páginas diferentes, no lo duplicas — llamas al mismo procedimiento.
    
*   **El debugging es más claro.** Cuando algo falla, sabes dónde buscar: en el procedimiento, no entre diez Dynamic Actions encadenadas.
    
*   **Las herramientas de base de datos te ayudan.** Puedes probar un procedimiento directamente desde SQL Developer o SQLcl sin necesidad de navegar la aplicación completa.
    

Una página de APEX complicada con regiones innecesarias y sin identificación, procesos encadenados y lógica dispersa es técnicamente igual de funcional que una página limpia — hasta que algo falla o necesitas modificarla. En ese momento, la diferencia es enorme.

Construye páginas simples. Pon la inteligencia en la base de datos. Oracle te da las herramientas para hacerlo bien, solo hay que usarlas.

* * *

## Que le diría a un desarrollador que empieza hoy

**Planificar vale la pena**. Siempre van a surgir cosas en el camino — un cliente/usuario con un caso especial, una funcionalidad que no anticipaste, un módulo completamente nuevo. Pero cuando tienes una ruta clara y un modelo definido desde el inicio, cada cosa nueva encuentra su lugar. No negocias con el caos, lo integras con orden.

**Define tu modelo según tus necesidades antes de escribir código y crear tablas.** No después, no cuando ya tienes una aplicación productiva con 30 tablas, usuarios ingresando al sistema que se pueden ver afectados y te das cuenta de que ninguna tiene un `company_id`. Hoy en día hay herramientas que te pueden ayudar con esto, pero nunca dejes de entender como funciona porque si importa y será necesario a futuro.

**Los estándares son más fáciles de crear que de corregir.** Una convención de nomenclatura que estableces en los objetos, te salva de horas de refactoring cuando tienes mas de cincuenta objetos (tablas, procedimientos, secuencias, trigger, jobs, entre otros) creados.

**Automatiza el deployment desde el principio.** No tienes que llegar al nivel de CI/CD completo en el día uno, pero sí tienes que tener un proceso repetible y documentado para pasar cambios de DEV a PROD sin que sea una tarea engorrosa con falencias por que se te olvido pasar una secuencia y cuando el usuario esta creando un registro comienza el caos.

**Diseña para el mantenimiento, no solo para la entrega.** El código que entregas hoy lo va a mantener alguien, probablemente tú mismo dentro de seis meses. Escríbelo pensando en ese momento.

* * *

En el próximo capítulo: la implementación más retadora que he enfrentado en GESTIONA+ — el punto de venta, la integración con hardware real y todo lo que nadie documenta sobre conectar software con una caja registradora.

* * *

*Jhosh Gomez Ledlaw - Desarrollador Oracle APEX y creador de GESTIONA+, una solución SaaS para emprendedores y pequeñas empresas.*

* * *

## Referencias

*   [Oracle SQLcl — Documentación oficial](https://docs.oracle.com/en/database/oracle/sql-developer-command-line/)
    
*   [Oracle SQLcl — Descarga y recursos](https://www.oracle.com/database/sqldeveloper/technologies/sqlcl/)
    
*   [CREATE SEQUENCE — Parámetro CACHE en Oracle Database](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/CREATE-SEQUENCE.html)
    
*   [Oracle APEX App Builder User's Guide 24.2](https://docs.oracle.com/en/database/oracle/apex/24.2/htmdb/)
    
*   [Oracle PL/SQL Language Reference — Packages](https://docs.oracle.com/en/database/oracle/oracle-database/19/lnpls/)