# Arquitectura

Base: el diagrama de referencia del profesor (3 servidores, cada motor con su fragmentación y replicación, monitoreo centralizado), con dos mejoras justificadas abajo.

> Estado: **propuesta**. La forma de instalar (VM o Windows nativo) depende de la RAM de cada PC — ver [pendientes.md](pendientes.md).

## Distribución de servicios

| Servicio | PC1 | PC2 | PC3 |
|---|---|---|---|
| MySQL (Group Replication, single-primary) | Primario | Miembro | Miembro |
| PostgreSQL | — | Primario | Standby (streaming) |
| MongoDB shard (replica set) | Miembro | Miembro | Miembro |
| MongoDB config servers (replica set) | Miembro | Miembro | Miembro |
| mongos (router) | ✔ | ✔ | — |
| MySQL Router | ✔ | ✔ | — |
| API + simulador | Principal | Respaldo | — |
| Prometheus + Grafana | — | — | ✔ |
| Exporters | ✔ | ✔ | ✔ |

## Mejoras sobre el diagrama del profesor

1. **MySQL con 3 nodos en vez de 2.** Con Group Replication de 2 nodos, si cae uno el otro pierde la mayoría y se bloquea para escritura. Con 3, los dos que quedan eligen un nuevo primario solos. MySQL Router, junto a la API, la dirige siempre al primario vigente.
2. **Monitoreo en PC3 en vez de PC1.** PC3 solo aloja réplicas, y en las pruebas de fallas lo más probable es que se tumben los primarios (PC1 o PC2). Así Grafana sigue vivo para mostrar la caída y la recuperación.
3. **API en dos PCs.** El diagrama no indica dónde corre la API; si vive en una sola PC y esa PC cae, toda la aplicación cae aunque las bases sigan vivas.

## Puertos

| Servicio | Puerto |
|---|---|
| MySQL | 3306 |
| Group Replication (comunicación interna) | 33061 |
| MySQL Router (escritura / lectura) | 6446 / 6447 |
| PostgreSQL | 5432 |
| mongos | 27017 |
| Mongo shard | 27018 |
| Mongo config servers | 27019 |
| API | 8080 |
| Prometheus | 9090 |
| Grafana | 3000 |
| node_exporter / windows_exporter | 9100 / 9182 |
| mysqld_exporter | 9104 |
| postgres_exporter | 9187 |
| mongodb_exporter | 9216 |

## Red

- Router o switch propio, llevado el día de la demo (la red de la UAA suele aislar los equipos entre sí).
- IPs fijas, propuesta: `192.168.50.11` (PC1), `192.168.50.12` (PC2), `192.168.50.13` (PC3).
- Conexión por **cable Ethernet**, sobre todo si se usan máquinas virtuales.
- Puertos de la tabla abiertos en el firewall de cada equipo.

## Opciones de instalación

| Opción | Requisito | Valoración |
|---|---|---|
| **VM Ubuntu Server en cada PC + Docker** | 12–16 GB de RAM por PC, cable al router | Preferida si alcanza la RAM |
| Windows + bases nativas (monitoreo y API en Docker) | 8 GB de RAM por PC | Buena si alguna PC tiene poca RAM |
| Windows + todo en Docker | — | Descartada (ver decisiones) |

### Si se usan máquinas virtuales

- Red en **modo puente** (bridged), nunca NAT: la VM debe tener su propia IP en la LAN.
- Una VM por PC física; las tres no pueden vivir en la misma PC.
- Hipervisor: VirtualBox o VMware Workstation Pro (Windows 11 Home no trae Hyper-V).
- Si se clona la VM, hacerlo **antes** de instalar las bases y cambiar hostname y MAC.
- Tomar snapshots con todo funcionando antes de la demo.

### Si se instala nativo en Windows

- En lugar de `node_exporter` se usa `windows_exporter`.
- Scripts en PowerShell.
- Red marcada como **Privada** y reglas de firewall por puerto.
- Pausar Windows Update y desactivar la suspensión antes de la demo.
