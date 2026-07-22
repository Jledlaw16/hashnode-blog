---
title: "El Camino APEX #2: Infraestructura sin excusas — de un ambiente gratuito a producción real"
slug: el-camino-apex-2-infraestructura-sin-excusas
tags: oracle-apex, oracle-cloud, oci, infraestructura, saas
subtitle: "El viaje real desde el workspace gratuito de Oracle hasta una instancia APEX paga en OCI — con los errores, las noches de pánico y las decisiones que nadie documenta."
saveAsDraft: false
---

Nadie te habla honestamente de infraestructura cuando empiezas. Te hablan del código, del diseño, de las funcionalidades. La infraestructura aparece más tarde, generalmente en el peor momento posible.

Este capítulo es sobre ese camino — el que fui construyendo a tropiezos, con decisiones forzadas por las circunstancias y aprendizajes que hoy no cambiaría por nada.

---

## El punto de partida: el workspace gratuito de Oracle

Cuando empecé a desarrollar con Oracle APEX, el workspace gratuito que Oracle ofrece fue mi primer hogar. Y es un regalo genuino para quien está empezando: cero costos, puedes crear aplicaciones, explorar, aprender, equivocarte sin consecuencias económicas.

Para alguien que está iniciando, es exactamente lo que necesita. No hay excusa para no comenzar.

El límite llega cuando quieres dar el siguiente paso.

---

## El primer cliente real y la primera decisión seria

Junto con una colega, tomamos nuestro primer cliente con una propuesta concreta: un sistema para gestionar inscripciones de estudiantes y otros registros de su negocio. Era un proyecto real, con un cliente real que tendría usuarios reales dependiendo del sistema.

Antes de comprometernos, me puse a leer. A documentarme. Y ahí encontré algo que cambió la dirección del proyecto: Oracle es explícito en que el workspace gratuito **no está recomendado para aplicaciones en producción**.

Eso no es un detalle menor. Significa que si Oracle programa un mantenimiento, tu cliente enfrenta un outage que tú no puedes anticipar, no puedes mitigar y no puedes resolver — porque no tienes administración del ambiente. No tienes ninguna palanca.

Eso no es una solución. Eso es una promesa que no puedes cumplir.

La decisión fue clara: había que buscar otra infraestructura.

---

## Oracle Cloud: el Lamborghini que le dimos a un cliente

Creé mi cuenta de Oracle Cloud. Y creé también una cuenta para la cliente. Levantamos una APEX instance pequeña en modo gratuito — suficiente para alojar la aplicación que necesitaba.

El proyecto fue un éxito. La cliente quedó satisfecha. Entregamos no solo el sistema sino la infraestructura completa: una cuenta Oracle Cloud con su propia APEX instance corriendo de forma estable. Para una cliente que no domina la parte técnica, eso era algo que pocas empresas le ofrecerían.

Internamente lo llamé el Lamborghini para el bebé. Era una infraestructura robusta, enterprise-grade, para una aplicación que en ese momento solo necesitaba cubrir funciones básicas. Pero eso es precisamente lo que le garantizaba estabilidad, control y capacidad de crecer.

En ese proceso también fui adquiriendo experiencia que no tenía: crear APEX instances, manejar temas de base de datos, administración de usuarios. Mi experiencia previa trabajando cerca de un DBA me dio bases — aquí las puse en práctica por primera vez de forma autónoma.

---

## El servidor que se apagaba solo

Con mi cuenta de Oracle Cloud ya activa, tenía mi propia APEX instance gratuita donde seguía desarrollando GESTIONA+. Iba avanzando, aunque no con la constancia que quería — la vida absorbe tiempo de maneras que uno no anticipa.

Fue entonces cuando descubrí el comportamiento que nadie te advierte claramente: **las instancias gratuitas de OCI se apagan automáticamente después de períodos de inactividad.**

Y llegó el día en que encendí la instancia y algo estaba mal. No podía acceder a mis aplicaciones.

Lo que siguió fueron semanas de diagnóstico, consultas, intentos. Semanas donde vi de cerca la posibilidad de perder todo lo que había construido. Esa sensación es difícil de describir — no es solo código lo que estaba en riesgo, era tiempo, energía, decisiones, aprendizajes acumulados.

No me rendí.

Eventualmente lo resolví. Logré acceder nuevamente a mis aplicaciones. Y esa experiencia me dejó dos hábitos que mantengo hasta hoy: hacer seguimiento activo para que el servidor no se apague por inactividad, y mantener **respaldos regulares de todas mis aplicaciones**. No como buena práctica opcional — como regla no negociable.

---

## El paréntesis de la vida y el regreso

Pasó un tiempo. La vida tiene esa forma de absorber las intenciones. GESTIONA+ quedó en pausa mientras otras responsabilidades ocupaban el espacio.

Pero llegó un punto en que decidí no seguir postergando.

