# DeliverySuite

# Proyecto: DeliverySuite

# Reglas del Negocio

## Versión 1.0

---

**Historial de Revisiones**

| Fecha | Versión | Descripción | Autor |
|---|---|---|---|
| 28/Jun/26 | 1.0 | Primera versión — reglas del ciclo de pedido, tarifa de envío y comisiones | Flores Hoyos, Mathias |
| 28/Jun/26 | 1.0 | Primera versión — reglas del ciclo de pedido, tarifa de envío y comisiones | Príncipe Caballa, Aida |
| 28/Jun/26 | 1.0 | Primera versión — reglas del ciclo de pedido, tarifa de envío y comisiones | Ochoa Torres, Yoant Alnor |

---

## Tabla de Contenidos

## RN-01. Un pedido solo puede ser creado por un Cliente autenticado
## RN-02. El estado de un pedido sigue una secuencia estricta y unidireccional
## RN-03. Solo la Tienda / Dispatcher puede confirmar o rechazar un pedido PENDING
## RN-04. Un pedido PENDING solo puede ser cancelado por el Cliente; estados posteriores requieren Administrador o Soporte
## RN-05. La cancelación lleva un tipo que determina su tratamiento financiero
## RN-06. Un Repartidor solo puede recibir pedidos si está online y no ha alcanzado su límite de órdenes simultáneas
## RN-07. El límite de pedidos simultáneos de un Repartidor está determinado por su rango
## RN-08. El modo de asignación de repartidores es un parámetro global configurable
## RN-09. La tarifa de envío se calcula en el momento de creación del pedido y queda fija
## RN-10. El modo de cálculo de tarifa activo (ZONE o DISTANCE) es global y configurable
## RN-11. En modo ZONE, la tarifa base se determina por la zona geográfica del punto de entrega
## RN-12. En modo DISTANCE, la tarifa se calcula linealmente por kilómetro
## RN-13. Las reglas de tarifa de envío se aplican en orden de prioridad sobre la tarifa base
## RN-14. La tarifa final se multiplica por el factor de la tienda y se redondea al medio sol superior
## RN-15. La comisión de plataforma sobre la tienda depende del tipo de tienda
## RN-16. La tasa de comisión del Repartidor se resuelve por jerarquía de configuración
## RN-17. La comisión del Repartidor se calcula sobre la tarifa de envío del pedido entregado
## RN-18. Los precios de los ítems se registran como snapshot inmutable al momento de crear el pedido

---

## Reglas del Negocio

### RN-01. Un pedido solo puede ser creado por un Cliente autenticado

Un pedido solo puede iniciarse cuando existe un Cliente registrado y autenticado en el sistema. El pedido debe incluir al menos un ítem, una dirección de entrega válida con coordenadas geográficas, un método de pago y la referencia a una tienda afiliada activa. Al crearse, el pedido entra automáticamente en estado PENDING.

---

### RN-02. El estado de un pedido sigue una secuencia estricta y unidireccional

Los estados de un pedido siguen la secuencia:

```
PENDING → CONFIRMED → WAITING_DRIVER → DRIVER_ASSIGNED → ACCEPTED
       → AT_STORE → PICKED_UP → AT_CUSTOMER → DELIVERED
```

Ningún estado puede saltearse ni revertirse, con la siguiente excepción operativa: cuando el pedido se encuentra en DRIVER\_ASSIGNED y el candidato designado no acepta la asignación dentro del tiempo de espera establecido, el Sistema de Asignación puede reiniciar el proceso de selección y asignar un nuevo candidato, manteniendo el pedido en el estado DRIVER\_ASSIGNED. Esta es la única situación en que el estado no avanza linealmente; no constituye una reversión sino una reiteración del mismo estado con un candidato distinto. Este comportamiento es consistente con la descripción del actor "Sistema de Asignación" en DS-ACT-01.

El único estado terminal alternativo es CANCELLED, al que puede llegar desde PENDING (por cancelación del Cliente) o desde cualquier estado posterior a CONFIRMED (por acción del Administrador o Soporte). Un pedido DELIVERED o CANCELLED es definitivo y no admite más transiciones.

---

### RN-03. Solo la Tienda / Dispatcher puede confirmar o rechazar un pedido PENDING

La transición de PENDING a CONFIRMED es responsabilidad exclusiva de la Tienda / Dispatcher asignada al pedido. Al confirmar, la Tienda / Dispatcher debe indicar la sucursal de despacho (`storeLocationId`) y el tiempo estimado de preparación en minutos (`estimatedPrepTime`). Si la Tienda / Dispatcher rechaza el pedido, este pasa directamente a CANCELLED con tipo de cancelación STORE\_FAULT\_BEFORE\_DRIVER\_PAID.

---

### RN-04. La cancelación por el Cliente y el rechazo por la Tienda son acciones distintas que producen el mismo estado terminal

