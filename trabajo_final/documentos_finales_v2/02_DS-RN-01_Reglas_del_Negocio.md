# DeliverySuite

# Proyecto: DeliverySuite

# Reglas del Negocio

## Versión 1.0

Identificador del documento: DS-RN-01

---

# Historial de Revisiones

| Fecha | Versión | Descripción | Autor |
|---|---|---|---|
| 28/Jun/26 | 1.0 | Primera versión — reglas del ciclo de pedido, tarifa de envío y comisiones | Flores Hoyos, Mathias |
| 28/Jun/26 | 1.0 | Primera versión — reglas del ciclo de pedido, tarifa de envío y comisiones | Príncipe Caballa, Aida |
| 28/Jun/26 | 1.0 | Primera versión — reglas del ciclo de pedido, tarifa de envío y comisiones | Ochoa Torres, Yoant Alnor |

---

# Tabla de Contenidos

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

# Reglas del Negocio

Las reglas presentadas a continuación constituyen las políticas y restricciones que gobiernan el comportamiento del negocio con independencia de su implementación técnica particular. Se agrupan en tres grandes dominios: las reglas que gobiernan el ciclo de vida del pedido y la autoridad de cada actor sobre sus transiciones (RN-01 a RN-08), las reglas que determinan el cálculo de la tarifa de envío (RN-09 a RN-14) y las reglas que rigen la liquidación de comisiones entre la plataforma, las tiendas y los repartidores (RN-15 a RN-18).

## RN-01. Un pedido solo puede ser creado por un Cliente autenticado

Un pedido solo puede iniciarse cuando existe un Cliente registrado y autenticado en el sistema. Para que la creación sea válida, el pedido debe incluir, como mínimo, un ítem seleccionado del catálogo, una dirección de entrega válida provista de coordenadas geográficas, un método de pago definido y la referencia a una tienda afiliada que se encuentre activa. Una vez satisfechas estas condiciones, el pedido ingresa automáticamente al sistema en el estado PENDING.

---

## RN-02. El estado de un pedido sigue una secuencia estricta y unidireccional

El estado de un pedido evoluciona siguiendo estrictamente la secuencia:

```
PENDING → CONFIRMED → WAITING_DRIVER → DRIVER_ASSIGNED → ACCEPTED
       → AT_STORE → PICKED_UP → AT_CUSTOMER → DELIVERED
```

Ningún estado de esta secuencia puede omitirse ni revertirse. La única excepción de carácter operativo se presenta cuando el pedido se encuentra en el estado DRIVER_ASSIGNED y el candidato designado no acepta la asignación dentro del tiempo de espera establecido: en tal circunstancia, el Sistema de Asignación puede reiniciar el proceso de selección y asignar un nuevo candidato, manteniendo el pedido en el mismo estado DRIVER_ASSIGNED. Esta situación no constituye una reversión del estado, sino una reiteración del mismo estado con un candidato distinto, y resulta consistente con la descripción del actor Sistema de Asignación establecida en el documento DS-ACT-01.

El único estado terminal alternativo a DELIVERED es CANCELLED, al cual el pedido puede llegar desde el estado PENDING —por cancelación decidida por el Cliente— o desde cualquier estado posterior a CONFIRMED, en cuyo caso la cancelación requiere la intervención del Administrador o de Soporte. Un pedido que alcanza el estado DELIVERED o el estado CANCELLED se considera definitivo y no admite ninguna transición ulterior.

---

## RN-03. Solo la Tienda / Dispatcher puede confirmar o rechazar un pedido PENDING

La transición del estado PENDING al estado CONFIRMED es responsabilidad exclusiva de la Tienda / Dispatcher a la cual ha sido dirigido el pedido. Al ejecutar la confirmación, la Tienda / Dispatcher debe indicar obligatoriamente la sucursal desde la cual se realizará el despacho (`storeLocationId`) así como el tiempo estimado de preparación, expresado en minutos (`estimatedPrepTime`). En caso de que la Tienda / Dispatcher decida rechazar el pedido, este transiciona directamente al estado CANCELLED, registrándose el tipo de cancelación STORE_FAULT_BEFORE_DRIVER_PAID.

