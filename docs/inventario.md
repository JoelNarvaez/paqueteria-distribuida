# Inventario

Entregable: equipos, direcciones, puertos y versiones. Se llena con los datos reales al instalar; lo marcado `[completar]` aún no existe.

## Equipos

| Equipo | Responsable | Sistema operativo | RAM | CPU | IP |
|---|---|---|---|---|---|
| PC1 | [completar] | [completar] | [completar] | [completar] | [completar] (propuesta 192.168.50.11) |
| PC2 | [completar] | [completar] | [completar] | [completar] | [completar] (propuesta 192.168.50.12) |
| PC3 | [completar] | [completar] | [completar] | [completar] | [completar] (propuesta 192.168.50.13) |

## Servicios

| Servicio | Equipo | Puerto | Versión | Rol |
|---|---|---|---|---|
| MySQL | PC1 | 3306 | [completar] | Primario (Group Replication) |
| MySQL | PC2 | 3306 | [completar] | Miembro |
| MySQL | PC3 | 3306 | [completar] | Miembro |
| MySQL Router | PC1, PC2 | 6446 / 6447 | [completar] | Enrutamiento a primario / lectura |
| PostgreSQL | PC2 | 5432 | [completar] | Primario |
| PostgreSQL | PC3 | 5432 | [completar] | Standby |
| Mongo shard | PC1, PC2, PC3 | 27018 | [completar] | Replica set del shard |
| Mongo config | PC1, PC2, PC3 | 27019 | [completar] | Replica set de configuración |
| mongos | PC1, PC2 | 27017 | [completar] | Router |
| API | PC1, PC2 | 8080 | [completar] | Principal / respaldo |
| Prometheus | PC3 | 9090 | [completar] | Recolección de métricas |
| Grafana | PC3 | 3000 | [completar] | Dashboards |
| Exporters | PC1, PC2, PC3 | 9100–9216 | [completar] | Métricas de equipo y motores |
