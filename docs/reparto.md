# Reparto del trabajo

Equipo: **Joel** y **Mariel**.

> Estado: **propuesta**, falta confirmarla con Mariel.

## Criterio

Cada quien es dueño de una parte completa de punta a punta (instalación, replicación, fragmentación, esquema, datos de prueba y documentación de sus servicios), para no depender del otro en el día a día. Lo que involucra las 3 PCs a la vez se hace en conjunto.

La defensa es **individual**: cada uno tiene que poder explicar la parte del otro. Por eso todo entra por Pull Request revisado por el otro integrante.

## Joel — MySQL, PostgreSQL y API

| Área | Tareas |
|---|---|
| MySQL | Group Replication de 3 nodos, MySQL Router, fragmentación vertical de `clientes`, esquema de 8 tablas, datos de prueba (50k+) |
| PostgreSQL | Streaming Replication (primario PC2 → standby PC3), procedimiento de failover, particionamiento por región de `envios`, esquema de 5 tablas, datos de prueba (50k+) |
| API | Backend en Node.js con el proceso de negocio completo, conexión a los tres gestores, manejo de errores y reintentos entre gestores (pago/factura) |
| Documentación | `docs/instalacion/` de MySQL, PostgreSQL y API |

## Mariel — MongoDB, monitoreo y simulador

| Área | Tareas |
|---|---|
| MongoDB | Replica set del shard, replica set de config servers, routers mongos, sharding de `eventos_rastreo`, 5 colecciones, datos de prueba (50k+) |
| Monitoreo | Prometheus, Grafana y exporters en las 3 PCs; dashboards de CPU/RAM/disco, latencia, transacciones por segundo y estado de réplicas |
| Simulador | Generador automático de transacciones que llama a la API (decenas, cientos y miles de operaciones) |
| Documentación | `docs/instalacion/` de MongoDB, monitoreo y simulador |

## En conjunto

| Área | Tareas |
|---|---|
| Infraestructura | Preparar las 3 PCs (VMs o Windows), red, IPs fijas y firewall |
| Integración | Conectar la API con MongoDB y con el simulador; prueba de punta a punta del proceso de negocio |
| Pruebas de fallas | Definir la lista de fallas, ensayarlas y documentar cómo se recupera cada servicio |
| Entregable | Inventario, diagrama de red, bitácora de problemas |
| Defensa | Sesiones donde cada uno le explica su parte al otro antes de la demo |

## Orden de trabajo sugerido

1. **Infraestructura (juntos)** — las 3 PCs comunicándose por red.
2. **Bases en paralelo** — Joel con MySQL y PostgreSQL, Mariel con MongoDB y el monitoreo básico.
3. **Esquemas y datos de prueba** — cada uno los de sus gestores.
4. **API y simulador** — Joel la API, Mariel el simulador contra la API.
5. **Dashboards y pruebas de fallas** — juntos.
6. **Documentación final y ensayo de la demo** — juntos.

Fecha de entrega: diciembre 2026, día exacto [completar].