---

## RN-04. La cancelación por el Cliente y el rechazo por la Tienda son acciones distintas que producen el mismo estado terminal

Sobre un pedido que se encuentra en estado PENDING pueden ocurrir dos acciones diferentes que, si bien ambas producen el estado CANCELLED, se distinguen por el actor que las ejecuta y por el tipo de cancelación que registran. La primera de ellas es la cancelación por el Cliente, mediante la cual el propio Cliente decide retirar su pedido mientras este permanece en PENDING; en este caso, el tipo de cancelación aplicable es CLIENT_FAULT, o bien uno sin penalidad según la política vigente, y esta constituye la única circunstancia en la que el Cliente puede producir por sí mismo una transición hacia CANCELLED. La segunda acción es el rechazo por la Tienda / Dispatcher, definido en la regla RN-03, mediante el cual la tienda declina atender el pedido durante su revisión inicial; en este supuesto, el tipo de cancelación resultante es STORE_FAULT_BEFORE_DRIVER_PAID.

Una vez que el pedido ha avanzado más allá del estado PENDING —es decir, se encuentra en CONFIRMED o en un estado posterior— ninguno de estos dos actores conserva la facultad de cancelarlo por iniciativa propia. A partir de ese punto, la cancelación solo puede ser ejecutada por el Administrador o por Soporte.

---

## RN-05. La cancelación lleva un tipo que determina su tratamiento financiero

Toda cancelación registrada en el sistema debe llevar asociado un tipo (`cancellation_type`), del cual se derivan consecuencias financieras específicas, según se detalla a continuación:

| Tipo | Consecuencia |
|---|---|
| CLIENT_FAULT | Se aplica una penalidad económica configurable a la cuenta del Cliente; las comisiones de tienda y repartidor se registran con normalidad. El mecanismo de cálculo, cobro y liquidación de dicha penalidad está fuera del alcance de este documento. |
| STORE_FAULT_BEFORE_DRIVER_PAID | Anulación total sin registro de comisiones en el ledger. |
| STORE_FAULT_AFTER_DRIVER_PAID | El Repartidor conserva la tarifa de envío; la tienda no genera comisión. |
| DRIVER_FAULT | Las comisiones se registran con normalidad; el Repartidor pierde la entrega. |

---

## RN-06. Un Repartidor solo puede recibir pedidos si está online y no ha alcanzado su límite de órdenes simultáneas

Tanto el Sistema de Asignación, cuando el modo activo es AUTO_PROXIMITY, como el Administrador, cuando el modo activo es ADMIN_MANUAL, solo pueden asignar un pedido a un Repartidor que satisfaga simultáneamente tres condiciones: que se encuentre conectado (`is_online = true`), que no haya alcanzado el límite de pedidos simultáneos que le corresponde según su rango —es decir, que el número de pedidos activos que porta sea menor que su valor de `max_concurrent_orders`— y que su región geográfica registrada coincida con la región del pedido.

---

## RN-07. El límite de pedidos simultáneos de un Repartidor está determinado por su rango

El número máximo de órdenes que un Repartidor puede llevar de manera concurrente (`max_concurrent_orders`) se encuentra vinculado a su rango (`rank`) conforme a la siguiente tabla:

| Rango | Pedidos simultáneos máximos |
|---|---|
| BRONZE | 1 |
| SILVER | 2 |
| GOLD | 3 |
| DIAMOND | 4 |
| LEGEND | 5 |

---

## RN-08. El modo de asignación de repartidores es un parámetro global configurable

