---
title: "SaaS con Oracle APEX 24.2: Arquitectura y Lecciones Reales"
slug: saas-con-oracle-apex-24-2
tags: oracle-apex, saas, oracle, plsql, oracle-database
subtitle: "Cómo construí una app SaaS en producción: arquitectura multi-ambiente, multi-tenant y las lecciones que aprendí."
saveAsDraft: false
---

## Introducción

Durante los últimos meses he estado construyendo una aplicación SaaS de producción para pequeñas y medianas empresas usando Oracle APEX 24.2 como plataforma principal de desarrollo. Para alguien que venía de un mundo donde "SaaS" significa casi obligatoriamente Node.js + React o algún stack cloud-native, la elección generó más de una pregunta — pero los resultados han sido lo suficientemente sólidos como para documentarlos.

Este artículo cubre las decisiones arquitectónicas que tomé, cómo estructuré los ambientes de desarrollo y producción, y las lecciones difíciles que solo aprendí después de tener clientes reales en el sistema.

---

## ¿Por qué Oracle APEX para una aplicación SaaS?

La pregunta obvia. Al iniciar el proyecto evalué varias opciones: un stack custom (Node/Next.js + PostgreSQL), una plataforma low-code genérica, y Oracle APEX 24.2.

APEX ganó por tres razones concretas:

**1. Velocidad hasta producción.** Los componentes nativos de APEX — Interactive Reports, Interactive Grids, páginas de formulario — cubren el 80% de lo que una app de negocio necesita desde el primer día. Construir funcionalidad equivalente desde cero toma semanas por feature.

**2. Integración con Oracle Database.** Mis clientes necesitan integridad transaccional real. PL/SQL + Oracle DB ofrece compliance ACID, capacidades de query sofisticadas y décadas de fiabilidad probada en producción.

**3. Costo total de operación.** Para un desarrollador independiente construyendo SaaS para PYMEs, el modelo de licenciamiento de APEX es significativamente más económico que el costo de infraestructura de stacks comparables.

Lo que me sorprendió: APEX 24.2 es considerablemente más capaz de lo que su reputación sugiere. Las actualizaciones en Universal Theme 42, la API JavaScript mejorada y el Interactive Grid revisado lo hacen genuinamente competitivo para aplicaciones de datos intensivos.

---

## Arquitectura: Misma Base de Datos, Schemas Separados

La decisión más importante para un SaaS multi-ambiente en APEX es cómo aislar desarrollo de producción. Llegué a este patrón:

```
Oracle Database Instance
├── DEV_SCHEMA        ← Workspace de desarrollo (App ID: 100)
│   ├── Tablas (con datos de prueba)
│   └── Paquetes PL/SQL
└── PROD_SCHEMA       ← Workspace de producción (App ID: 1000)
    ├── Tablas (estructura idéntica, datos reales de clientes)
    └── Paquetes PL/SQL (desplegados por separado)
```

**¿Por qué la misma base de datos?** Costo y simplicidad. En una etapa temprana de SaaS, mantener dos instancias Oracle separadas es caro. El aislamiento por schema ofrece separación suficiente mientras la infraestructura se mantiene mínima.

**¿Por qué App IDs distintos?** Las aplicaciones APEX están vinculadas a workspaces, y los workspaces mapean a schemas. Usar IDs distintos (100 para DEV, 1000 para PROD) permite ejecutar ambos ambientes simultáneamente sin conflictos, y las URLs son claramente distinguibles.

### La disciplina de DDL que te salva

La regla más crítica que establecí desde temprano: **todo cambio DDL se ejecuta en ambos schemas, por separado, de forma explícita.**

```sql
-- Se ejecuta primero en DEV_SCHEMA, se valida, luego:
ALTER TABLE orders ADD (discount_pct NUMBER(5,2) DEFAULT 0);

-- El mismo statement se ejecuta manualmente en PROD_SCHEMA
ALTER TABLE orders ADD (discount_pct NUMBER(5,2) DEFAULT 0);
```

Sin automatización — de forma intencional. Obligar la ejecución manual crea un punto de control: tienes que aplicar y verificar conscientemente cada cambio antes de que llegue a producción. Mantengo una tabla `DDL_CHANGELOG` en cada schema para registrar lo que se ha aplicado:

```sql
CREATE TABLE ddl_changelog (
    change_id   NUMBER GENERATED ALWAYS AS IDENTITY,
    change_date DATE DEFAULT SYSDATE,
    description VARCHAR2(500),
    applied_by  VARCHAR2(100) DEFAULT USER,
    CONSTRAINT pk_ddl_changelog PRIMARY KEY (change_id)
);
```

---

## Estructura de la Aplicación APEX

Para un SaaS multi-tenant, la estructura de la app importa más de lo que la mayoría de tutoriales cubren.

### Convención de numeración de páginas

Uso un esquema de numeración deliberado que escala con el proyecto:

- **Página 0** — Página global (header, footer, navegación)
- **Páginas 1–9** — Dashboard y resúmenes
- **Páginas 10–99** — Módulos funcionales (bloques de 10 por módulo)
- **Páginas 100–199** — Páginas administrativas
- **Páginas 9000–9998** — Modales y diálogos
- **Página 9999** — Login

Esto hace la app navegable sin un mapa mental y evita el caos de la numeración secuencial cuando el proyecto crece.