Sobre un pedido en estado PENDING pueden ocurrir dos acciones diferentes que ambas producen CANCELLED, pero con actores y tipos distintos:

- **Cancelación por el Cliente:** el Cliente decide retirar su propio pedido mientras está en PENDING. El tipo de cancelación aplicable es CLIENT\_FAULT o uno sin penalidad según la política vigente. Esta es la única circunstancia en que el Cliente puede producir una transición a CANCELLED.
- **Rechazo por la Tienda / Dispatcher:** la Tienda / Dispatcher rechaza el pedido en estado PENDING durante su revisión (definido en RN-03). El tipo de cancelación resultante es STORE\_FAULT\_BEFORE\_DRIVER\_PAID.

Una vez que el pedido ha avanzado más allá de PENDING (estado CONFIRMED o posterior), ninguno de estos dos actores puede cancelar el pedido por sí mismo. En esos estados, la cancelación solo puede ser ejecutada por el Administrador o por Soporte.

---

### RN-05. La cancelación lleva un tipo que determina su tratamiento financiero

Toda cancelación debe registrar un tipo (`cancellation_type`) que define las consecuencias financieras:

| Tipo | Consecuencia |
|---|---|
| CLIENT\_FAULT | Se aplica una penalidad económica configurable a la cuenta del Cliente; las comisiones de tienda y repartidor se registran con normalidad. El mecanismo de cálculo, cobro y liquidación de dicha penalidad está fuera del alcance de este documento. |
| STORE\_FAULT\_BEFORE\_DRIVER\_PAID | Anulación total sin registro de comisiones en el ledger. |
| STORE\_FAULT\_AFTER\_DRIVER\_PAID | El Repartidor conserva la tarifa de envío; la tienda no genera comisión. |
| DRIVER\_FAULT | Las comisiones se registran con normalidad; el Repartidor pierde la entrega. |

---

### RN-06. Un Repartidor solo puede recibir pedidos si está online y no ha alcanzado su límite de órdenes simultáneas

El Sistema de Asignación (modo AUTO\_PROXIMITY) o el Administrador (modo ADMIN\_MANUAL) solo pueden asignar un pedido a un Repartidor que cumpla simultáneamente las siguientes condiciones: está online (`is_online = true`), no ha alcanzado su límite de pedidos simultáneos según rango (número de pedidos activos menor que `max_concurrent_orders`), y su región geográfica registrada coincide con la del pedido.

---

### RN-07. El límite de pedidos simultáneos de un Repartidor está determinado por su rango

El número máximo de órdenes que un Repartidor puede llevar al mismo tiempo (`max_concurrent_orders`) está vinculado a su rango (`rank`) según la siguiente tabla:

| Rango | Pedidos simultáneos máximos |
|---|---|
| BRONZE | 1 |
| SILVER | 2 |
| GOLD | 3 |
| DIAMOND | 4 |
| LEGEND | 5 |

---

### RN-08. El modo de asignación de repartidores es un parámetro global configurable

El sistema opera en uno de dos modos de asignación, almacenado en la entidad `Setting` (singleton global):

- **AUTO\_PROXIMITY:** el Sistema de Asignación selecciona y asigna automáticamente al candidato más adecuado disponible en la región del pedido.
- **ADMIN\_MANUAL:** el Administrador asigna manualmente al Repartidor mediante la acción `assign-driver`. **Supuesto a confirmar:** la documentación fuente no especifica si en modo ADMIN\_MANUAL el pedido transita igualmente por WAITING\_DRIVER → DRIVER\_ASSIGNED → ACCEPTED, o si el Administrador puede saltar directamente a un estado posterior al asignar. Se asume que los estados intermedios se respetan de la misma forma que en AUTO\_PROXIMITY; confirmar con el equipo técnico antes de especificar los casos de uso de asignación manual.

Solo puede estar activo un modo a la vez. El Administrador puede cambiar el modo activo desde el panel web de administración.

---

### RN-09. La tarifa de envío se calcula en el momento de creación del pedido y queda fija

El cálculo de la tarifa de envío se ejecuta al recibir la solicitud de creación del pedido (`POST /orders`). El valor resultante queda almacenado en el campo `delivery_fee` del pedido y no se recalcula en estados posteriores, independientemente de cambios en las reglas de tarifa o en la configuración global.

---

### RN-10. El modo de cálculo de tarifa activo (ZONE o DISTANCE) es global y configurable

El sistema soporta dos modos de cálculo de tarifa de envío, controlados por el campo `fee_mode` de la entidad `Setting`:

- **ZONE (`zone`):** la tarifa base se determina por la zona geográfica (polígono GeoJSON) a la que pertenece el punto de entrega.
- **DISTANCE (`distance`):** la tarifa se calcula de forma lineal en función de la distancia en kilómetros entre la tienda y el punto de entrega.

El modo activo aplica a todos los pedidos nuevos del sistema.

---

