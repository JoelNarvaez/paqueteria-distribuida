# Decisiones

Registro de decisiones del proyecto, con las alternativas evaluadas y por qué se descartaron.

**Estado:** ✅ decidida · 🟡 propuesta, falta confirmar

---

## 2026-11-11 — Temática: paquetería y rastreo de envíos ✅

**Alternativas evaluadas:**

- *Marketplace (flujo de compra)* — es la propuesta de ejemplo del profesor; se pidió una temática propia.
- *Red de clínicas / citas médicas* — buena justificación de la fragmentación vertical por privacidad, pero un sistema real guardaría el expediente en relacional o en estándares médicos (MongoDB se siente forzado), y miles de citas por segundo no es una carga realista para el simulador.
- *Transporte de autobuses* — el GPS justifica muy bien MongoDB, pero el núcleo (corridas y asientos) se parece a un sistema de reservaciones y aporta menos.

**Por qué paquetería:** cada gestor se justifica por la naturaleza del dato, la fragmentación horizontal por región refleja la operación real, el volumen de eventos hace realista el simulador y es el caso más cercano a ingeniería de datos real.

## 2026-11-11 — Seguir la arquitectura del profesor con mejoras puntuales ✅

Es contra lo que se evalúa. Solo se cambia lo que tenga una razón técnica defendible (ver [arquitectura.md](arquitectura.md)).

## 2026-11-11 — Las bases no se conectan entre sí ✅

Lo indica el diagrama del profesor: toda operación pasa por la API. Entre gestores solo se guardan IDs; las llaves foráneas existen únicamente dentro de cada gestor.

## 2026-11-11 — Fragmentación vertical de `clientes` por frecuencia de acceso (MySQL) ✅

Fragmentos `clientes_contacto` (se lee en cada envío) y `clientes_fiscal` (solo al facturar). **Alternativa:** fragmentar por privacidad de datos; se prefirió la frecuencia de acceso por ser la justificación clásica de la fragmentación vertical.

## 2026-11-11 — Fragmentación horizontal de `envios` por región (PostgreSQL) ✅

Cuatro fragmentos: norte, bajío, centro y sur, con particionamiento nativo `LIST`. Refleja los centros regionales de una paquetería. **Pendiente:** qué estados van en cada región.

## 2026-11-11 — MongoDB con 1 shard y `envio_id` hasheado como shard key ✅

Un shard (replica set en las 3 PCs) más config servers y mongos es lo que pide el profesor. **Alternativa:** 2 shards — queda como extra si sobra tiempo, porque triplica los procesos de Mongo por PC.

## 2026-11-11 — Pago (PostgreSQL) y factura (MySQL) coordinados por la API ✅

Si un gestor cae a la mitad, la API reintenta o revierte el pago. Se documenta como el manejo de errores entre gestores.

## 2026-11-11 — MySQL Group Replication con 3 nodos 🟡

**Alternativas:** 2 nodos como en el diagrama (si cae uno, el otro pierde la mayoría y se bloquea), o replicación asíncrona con promoción manual (más simple, pero no se recupera sola). Con 3 nodos el failover es automático y MySQL Router dirige a la API al primario.

## 2026-11-11 — Monitoreo en PC3 🟡

**Alternativa:** PC1, como en el diagrama. Se descartó porque si tumban PC1 en las pruebas se pierde justo el monitoreo que debe mostrar la falla; PC3 solo tiene réplicas.

## 2026-11-11 — API en PC1 con respaldo en PC2 🟡

El diagrama no indica dónde corre. En una sola PC sería un punto único de falla.

## 2026-11-11 — Red propia con IPs fijas 🟡

La red de la universidad suele aislar equipos entre sí. Se lleva un router o switch propio a la demo.

## 2026-11-11 — Forma de instalación: VM Ubuntu o Windows nativo 🟡

- *Windows + todo en Docker* — **descartada.** Los contenedores ven una IP interna de Docker, no la IP real de la PC. Para MongoDB y PostgreSQL tiene arreglo, pero Group Replication probablemente valida que la dirección configurada sea local y no arrancaría (sin verificar).
- *VM Ubuntu Server por PC + Docker* — preferida: red en modo puente, snapshots, máquinas idénticas, scripts en bash. Necesita 12–16 GB de RAM por PC.
- *Windows + bases nativas* — respaldo si la RAM no alcanza para VMs.

Se decide al conocer la RAM de cada PC.

## 2026-11-11 — Backend en Node.js 🟡

Es el stack de Joel. Falta confirmarlo con el compañero.

## 2026-11-11 — Repositorio y forma de trabajo ✅

- Un repositorio con documentación, configuración, scripts y código; cada PC hace `git pull` y queda igual.
- `main` siempre funcional, ramas `feature/...` y Pull Requests revisados por el otro integrante (también prepara la defensa individual).
- Contraseñas en `.env` local, nunca en el repo.
- Todo en español.

## 2026-11-11 — Reparto: Joel (MySQL, PostgreSQL, API) y Mariel (MongoDB, monitoreo, simulador) 🟡

Cada quien es dueño de partes completas de punta a punta; infraestructura, integración, pruebas de fallas y entregable se hacen en conjunto. **Alternativa:** repartir por capas (uno toda la infraestructura, otro todo el código); se descartó porque deja a uno sin dominar las bases para la defensa individual. Detalle en [reparto.md](reparto.md).