Retomé el desarrollo. Y casi al mismo tiempo llegó algo que cambió el ritmo de todo: el boom de la inteligencia artificial. ChatGPT primero, luego Claude. De repente había herramientas que podían acelerar el desarrollo de maneras que yo no había imaginado.

Fue ahí donde me dije: *este es el momento. Hay que meterle turbo.*

---

## La decisión más costosa: integrar Claude vía MCP

Para integrar Claude con mi ambiente APEX a través de un MCP server necesitaba algo específico: el wallet de mi base de datos. Y ahí encontré el problema.

Mi APEX instance existente tenía un tipo de workload que no me permitía descargarlo. Leí, revisé mi configuración, consulté con colegas. La conclusión fue unánime: el workload type de mi instancia no era el correcto, y **no era posible cambiarlo una vez creada la instancia**.

Solo había un camino: crear una nueva APEX instance desde cero con los parámetros correctos.

Tomé la decisión. Y si bien el proceso era retador, lo hice con orden:

- Creé la nueva instancia con el workload type correcto
- Recreé los schemas, los objetos, los workspaces
- Migré las aplicaciones
- Coordiné una ventana de mantenimiento con mi cliente, comunicando los cambios con transparencia

---

## Lo que encontré durante la migración

Al revisar la aplicación de mi amigo antes de migrarla, encontré algo que no esperaba: tenía una funcionalidad de carga de fotos que guardaba las imágenes directamente en la base de datos. En menos de un mes de implementada esa función, ya había más de mil registros con su respectiva foto almacenada en la tabla.

Hice los números mentalmente. Si ese crecimiento continuaba, en meses estaría enfrentando un problema serio de almacenamiento — y no barato de resolver.

La migración fue la oportunidad perfecta para corregirlo antes de que se convirtiera en una crisis. Migré los archivos a **Oracle Object Storage**, los eliminé de la tabla y modifiqué el flujo de carga para que todos los archivos futuros fueran directo al bucket. El problema del storage exponencial quedó resuelto antes de que existiera.

Esa es una de las lecciones que más valoro: aprovechar los momentos de cambio para corregir lo que sabes que va a fallar más adelante.

---

## El costo como nueva realidad

Con la nueva instancia en modo pago y el servidor anterior todavía activo durante la transición, llegó una preocupación nueva: los costos de tener dos servidores corriendo en paralelo.

La solución fue un **scheduled start/stop** — configurar que la instancia se encienda y apague automáticamente según el horario real de uso. No pagar por horas en que nadie está conectado.

Una vez completada la migración, el servidor anterior se apagó definitivamente.

---

## Lo que aprendería si empezara hoy

Si alguien me pregunta cómo empezar con infraestructura para una aplicación APEX, esto es lo que digo:

**Empieza con el workspace gratuito de Oracle — sin culpa y sin excusas.** Es la plataforma perfecta para aprender, explorar y construir versiones iniciales. Aprovéchala.

**Cuando tengas un cliente real, investiga antes de comprometerte.** Entiende qué ambiente puedes controlar y cuál no. La estabilidad que le prometes a un cliente depende de la infraestructura que controlas.

**Haz backups desde el primer día.** No cuando casi pierdes todo — desde el primer día.

**Cada decisión de infraestructura tiene un costo futuro.** El workload type que eliges al crear tu instancia, el lugar donde guardas los archivos, el tamaño del storage — esas decisiones no son triviales. Tómate el tiempo de entenderlas antes de ejecutarlas.

**El momento de migrar es el momento de mejorar.** Cuando tengas que mover cosas, aprovecha para corregir lo que sabes que está mal. No solo muevas — mejora.

---

En el próximo capítulo: las decisiones de arquitectura dentro de la aplicación — por qué diseñé GESTIONA+ como multi-compañía y multi-sucursal desde el inicio, y qué hubiera pasado si no lo hubiera hecho.

---

*Jhosh Gomez Ledlaw es desarrollador Oracle APEX y fundador de GESTIONA+, una solución SaaS para emprendedores y pequeñas empresas.*

---

## Referencias

- [apex.oracle.com — Información del servicio gratuito y sus limitaciones](https://apex.oracle.com/en/learn/getting-started/apex-oracle-com/)
- [Always Free Oracle APEX Application Development en OCI](https://docs.oracle.com/en/cloud/paas/apex/gsadd/always-free-oracle-apex-application-development.html)
- [Programar inicio y parada automática en Autonomous Database](https://docs.oracle.com/en/cloud/paas/autonomous-database/adbsa/autonomous-auto-stop-start.html)
- [Oracle SQLcl — Documentación oficial](https://docs.oracle.com/en/database/oracle/sql-developer-command-line/)
- [OCI Object Storage — Documentación](https://docs.oracle.com/en-us/iaas/Content/Object/home.htm)
