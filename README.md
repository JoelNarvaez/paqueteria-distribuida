# Paquetería Distribuida

Proyecto final de **Ingeniería de Datos** (UAA, 7.º semestre — Dr. en C. Sergio Galván Cruz). Entrega: diciembre 2026.

Sistema de paquetería y rastreo de envíos que usa **MySQL, PostgreSQL y MongoDB** de forma integrada, distribuido en **3 computadoras físicas** con fragmentación, replicación, recuperación ante fallas y monitoreo con Prometheus + Grafana.

## Documentación

| Documento | Contenido |
|---|---|
| [docs/proyecto.md](docs/proyecto.md) | Idea central, requisitos del profesor y proceso de negocio |
| [docs/modelo-datos.md](docs/modelo-datos.md) | Tablas y colecciones de cada gestor, fragmentación |
| [docs/arquitectura.md](docs/arquitectura.md) | Distribución de servicios por PC, puertos y red |
| [docs/decisiones.md](docs/decisiones.md) | Decisiones tomadas, alternativas y por qué |
| [docs/reparto.md](docs/reparto.md) | Qué hace Joel, qué hace Mariel y qué se hace en conjunto |
| [docs/pendientes.md](docs/pendientes.md) | Lo que falta definir |
| [docs/inventario.md](docs/inventario.md) | Equipos, IPs, puertos y versiones (entregable) |
| [docs/bitacora.md](docs/bitacora.md) | Problemas encontrados y soluciones (entregable) |
| `docs/instalacion/` | Procedimiento de instalación por servicio (entregable) |

## Estructura del repositorio

```
docs/            Documentación y entregable digital
despliegue/      Lo que se levanta en cada máquina (pc1, pc2, pc3)
config/          Configuración de cada motor y del monitoreo
scripts/         Inicializar replicación, sharding y failover
bd/              Esquemas de cada gestor y generador de datos de prueba
api/             Backend (coordina los tres gestores)
simulador/       Generador automático de transacciones
pruebas-fallas/  Scripts y checklist para las pruebas de fallas
```

## Reglas del repo

- `main` siempre funcionando; cada quien trabaja en su rama (`feature/...`) y se integra con Pull Request.
- Contraseñas y datos sensibles van en `.env` en la raíz (nunca se sube); en el repo solo `.env.example`. El `.env` debe ser idéntico en las 3 PCs y se comparte por fuera del repo.
- Todo el proyecto (código, tablas, archivos) en español.