### Autenticación para SaaS Multi-Tenant

APEX 24.2 ofrece varios esquemas de autenticación. Para un SaaS con tenants aislados, construí un esquema de autenticación custom respaldado por una tabla `APP_USERS`:

```sql
CREATE TABLE app_users (
    user_id       NUMBER GENERATED ALWAYS AS IDENTITY,
    username      VARCHAR2(100) NOT NULL,
    password_hash VARCHAR2(256) NOT NULL,
    tenant_id     NUMBER NOT NULL,
    is_active     VARCHAR2(1) DEFAULT 'Y',
    last_login    TIMESTAMP,
    CONSTRAINT pk_app_users PRIMARY KEY (user_id),
    CONSTRAINT uq_username UNIQUE (username)
);
```

La función de autenticación establece un item de aplicación (`APP_TENANT_ID`) al hacer login exitoso, que cada query subsecuente usa para filtrar datos:

```sql
-- Toda consulta de datos sigue este patrón
SELECT * FROM orders
WHERE tenant_id = :APP_TENANT_ID;
```

### Navegación y Esquemas de Autorización

Una funcionalidad subutilizada en APEX: los **Authorization Schemes**. Creé esquemas basados en roles que controlan tanto la visibilidad de la navegación como el acceso a páginas:

- `IS_ADMIN` — Acceso total
- `IS_MANAGER` — Acceso a nivel de módulo
- `IS_OPERATOR` — Solo transacciones

Cada item del menú y cada página sensible verifica el esquema correspondiente. Esto centraliza la lógica de autorización en lugar de dispersarla entre condiciones de página y procesos.

---

## El Flujo de Deployment

Pasar cambios de DEV a PROD es donde la mayoría de desarrolladores APEX se queman. Mi flujo:

1. **Desarrollar y probar en DEV (App 100)** — Todo feature nuevo, cambio de UI y código PL/SQL entra aquí primero.
2. **Exportar la aplicación APEX** — APEX Builder → Export → Full Export como archivo SQL, commitado a control de versiones.
3. **Aplicar cambios DDL en PROD schema** — Manual, registrado en `DDL_CHANGELOG`.
4. **Importar en el workspace PROD (App 1000)** — Siempre con "Overwrite Existing Application". Nunca dejar que APEX asigne un nuevo ID automáticamente.
5. **Smoke test de flujos críticos** — Pruebo los 5–10 recorridos más importantes antes de notificar a los clientes.

---

## Lo que APEX 24.2 Cambió Específicamente

**Interactive Grid mejorado:** El IG de 24.2 maneja datasets grandes con mejor virtual scrolling. Para pantallas con cientos de filas de transacciones, esto fue una mejora significativa.

**Variables de tema unificadas:** Manejar una identidad visual consistente en una app multi-página es más fácil con el soporte expandido de CSS custom properties en Universal Theme 42. Sobrescribo exactamente 8 variables en el CSS global para lograr personalización de marca completa sin tocar estilos a nivel de componente.

**Rendimiento del Page Designer:** El árbol de componentes y el editor de propiedades en 24.2 son notablemente más rápidos en páginas complejas — una mejora de calidad de vida que se acumula en proyectos largos.

---

## Lecciones Difíciles

**Lección 1: Establece la disciplina de DDL desde el día uno.** No lo hice, y pasé una tarde dolorosa reconciliando por qué DEV tenía columnas que PROD no tenía. El patrón `DDL_CHANGELOG` vino directamente de ese dolor.

**Lección 2: Nombra todo con intención.** Los items, regiones y procesos de APEX se acumulan rápido. Una convención como `P{página}_{COMPONENTE}_{DESCRIPCION}` ahorra horas de confusión en la página 50 cuando ya no recuerdas para qué era `P50_X`.

**Lección 3: La Página Global (Página 0) es poderosa y peligrosa.** Todo lo que coloques en Página 0 se ejecuta en cada carga de página. Accidentalmente agregué un cómputo pesado ahí al principio y degradé el rendimiento de toda la app antes de encontrar la causa.

**Lección 4: Defiende contra state de sesión nulo.** ¿Qué pasa cuando la sesión de un usuario expira en medio de una transacción? ¿Qué pasa si `APP_TENANT_ID` es null? Construye validaciones defensivas en PL/SQL desde el inicio — no asumas que los items de sesión siempre están poblados.

---

## Conclusión

Oracle APEX 24.2 es una plataforma legítima para construir aplicaciones SaaS en producción, especialmente para software de negocio intensivo en datos dirigido a PYMEs. La ventaja en velocidad de desarrollo es real, la integración con Oracle Database no tiene comparación, y la complejidad operacional es manejable para un equipo pequeño.

Los patrones que compartí aquí — aislamiento por schema, changelogs de DDL explícitos, esquemas de autorización por rol y numeración de páginas deliberada — son lo que quisiera haber encontrado documentado junto cuando empecé.

Si estás construyendo algo similar o tienes preguntas sobre alguno de estos patrones, encuéntrame en los foros de Oracle APEX Community o en los comentarios.

---

*Jhosh Gomez Ledlaw es Oracle ACE Apprentice y desarrollador de software independiente construyendo aplicaciones SaaS sobre Oracle APEX.*
