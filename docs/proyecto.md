# Proyecto: Paquetería Distribuida

## Idea central

Una empresa de paquetería con centros regionales necesita registrar envíos, cobrarlos, asignarlos a rutas de reparto y rastrearlos en tiempo real. El sistema reparte esa información entre tres gestores según la naturaleza de cada dato:

- **MySQL** — datos maestros que cambian poco: clientes, direcciones, sucursales, empleados, vehículos, tarifas y facturas.
- **PostgreSQL** — la operación transaccional, que no puede quedar a medias: envíos, paquetes, pagos y rutas.
- **MongoDB** — eventos de alto volumen y estructura variable: rastreo, evidencias de entrega, incidencias, notificaciones y bitácora de la API.

Las tres bases **no se conectan entre sí**: toda operación pasa por la API, que coordina los gestores y maneja errores y reintentos. Entre gestores solo se comparten identificadores (por ejemplo, `cliente_id` dentro de un envío).

El objetivo es demostrar que el sistema sigue operando, se monitorea y se recupera cuando se cae un servicio o un servidor, mientras procesa una carga alta de transacciones generadas por el simulador.

## Por qué paquetería

- Cada gestor tiene una razón natural de estar ahí, no forzada por el requisito.
- La fragmentación horizontal por región refleja cómo operan las paqueterías reales (centros regionales).
- El rastreo genera miles de eventos por minuto, así que la carga del simulador es realista y justifica el sharding.
- Es un caso cercano a ingeniería de datos real (flujos de eventos, métricas operativas por zona).

Detalle de las alternativas evaluadas en [decisiones.md](decisiones.md).

## Requisitos del profesor (resumen)

| Rubro | Qué se pide | Valor |
|---|---|---|
| MySQL | 5+ tablas relacionadas, 50,000+ registros, fragmentación **vertical**, replicación | 5 % |
| PostgreSQL | 5+ tablas relacionadas, 50,000+ registros, fragmentación **horizontal** con criterio justificable, Streaming Replication | 5 % |
| MongoDB | 5+ colecciones, Replica Set de 3+ miembros en las PCs, sharding con mongos y config servers | 5 % |
| MongoDB | 50,000+ documentos | 3 % |
| Monitoreo | Prometheus + Grafana centralizado | 3 % |
| Entregable digital | Inventario, diagrama de red, instalación y configuración, scripts, bitácora | 2 % |

Además:

- Backend o API que interactúe con los tres gestores.
- Al menos un proceso de negocio completo que involucre **todas** las tablas y colecciones.
- Simulador de transacciones automático (decenas, cientos o miles de operaciones).
- Al menos 3 computadoras físicas en red local, con distribución real de las bases y su replicación.
- Pruebas controladas de fallas durante la evaluación.
- Demostración presencial y defensa **individual** (los dos integrantes deben dominar todo el proyecto).
- Se permiten mejoras a la arquitectura del profesor si se justifican y no eliminan requisitos mínimos.

## Proceso de negocio

| Paso | Qué pasa | Gestores |
|---|---|---|
| 1. Cotizar | Se leen cliente, direcciones y tarifa aplicable | MySQL: `clientes_contacto`, `direcciones`, `tarifas` |
| 2. Crear envío | Se registran envío y paquetes en una sola transacción; evento "recolectado" | PostgreSQL: `envios`, `paquetes` · MongoDB: `eventos_rastreo` |
| 3. Pagar y facturar | Se registra el pago, se genera la factura con los datos fiscales y se notifica al cliente | PostgreSQL: `pagos` · MySQL: `clientes_fiscal`, `facturas` · MongoDB: `notificaciones` |
| 4. Asignar a ruta | Se arma la ruta con vehículo y repartidor de la sucursal; evento "en_reparto" | PostgreSQL: `rutas`, `asignaciones_ruta` · MySQL: `sucursales`, `vehiculos`, `empleados` |
| 5. Rastreo | Eventos de tránsito conforme avanza el paquete | MongoDB: `eventos_rastreo` |
| 6. Entregar | Se actualiza el estado, se guarda la evidencia y se notifica; si falla, se abre una incidencia | PostgreSQL: `envios` · MongoDB: `entregas`, `notificaciones`, `incidencias` |
| Todos | Cada operación de la API queda registrada | MongoDB: `bitacora_api` |

**Punto crítico (paso 3):** el pago vive en PostgreSQL y la factura en MySQL. Si MySQL cae a la mitad, la API debe reintentar la factura o revertir el pago para no dejar datos inconsistentes. Es el "maneja errores / reintentos" que pide el diagrama del profesor.