### RN-11. En modo ZONE, la tarifa base se determina por la zona geográfica del punto de entrega

Cuando `fee_mode = 'zone'`, el sistema evalúa las zonas de entrega activas (`delivery_zones`) y determina si el punto de entrega cae dentro del polígono de alguna zona. La tarifa se calcula así:

```
tarifa = tarifa_base + (distancia_km × tarifa_km_extra)
tarifa = máx(tarifa_mínima, mín(tarifa, tarifa_máxima))
```

Si el punto de entrega no pertenece a ninguna zona configurada, se aplica un fallback de S/ 2.00 por kilómetro.

---

### RN-12. En modo DISTANCE, la tarifa se calcula linealmente por kilómetro

Cuando `fee_mode = 'distance'`, la tarifa se obtiene multiplicando la distancia en kilómetros por el precio por kilómetro configurado en `Setting`, y luego aplicando un mínimo y un máximo también configurados:

```
tarifa = distancia_km × precio_por_km
tarifa = máx(tarifa_mínima, mín(tarifa, tarifa_máxima))
```

---

### RN-13. Las reglas de tarifa de envío se aplican en orden de prioridad sobre la tarifa base

Tras calcular la tarifa base (por zona o por distancia), el sistema aplica las reglas de tarifa activas (`delivery_fee_rules`) en orden ascendente de prioridad (`priority ASC`). Cada regla puede modificar la tarifa según su tipo:

| Tipo de regla | Efecto |
|---|---|
| SURCHARGE\_FIXED | Suma un monto fijo: `tarifa += valor` |
| SURCHARGE\_PERCENTAGE | Suma un porcentaje: `tarifa += tarifa × (valor / 100)` |
| MINIMUM\_FEE | Establece un mínimo: `tarifa = máx(tarifa, valor)` |
| MULTIPLIER | Multiplica la tarifa: `tarifa = tarifa × valor` |
| BASE\_FEE | Reemplaza la tarifa base: `tarifa = (tarifa − base_original) + valor` |

Una regla puede restringirse a una ciudad, a días de la semana específicos y a una ventana horaria (con soporte de cruce de medianoche).

---

### RN-14. La tarifa final se multiplica por el factor de la tienda y se redondea al medio sol superior

Después de aplicar todas las reglas de tarifa, se aplica el multiplicador propio de la tienda (`store.fee_multiplier`). El valor resultante se redondea al medio sol más cercano hacia arriba:

```
tarifa_final = ⌈tarifa × 2⌉ / 2
```

Ejemplo: S/ 7.30 → S/ 7.50.

---

### RN-15. La comisión de plataforma sobre la tienda depende del tipo de tienda

Al entregar un pedido (transición a DELIVERED), el sistema calcula y registra la comisión de plataforma según el tipo de tienda (`store_type`):

| Tipo de tienda | Cálculo de comisión de plataforma |
|---|---|
| OFFICIAL\_10 | `comisión = subtotal × 10%`; adicionalmente, `store_revenue = subtotal` se registra en el ledger únicamente cuando el método de pago es CARD. En pagos en efectivo (CASH) no se genera esa entrada de ingreso de tienda. |
| OFFICIAL\_5 | `base = subtotal / 1.05`; `comisión = base × 10%` (el 5% ya está incluido en el precio cobrado al cliente) |
| NON\_OFFICIAL | `base = subtotal / 1.10`; `comisión = subtotal − base` (la plataforma retiene el margen del 10% incorporado en el precio) |

---

### RN-16. La tasa de comisión del Repartidor se resuelve por jerarquía de configuración

La tasa porcentual de comisión aplicada a un Repartidor en un pedido se determina siguiendo esta jerarquía (de mayor a menor especificidad):

1. `Driver.commission_rate` — configuración individual del Repartidor.
2. `CommissionConfig` con `city` coincidente — configuración por ciudad.
3. `CommissionConfig` con `city = NULL` — configuración global por defecto (10% tienda / 5% driver).
4. `Settings.driver_commission_percentage` — valor de reserva absoluto (5%).

---

### RN-17. La comisión del Repartidor se calcula sobre la tarifa de envío del pedido entregado

La comisión que el Repartidor paga a la plataforma se calcula como un porcentaje de la tarifa de envío (`delivery_fee`) del pedido, usando la tasa resuelta según RN-16:

```
comisión_driver = delivery_fee × (commission_rate / 100)
```

Esta comisión se registra en el ledger financiero en el momento en que el pedido alcanza el estado DELIVERED.

---

### RN-18. Los precios de los ítems se registran como snapshot inmutable al momento de crear el pedido

Al crear un pedido, el sistema guarda en `order_items` el nombre del producto (`product_name`), el precio unitario (`unit_price`), el subtotal, y los datos de variante (`variant_label`, `variant_price`) vigentes en ese momento. Este snapshot es inmutable: modificaciones posteriores al catálogo de la tienda no afectan el valor registrado en pedidos ya creados.
