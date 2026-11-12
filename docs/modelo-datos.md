# Modelo de datos

Regla general: **las bases no se referencian entre sí**. Una llave foránea solo existe dentro del mismo gestor; entre gestores se guarda el ID y la API resuelve la relación.

Los volúmenes son aproximados y se ajustarán al construir el generador de datos (`bd/generador/`).

## MySQL — datos maestros + fragmentación vertical

| Tabla | Campos principales | Registros aprox. |
|---|---|---|
| `clientes_contacto` *(fragmento 1)* | cliente_id, nombre, telefono, email, fecha_registro | 15,000 |
| `clientes_fiscal` *(fragmento 2)* | cliente_id, rfc, razon_social, regimen_fiscal, cp_fiscal, uso_cfdi | 15,000 |
| `direcciones` | direccion_id, cliente_id, calle, colonia, ciudad, estado, cp, region | 20,000 |
| `sucursales` | sucursal_id, nombre, ciudad, region | 30 |
| `empleados` | empleado_id, sucursal_id, nombre, puesto | 500 |
| `vehiculos` | vehiculo_id, sucursal_id, placas, tipo, capacidad_kg | 300 |
| `tarifas` | tarifa_id, tipo_servicio, peso_min, peso_max, precio_base, precio_kg | 20 |
| `facturas` | factura_id, envio_id, cliente_id, subtotal, iva, total, fecha | 20,000 |
| **Total** | | **~70,000** |

### Fragmentación vertical

La entidad `clientes` se divide en dos fragmentos unidos por `cliente_id`, según la **frecuencia de acceso** de sus atributos:

- `clientes_contacto` — se consulta en cada envío (nombre, teléfono, email).
- `clientes_fiscal` — solo se consulta al facturar (RFC, razón social, régimen fiscal).

Así las consultas frecuentes leen filas más pequeñas y los datos fiscales quedan aislados. La tabla `facturas` obliga a usar el fragmento 2 dentro del proceso de negocio.

## PostgreSQL — operación transaccional + fragmentación horizontal

| Tabla | Campos principales | Registros aprox. |
|---|---|---|
| `envios` *(particionada)* | envio_id, guia, cliente_id, dir_origen_id, dir_destino_id, sucursal_origen_id, region, tipo_servicio, estado, costo, fecha_creacion, fecha_entrega_estimada | 20,000 |
| `paquetes` | paquete_id, envio_id, peso_kg, largo, ancho, alto, descripcion, valor_declarado | 25,000 |
| `pagos` | pago_id, envio_id, monto, metodo, estado, fecha | 20,000 |
| `rutas` | ruta_id, sucursal_id, vehiculo_id, repartidor_id, fecha, estado | 2,000 |
| `asignaciones_ruta` | ruta_id, envio_id, orden_entrega, estado | 20,000 |
| **Total** | | **~87,000** |

### Fragmentación horizontal

`envios` se fragmenta **por región**, con particionamiento nativo (`PARTITION BY LIST (region)`):

- `envios_norte`
- `envios_bajio`
- `envios_centro`
- `envios_sur`

Criterio: cada centro regional opera y consulta casi exclusivamente sus propios envíos.

**Detalle técnico:** en una tabla particionada la llave primaria debe incluir la columna de partición, así que la PK de `envios` será `(envio_id, region)` y las tablas que la referencian necesitan también `region`.

## MongoDB — eventos y datos de estructura variable

| Colección | Qué guarda | Documentos aprox. |
|---|---|---|
| `eventos_rastreo` **(shardeada)** | envio_id, tipo (recolectado, en_bodega, en_transito, en_reparto, intento_fallido, entregado), fecha, sucursal, ubicacion, `detalle` variable según el tipo | 130,000 |
| `entregas` | Evidencia: quién recibió, firma, foto, ubicación GPS | 18,000 |
| `incidencias` | Daño, extravío, dirección incorrecta; arreglo embebido `seguimiento[]` | 2,000 |
| `notificaciones` | cliente_id, envio_id, canal (sms/email/push), mensaje, leida | 40,000 |
| `bitacora_api` | Cada operación de la API: servicio, duracion_ms, resultado, error | 60,000+ |
| **Total** | | **~250,000** |

### Por qué documental

Cada tipo de evento trae campos distintos (un escaneo en bodega no se parece a un intento de entrega fallido con foto y motivo), los eventos solo se agregan y nunca se editan, y su volumen crece con cada paquete.

### Sharding

- Colección shardeada: `eventos_rastreo`.
- Shard key: `envio_id` **hasheado** — reparte los eventos de forma pareja y el historial de un envío se consulta de un solo shard.
- `bitacora_api` sirve como evidencia en las pruebas de fallas: registra qué falló y cuándo se recuperó.
