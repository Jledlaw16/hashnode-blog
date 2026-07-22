---
title: "El Camino APEX #5: Desarrollar con IA — lo que ChatGPT y Claude cambiaron en mi flujo de trabajo"
slug: el-camino-apex-5-desarrollar-con-ia
tags: oracle-apex, inteligencia-artificial, claude, chatgpt, mcp, productividad
subtitle: "Cómo integré herramientas de IA a mi desarrollo en Oracle APEX, qué diferencia hay entre usarlas y usarlas bien, y por qué hoy no podría prescindir de ellas."
saveAsDraft: false
---

La inteligencia artificial llegó al mundo del desarrollo con mucho ruido. Promesas de automatización total, de código generado en segundos, de que los desarrolladores serían reemplazados. La realidad que viví es más matizada — y mucho más interesante.

Este capítulo es sobre cómo incorporé la IA a mi flujo de trabajo real como desarrollador Oracle APEX, qué funcionó, qué no, y cómo pienso en estas herramientas hoy.

---

## ChatGPT: útil, pero con una dependencia importante

Empecé a usar ChatGPT antes del boom masivo. Y siendo desarrollador de Oracle APEX — que implica full programación, corrección de errores, reestructuración de procesos, PL/SQL, lógica de negocio — encontré que era funcional pero exigente en un aspecto específico: **tenía que darle todos los detalles, por mínimos que fueran**.

Para obtener una respuesta objetiva y que cumpliera mis requerimientos, necesitaba construir el contexto completo desde cero en cada conversación. La estructura de mis tablas, las convenciones que uso, el comportamiento esperado, los casos borde. Si omitía algo, la respuesta llegaba con suposiciones que no correspondían a mi sistema.

No es una crítica — es la naturaleza de un modelo que no tiene acceso a tu ambiente. Pero para un desarrollador solo trabajando en sus horas libres, armar ese contexto cada vez tiene un costo real en tiempo y energía.

ChatGPT sigue siendo parte de mi flujo hoy. Pero su rol cambió.

---

## Claude + MCP: cuando la IA puede ver tu sistema

La diferencia fundamental con Claude fue el acceso directo a mi ambiente Oracle APEX a través de un MCP server.

Claude puede leer mis aplicaciones, ver las páginas, inspeccionar los objetos de base de datos, escribir y modificar código directamente. No le tengo que describir la estructura de mi sistema — él la ve. Eso elimina la capa de traducción que consumía tiempo y era fuente de errores de interpretación.

El impacto en mi velocidad de desarrollo fue inmediato. Lo que antes requería construir contexto manualmente, ahora parte del estado real del sistema.

Pero la herramienta más poderosa no es útil si no sabes usarla bien. Lo que fui aprendiendo con el tiempo es que la clave está en **cómo pides las cosas**, no solo en tener acceso.

---

## Mi estrategia actual: ChatGPT + Claude en tándem

El flujo que terminé adoptando combina ambas herramientas de forma deliberada:

**Paso 1 — Refinar el requerimiento con ChatGPT.**
Antes de pedirle algo a Claude, uso ChatGPT para estructurar y afinar mi requerimiento. Le explico lo que necesito, le pido que me ayude a ser más preciso, que identifique ambigüedades. El resultado es un prompt más claro y completo.

**Paso 2 — Ejecutar con Claude.**
Con el requerimiento bien definido, se lo paso a Claude. Él tiene el contexto del sistema, ejecuta los cambios y me muestra el resultado.

Puede parecer un paso extra innecesario, pero la calidad de lo que obtienes de Claude es directamente proporcional a la calidad del requerimiento que le das. El tiempo invertido en refinar antes ahorra mucho más tiempo después.

---

## Las reglas que no negocio

Con el tiempo fui estableciendo restricciones que protegen la integridad del sistema:

**Componentes nativos de APEX siempre.** No acepto implementaciones que metan HTML, CSS o JavaScript custom donde APEX tiene un componente nativo. Los componentes nativos son más fáciles de mantener, más coherentes visualmente y mejor integrados con el framework. Esta es una línea que no cruzo.

**DEV vs PROD: niveles de autonomía distintos.**
En el ambiente de desarrollo, Claude puede ejecutar acciones sobre la base de datos con más libertad — es el espacio para eso. En producción, cualquier acción sobre datos reales requiere mi aprobación explícita. Siempre. Sin excepciones.

Dependiendo del tipo de tarea también aplico modos distintos: autopilot para tareas rutinarias bien definidas, manual para cambios que tienen más impacto o que requieren mi revisión en cada paso.

**Desarrollo de páginas nuevas solo en ambiente de prueba.**
Cuando quiero que Claude construya una página completa, lo hace en un ambiente separado — una aplicación de test — nunca directamente sobre las aplicaciones productivas. Eso me da control para revisar antes de integrar.

---

## El skill de Oracle APEX que cambió la calidad de las respuestas

Una de las mejoras más significativas que implementé fue crear un **skill personalizado para Oracle APEX** en mi configuración de Claude.

El problema que resolvía era concreto: Claude a veces generaba respuestas incorrectas porque no tenía suficiente contexto sobre mis convenciones de desarrollo, mi estilo de programación, las restricciones específicas de mi sistema o los patrones que uso en GESTIONA+. Las respuestas eran técnicamente válidas para Oracle APEX en general, pero no para *mi* Oracle APEX.

El skill encapsula todo eso: mis convenciones de nomenclatura, los principios de desarrollo que aplico, las restricciones que tengo, los patrones preferidos. Cuando trabajo en GESTIONA+, ese contexto está disponible desde el primer momento — sin tener que repetirlo.

La diferencia en la precisión de las respuestas fue notable.

---

## Lo que la IA no reemplaza

Siendo honesto sobre lo que estas herramientas hacen y lo que no hacen:

La IA acelera la ejecución. Escribe código más rápido, sugiere soluciones, ayuda a depurar. Pero no toma las decisiones de arquitectura por ti. No sabe que tu cliente va a necesitar multi-sucursal en seis meses. No sabe que esa secuencia va a tener problemas de colisión cuando haya 50 usuarios simultáneos. No sabe que la lógica de negocio pertenece al procedimiento almacenado, no a la Dynamic Action.

Esas decisiones son tuyas. La IA las ejecuta — tú las diseñas.

Lo que sí puedo decir con claridad: hoy opero con una velocidad de desarrollo que antes hubiera requerido un equipo. El debugging es más rápido, la implementación de funcionalidades es más ágil, el tiempo entre "tengo una idea" y "está funcionando" se redujo de forma significativa.

Para un solopreneur construyendo un SaaS en sus horas libres, eso no es un detalle menor. Es lo que hace posible seguir avanzando sin necesitar contratar personas antes de que el negocio lo justifique.

---

## Lo que sigo mejorando

No tengo este proceso perfecto. Sigo aprendiendo cómo pedir mejor, cómo estructurar las restricciones, qué automatizar y qué mantener bajo control manual.

Lo que sí sé es que la IA llegó para quedarse en mi flujo de trabajo — no como reemplazo de mi criterio como desarrollador, sino como el par que ejecuta rápido mientras yo pienso en lo que realmente importa.

---

En el último capítulo de esta serie: el lado humano de todo esto — la pausa, el análisis, la resiliencia, y lo que me mantiene en el camino cuando la carga se siente demasiado pesada.

---

*Jhosh Gomez Ledlaw es Oracle ACE Apprentice, desarrollador Oracle APEX y fundador de GESTIONA+, una solución SaaS para emprendedores y pequeñas empresas.*