El sistema opera bajo uno de dos modos de asignación posibles, almacenado en la entidad `Setting`, la cual constituye un singleton de alcance global. En el modo AUTO_PROXIMITY, el Sistema de Asignación selecciona y asigna de manera automática al candidato más adecuado que se encuentre disponible en la región del pedido. En el modo ADMIN_MANUAL, en cambio, es el Administrador quien asigna manualmente al Repartidor mediante la acción `assign-driver`. Sobre este segundo modo existe un supuesto pendiente de confirmación: la documentación fuente no precisa si, en ADMIN_MANUAL, el pedido continúa transitando por los estados WAITING_DRIVER, DRIVER_ASSIGNED y ACCEPTED de la misma manera que en el modo automático, o si el Administrador cuenta con la posibilidad de saltar directamente a un estado posterior al momento de asignar. A efectos de este documento, se asume que los estados intermedios se respetan de la misma forma que en AUTO_PROXIMITY, quedando pendiente confirmar este punto con el equipo técnico antes de especificar en detalle los casos de uso de asignación manual.

En todo momento solo puede encontrarse activo un modo de asignación. El Administrador es el único actor facultado para modificar el modo activo, acción que ejecuta desde el panel web de administración.

---

## RN-09. La tarifa de envío se calcula en el momento de creación del pedido y queda fija

El cálculo de la tarifa de envío se ejecuta en el instante en que el sistema recibe la solicitud de creación del pedido (`POST /orders`). El valor resultante de dicho cálculo queda almacenado en el campo `delivery_fee` del pedido y no vuelve a recalcularse en ninguno de los estados posteriores del ciclo de vida, con independencia de que las reglas de tarifa o la configuración global experimenten modificaciones tras la creación del pedido.

---

## RN-10. El modo de cálculo de tarifa activo (ZONE o DISTANCE) es global y configurable

El sistema admite dos modalidades de cálculo de la tarifa de envío, controladas por el campo `fee_mode` de la entidad `Setting`. En la modalidad ZONE (`zone`), la tarifa base se determina según la zona geográfica —representada mediante un polígono en formato GeoJSON— a la que pertenece el punto de entrega. En la modalidad DISTANCE (`distance`), la tarifa se calcula de forma lineal en función de la distancia, expresada en kilómetros, entre la tienda y el punto de entrega. El modo que se encuentre activo en un momento dado se aplica de manera uniforme a todos los pedidos nuevos que ingresen al sistema.

---

## RN-11. En modo ZONE, la tarifa base se determina por la zona geográfica del punto de entrega

Cuando el campo `fee_mode` tiene el valor `zone`, el sistema evalúa el conjunto de zonas de entrega activas (`delivery_zones`) para determinar si el punto de entrega se encuentra dentro del polígono correspondiente a alguna de ellas. De verificarse esta condición, la tarifa se calcula conforme a la siguiente expresión:

```
tarifa = tarifa_base + (distancia_km × tarifa_km_extra)
tarifa = máx(tarifa_mínima, mín(tarifa, tarifa_máxima))
```

En caso de que el punto de entrega no pertenezca a ninguna zona configurada, el sistema aplica un valor de respaldo (fallback) equivalente a S/ 2.00 por kilómetro.

---

## RN-12. En modo DISTANCE, la tarifa se calcula linealmente por kilómetro

Cuando el campo `fee_mode` tiene el valor `distance`, la tarifa se obtiene multiplicando la distancia expresada en kilómetros por el precio por kilómetro configurado en la entidad `Setting`, aplicando a continuación los límites mínimo y máximo también configurados globalmente:

```
tarifa = distancia_km × precio_por_km
tarifa = máx(tarifa_mínima, mín(tarifa, tarifa_máxima))
```

---

## RN-13. Las reglas de tarifa de envío se aplican en orden de prioridad sobre la tarifa base

Una vez calculada la tarifa base —ya sea por zona o por distancia, según corresponda— el sistema aplica sobre ella las reglas de tarifa activas (`delivery_fee_rules`), procesándolas en orden ascendente de prioridad (`priority ASC`). Cada regla puede modificar la tarifa de acuerdo con su tipo, conforme al siguiente detalle:

| Tipo de regla | Efecto |
|---|---|
| SURCHARGE_FIXED | Suma un monto fijo: `tarifa += valor` |
| SURCHARGE_PERCENTAGE | Suma un porcentaje: `tarifa += tarifa × (valor / 100)` |
| MINIMUM_FEE | Establece un mínimo: `tarifa = máx(tarifa, valor)` |
| MULTIPLIER | Multiplica la tarifa: `tarifa = tarifa × valor` |
| BASE_FEE | Reemplaza la tarifa base: `tarifa = (tarifa − base_original) + valor` |

Cada regla puede, adicionalmente, restringirse a una ciudad determinada, a días de la semana específicos y a una ventana horaria particular, la cual admite el cruce de la medianoche.

---

## RN-14. La tarifa final se multiplica por el factor de la tienda y se redondea al medio sol superior

Una vez aplicadas todas las reglas de tarifa correspondientes, el sistema aplica el multiplicador propio de la tienda (`store.fee_multiplier`) sobre el valor obtenido. El resultado se redondea posteriormente al medio sol más cercano hacia arriba, conforme a la expresión:

```
tarifa_final = ⌈tarifa × 2⌉ / 2
```

A modo de ejemplo, una tarifa de S/ 7.30 se redondea a S/ 7.50.

---

## RN-15. La comisión de plataforma sobre la tienda depende del tipo de tienda

En el momento en que un pedido transiciona al estado DELIVERED, el sistema calcula y registra la comisión de plataforma correspondiente, la cual varía según el tipo de tienda (`store_type`) involucrada. Para las tiendas de tipo OFFICIAL_10, la comisión se calcula como el 10 % del subtotal, y adicionalmente se registra en el ledger un ingreso de tienda equivalente al subtotal (`store_revenue = subtotal`) únicamente cuando el método de pago empleado es CARD; en los pagos realizados en efectivo (CASH) no se genera esta entrada de ingreso. Para las tiendas de tipo OFFICIAL_5, se calcula primero una base equivalente al subtotal dividido entre 1.05, y la comisión corresponde al 10 % de dicha base, dado que el 5 % restante ya se encuentra incorporado en el precio cobrado al Cliente. Para las tiendas de tipo NON_OFFICIAL, la base se obtiene dividiendo el subtotal entre 1.10, y la comisión equivale a la diferencia entre el subtotal y dicha base, correspondiente al margen del 10 % que la plataforma retiene por estar incorporado en el precio final.

---

## RN-16. La tasa de comisión del Repartidor se resuelve por jerarquía de configuración

La tasa porcentual de comisión que se aplica a un Repartidor en un pedido determinado se resuelve siguiendo una jerarquía de configuración ordenada de mayor a menor especificidad. En primer lugar, se considera el campo `Driver.commission_rate`, correspondiente a la configuración individual del propio Repartidor. En su ausencia, se recurre a un registro de `CommissionConfig` cuya ciudad coincida con la del pedido. De no existir tampoco dicho registro, se aplica el `CommissionConfig` cuyo campo de ciudad sea nulo, el cual representa la configuración global por defecto, equivalente al 10 % para la tienda y al 5 % para el repartidor. Finalmente, si ninguna de las configuraciones anteriores estuviera disponible, se aplica el valor `Settings.driver_commission_percentage`, que constituye el valor de reserva absoluto, fijado en 5 %.

---

## RN-17. La comisión del Repartidor se calcula sobre la tarifa de envío del pedido entregado

La comisión que el Repartidor abona a la plataforma se calcula como un porcentaje de la tarifa de envío (`delivery_fee`) correspondiente al pedido, empleando la tasa resuelta conforme a la jerarquía establecida en la regla RN-16:

```
comisión_driver = delivery_fee × (commission_rate / 100)
```

Esta comisión queda registrada en el ledger financiero en el momento preciso en que el pedido alcanza el estado DELIVERED.

---

## RN-18. Los precios de los ítems se registran como snapshot inmutable al momento de crear el pedido

Al momento de crear un pedido, el sistema registra en la tabla `order_items` el nombre del producto (`product_name`), su precio unitario (`unit_price`), el subtotal correspondiente y los datos de la variante seleccionada (`variant_label`, `variant_price`) vigentes en ese instante. Este registro constituye un snapshot inmutable: cualquier modificación que se realice posteriormente sobre el catálogo de la tienda no afecta el valor que ya ha quedado registrado en los pedidos previamente creados.
