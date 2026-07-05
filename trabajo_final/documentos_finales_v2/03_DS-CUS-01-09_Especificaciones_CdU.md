# DeliverySuite

# Proyecto: DeliverySuite

# Especificaciones de Caso de Uso del Sistema

## Versión 1.0

Identificador del documento: DS-CUS-01-09

---

# Historial de Versiones

| Fecha | Versión | Descripción | Autor |
|---|---|---|---|
| 29/Jun/26 | 1.0 | Primera versión — CUS-01 a CUS-09, ciclo completo de pedido | Flores Hoyos, Mathias |
| 29/Jun/26 | 1.0 | Primera versión — CUS-01 a CUS-09, ciclo completo de pedido | Príncipe Caballa, Aida |
| 29/Jun/26 | 1.0 | Primera versión — CUS-01 a CUS-09, ciclo completo de pedido | Ochoa Torres, Yoant Alnor |

---

# Tabla de Contenidos

## CUS-01. Crear Pedido
## CUS-02. Confirmar Pedido
## CUS-03. Rechazar Pedido
## CUS-04. Asignar Repartidor Automáticamente
## CUS-05. Asignar Repartidor Manualmente
## CUS-06. Ejecutar Entrega
## CUS-07. Cancelar Pedido
## CUS-08. Consultar Estado / Seguimiento del Pedido
## CUS-09. Calificar Pedido

El presente documento reúne las especificaciones de los nueve casos de uso del sistema que conforman el ciclo completo de un pedido dentro de DeliverySuite, desde su creación por parte del Cliente hasta su entrega, cancelación o calificación. Cada especificación describe los actores involucrados, el propósito perseguido, el flujo de eventos —organizado en flujo básico, subflujos y flujos alternos— así como las precondiciones, poscondiciones y requerimientos especiales que resultan aplicables.

---

---

# Especificación del Caso de Uso del Sistema: CUS-01. Crear Pedido

---

## 1. Actores del Sistema

### 1.1 Cliente

Actor primario. Es quien inicia el caso de uso al seleccionar una tienda y los productos con los que desea conformar su pedido.

---

## 2. Propósito

Permitir al Cliente crear un nuevo pedido seleccionando productos de una tienda afiliada activa, especificando la dirección de entrega y el método de pago, de modo que el sistema registre el pedido y notifique a la Tienda / Dispatcher correspondiente.

---

## 3. Breve Descripción

El Cliente selecciona una tienda y uno o más productos de su catálogo, indica la dirección de entrega mediante coordenadas geográficas y elige un método de pago. A partir de esta información, el sistema calcula la tarifa de envío, valida los datos ingresados y registra el pedido en estado PENDING, conservando un snapshot inmutable de los precios de cada ítem seleccionado. Al concluir estas acciones, el sistema notifica a la Tienda / Dispatcher para que dé inicio a su proceso de revisión. El caso de uso concluye en el momento en que el pedido queda registrado y la notificación correspondiente ha sido enviada.

---

## 4. Flujo de Eventos

### 4.1 Flujo Básico

4.1.1 El Cliente selecciona una tienda afiliada que se encuentre activa y visible en la plataforma.

4.1.2 El Cliente selecciona uno o más productos del catálogo de la tienda, indicando para cada uno la cantidad deseada y, si corresponde, la variante elegida.

4.1.3 El Cliente especifica la dirección de entrega, incluyendo sus coordenadas geográficas.

4.1.4 El sistema ejecuta el cálculo de la tarifa de envío, descrito en el subflujo SF-01.

4.1.5 El Cliente selecciona el método de pago, el cual puede ser efectivo (CASH), tarjeta (CARD) o billetera (WALLET).

4.1.6 El Cliente revisa el resumen del pedido —compuesto por los ítems seleccionados, el subtotal, la tarifa de envío y el total— y procede a confirmarlo.

4.1.7 El sistema valida que el pedido cumpla con las condiciones mínimas establecidas en la regla RN-01: la presencia de al menos un ítem, una dirección con coordenadas válidas, una tienda activa y un Cliente autenticado.

4.1.8 El sistema registra el pedido en estado PENDING, conservando el snapshot inmutable del nombre, precio unitario, subtotal y variante de cada ítem, conforme a lo establecido en la regla RN-18.

4.1.9 El sistema envía una notificación a la Tienda / Dispatcher informando sobre la nueva orden recibida, empleando para ello FCM y, cuando se encuentre configurado, Telegram.

4.1.10 El sistema retorna al Cliente la confirmación del pedido creado, incluyendo su identificador y su estado PENDING.

### 4.2 Subflujos

**SF-01: Cálculo de tarifa de envío**

4.2.1 El sistema calcula la distancia, expresada en kilómetros, entre las coordenadas de la tienda —o de la sucursal, si esta ya se encuentra definida— y el punto de entrega indicado por el Cliente, empleando la fórmula de Haversine.

4.2.2 Si el modo de cálculo activo es ZONE (`fee_mode = 'zone'`), el sistema evalúa cada zona de entrega activa para determinar si el punto de entrega se encuentra dentro de su polígono GeoJSON. Al encontrar una zona coincidente, calcula la tarifa mediante la expresión `tarifa = tarifa_base + (distancia_km × tarifa_km_extra)` y aplica el rango `[tarifa_mínima, tarifa_máxima]` propio de esa zona, conforme a la regla RN-11. Si ninguna zona resulta coincidente, el sistema aplica el valor de respaldo de S/ 2.00 por kilómetro.

4.2.3 Si el modo de cálculo activo es DISTANCE (`fee_mode = 'distance'`), el sistema calcula la tarifa mediante la expresión `tarifa = distancia_km × precio_por_km` y aplica el rango `[tarifa_mínima, tarifa_máxima]` configurado de manera global, conforme a la regla RN-12.

4.2.4 El sistema aplica las reglas de tarifa activas (`delivery_fee_rules`) en orden ascendente de prioridad, evaluando en cada caso si la regla resulta aplicable según la ciudad, el día de la semana y la ventana horaria vigente, conforme a la regla RN-13.

4.2.5 El sistema multiplica la tarifa resultante por el factor propio de la tienda (`fee_multiplier`) y redondea el valor obtenido al medio sol más cercano hacia arriba, conforme a la regla RN-14.

4.2.6 El valor final queda fijado en el campo `delivery_fee` del pedido y no vuelve a recalcularse en los estados posteriores, conforme a la regla RN-09.

### 4.3 Flujos Alternos

**FA-01: La tienda está cerrada o inactiva**

En el paso 4.1.1, si la tienda seleccionada presenta el campo `is_open` en falso o el campo `is_hidden` en verdadero, el sistema informa al Cliente que la tienda no se encuentra disponible en ese momento y no le permite continuar con la creación del pedido.

**FA-02: El carrito no contiene ítems**

En el paso 4.1.6, si el Cliente intenta confirmar el pedido sin haber seleccionado ningún producto, el sistema rechaza la solicitud e informa que el pedido debe contener, como mínimo, un ítem.

**FA-03: Dirección de entrega sin coordenadas válidas**

En el paso 4.1.7, si la dirección de entrega no cuenta con coordenadas geográficas válidas, el sistema rechaza la solicitud e indica al Cliente que debe corregir la dirección proporcionada.

---

## 5. Precondiciones

### 5.1 Cliente autenticado

El Cliente debe encontrarse registrado y autenticado en el sistema antes de dar inicio al caso de uso.

### 5.2 Tienda activa

La tienda seleccionada debe existir, encontrarse activa (`is_open = true`) y no encontrarse oculta (`is_hidden = false`).

---

## 6. Poscondiciones

### 6.1 Pedido registrado en PENDING

El pedido queda persistido en la base de datos con estado PENDING, incluyendo el snapshot de ítems, la tarifa de envío calculada y la marca de tiempo `created_at`.

### 6.2 Snapshot de precios inmutable

Los precios, nombres y variantes de los ítems quedan registrados en `order_items` y no se ven afectados por cambios posteriores en el catálogo, conforme a la regla RN-18.

### 6.3 Tienda / Dispatcher notificada

La Tienda / Dispatcher ha recibido la notificación correspondiente al nuevo pedido, lo que le permite dar inicio a su revisión.

---

## 7. Puntos de Extensión

Ninguno en este caso de uso.

---

## 8. Requerimientos Especiales

8.1 El cálculo de la tarifa de envío debe completarse dentro del tiempo de respuesta de la solicitud de creación del pedido, sin generar retrasos perceptibles para el Cliente.

8.2 La notificación dirigida a la Tienda / Dispatcher debe enviarse de forma inmediata una vez creado el pedido.

---

## 9. Información Adicional

El cálculo de la tarifa de envío se rige por las Reglas del Negocio RN-09 a RN-14. El modo de cálculo activo, sea este ZONE o DISTANCE, constituye un parámetro global configurable exclusivamente por el Administrador, conforme a la regla RN-10.

---

---

# Especificación del Caso de Uso del Sistema: CUS-02. Confirmar Pedido

---

## 1. Actores del Sistema

### 1.1 Tienda / Dispatcher

Actor primario. Recibe la notificación del pedido y ejecuta la acción de confirmación.

---

## 2. Propósito

Permitir a la Tienda / Dispatcher revisar un pedido recibido y aceptarlo para su preparación, indicando la sucursal de despacho y el tiempo estimado de preparación, de modo que el sistema dé inicio a la búsqueda de un repartidor.

---

## 3. Breve Descripción

La Tienda / Dispatcher recibe la notificación de un nuevo pedido en estado PENDING, revisa sus detalles y decide confirmarlo. Al hacerlo, selecciona la sucursal desde la cual se realizará el despacho y estima el tiempo de preparación en minutos. El sistema transiciona el pedido al estado CONFIRMED, notifica al Cliente y activa de manera automática la búsqueda de un repartidor, llevando el pedido al estado WAITING_DRIVER. El caso de uso concluye cuando el pedido se encuentra en estado CONFIRMED y el proceso de asignación ha sido iniciado.

---

## 4. Flujo de Eventos

### 4.1 Flujo Básico

4.1.1 La Tienda / Dispatcher recibe la notificación de un nuevo pedido, a través de FCM o de Telegram, y accede a sus detalles desde la aplicación.

4.1.2 La Tienda / Dispatcher revisa los ítems del pedido, la dirección de entrega y las instrucciones proporcionadas por el Cliente.

4.1.3 La Tienda / Dispatcher selecciona la sucursal de despacho (`storeLocationId`) entre las sucursales activas de su tienda.

4.1.4 La Tienda / Dispatcher ingresa el tiempo estimado de preparación, expresado en minutos (`estimatedPrepTime`).

4.1.5 La Tienda / Dispatcher confirma el pedido.

4.1.6 El sistema valida que el pedido se encuentre en estado PENDING, conforme a la regla RN-03.

4.1.7 El sistema transiciona el pedido al estado CONFIRMED y registra la marca de tiempo `confirmed_at`.

4.1.8 El sistema notifica al Cliente, a través de FCM, que su pedido ha sido confirmado.

4.1.9 El sistema transiciona automáticamente el pedido al estado WAITING_DRIVER e inicia el proceso de asignación de repartidor según el modo activo, descrito en el CUS-04 o en el CUS-05 según corresponda.

### 4.2 Subflujos

Ninguno.

### 4.3 Flujos Alternos

**FA-01: El pedido ya no está en PENDING**

En el paso 4.1.6, si el pedido fue cancelado por el Cliente antes de que la Tienda / Dispatcher respondiera, el sistema rechaza la acción e informa a la Tienda / Dispatcher que el pedido ya no se encuentra disponible.

---

## 5. Precondiciones

### 5.1 Pedido en PENDING

El pedido existe y su estado actual es PENDING.

### 5.2 Tienda / Dispatcher autenticada

La Tienda / Dispatcher se encuentra autenticada y el pedido pertenece a su tienda.

### 5.3 Sucursal activa disponible

Existe al menos una sucursal activa asociada a la tienda, disponible para ser seleccionada como punto de despacho.

---

## 6. Poscondiciones

### 6.1 Pedido en CONFIRMED

El pedido queda en estado CONFIRMED, con los campos `storeLocationId`, `estimatedPrepTime` y `confirmed_at` registrados.

### 6.2 Cliente notificado

El Cliente ha recibido la notificación de confirmación de su pedido.

### 6.3 Proceso de asignación iniciado

El pedido ha transitado al estado WAITING_DRIVER y el proceso de búsqueda de repartidor ha comenzado, según lo descrito en el CUS-04 o en el CUS-05.

---

## 7. Puntos de Extensión

Ninguno en este caso de uso.

---

## 8. Requerimientos Especiales

8.1 La notificación dirigida al Cliente debe enviarse de forma inmediata una vez ejecutada la confirmación.

---

## 9. Información Adicional

La confirmación del pedido constituye el evento que desencadena la búsqueda de repartidor. Cuando la tienda cuenta con una única sucursal activa, el sistema puede preseleccionarla automáticamente; cuando cuenta con varias, la selección resulta obligatoria por parte de la Tienda / Dispatcher.

---

---

# Especificación del Caso de Uso del Sistema: CUS-03. Rechazar Pedido

---

## 1. Actores del Sistema

### 1.1 Tienda / Dispatcher

Actor primario. Decide no atender el pedido que ha recibido.

---

## 2. Propósito

Permitir a la Tienda / Dispatcher declinar un pedido en estado PENDING que no se encuentra en condiciones de ser atendido, registrando la cancelación con el tipo correspondiente y notificando al Cliente.

---

## 3. Breve Descripción

La Tienda / Dispatcher revisa un pedido en estado PENDING y decide no atenderlo. El sistema registra la cancelación con el tipo STORE_FAULT_BEFORE_DRIVER_PAID, transiciona el pedido al estado CANCELLED y notifica al Cliente, sin generar entradas en el ledger financiero. El caso de uso concluye cuando el pedido se encuentra en estado CANCELLED y el Cliente ha sido notificado.

---

## 4. Flujo de Eventos

### 4.1 Flujo Básico

4.1.1 La Tienda / Dispatcher accede a los detalles del pedido que se encuentra en estado PENDING.

4.1.2 La Tienda / Dispatcher decide rechazar el pedido e ingresa el motivo del rechazo.

4.1.3 La Tienda / Dispatcher confirma la acción de rechazo.

4.1.4 El sistema valida que el pedido se encuentre en estado PENDING, conforme a la regla RN-03.

4.1.5 El sistema registra el tipo de cancelación `STORE_FAULT_BEFORE_DRIVER_PAID` así como el motivo indicado en el campo `cancellation_reason`.

4.1.6 El sistema transiciona el pedido al estado CANCELLED.

4.1.7 El sistema notifica al Cliente, a través de FCM, que su pedido fue rechazado por la tienda.

4.1.8 El sistema se abstiene de generar ninguna entrada en el ledger financiero, en aplicación de la anulación total prevista por la regla RN-05 para este tipo de cancelación.

### 4.2 Subflujos

Ninguno.

### 4.3 Flujos Alternos

**FA-01: El pedido ya no está en PENDING**

En el paso 4.1.4, si el pedido ya no se encuentra en PENDING —por haber sido cancelado por el Cliente o por haber transitado a otro estado— el sistema rechaza la acción e informa de ello a la Tienda / Dispatcher.

---

## 5. Precondiciones

### 5.1 Pedido en PENDING

El pedido existe y su estado actual es PENDING.

### 5.2 Tienda / Dispatcher autenticada

La Tienda / Dispatcher se encuentra autenticada y el pedido pertenece a su tienda.

---

## 6. Poscondiciones

### 6.1 Pedido en CANCELLED

El pedido queda en estado CANCELLED, con el `cancellation_type` establecido en `STORE_FAULT_BEFORE_DRIVER_PAID` y el `cancellation_reason` registrado.

### 6.2 Cliente notificado

El Cliente ha recibido la notificación correspondiente al rechazo de su pedido.

### 6.3 Sin entradas financieras

No se ha generado ninguna entrada de comisión ni de ingreso en el ledger financiero.

---

## 7. Puntos de Extensión

Ninguno en este caso de uso.

---

## 8. Requerimientos Especiales

8.1 La notificación dirigida al Cliente debe enviarse de forma inmediata una vez ejecutado el rechazo.

---

## 9. Información Adicional

Este caso de uso representa la acción de rechazo por parte de la Tienda / Dispatcher, la cual es distinta de la acción de cancelación que puede ejecutar el Cliente, según se describe en el CUS-07 y en la regla RN-04. Ambas acciones producen el estado CANCELLED, pero se distinguen por el tipo de cancelación registrado y por el actor que las ejecuta.

---

---

# Especificación del Caso de Uso del Sistema: CUS-04. Asignar Repartidor Automáticamente

---

## 1. Actores del Sistema

### 1.1 Sistema de Asignación

Actor primario, de naturaleza secundaria y automatizada. Se activa por sí mismo cuando el pedido alcanza el estado WAITING_DRIVER bajo el modo AUTO_PROXIMITY.

### 1.2 Repartidor

Actor secundario. Interviene como candidato designado por el Sistema de Asignación y es quien, mediante su aceptación explícita, determina que el pedido pueda transicionar al estado ACCEPTED (paso 4.1.6).

---

## 2. Propósito

Seleccionar de manera automática al Repartidor disponible más adecuado dentro de la región del pedido, notificarlo y registrar la asignación correspondiente, habilitando de esta forma al Repartidor para aceptar el pedido.

---

## 3. Breve Descripción

Una vez que el pedido alcanza el estado WAITING_DRIVER encontrándose activo el modo AUTO_PROXIMITY, el Sistema de Asignación consulta los repartidores disponibles en la región correspondiente, selecciona un candidato y lo designa como responsable del pedido, transicionándolo al estado DRIVER_ASSIGNED. El candidato designado debe aceptar explícitamente el pedido; si no responde dentro del tiempo establecido, el Sistema de Asignación reintenta el proceso con el siguiente candidato disponible. El caso de uso concluye en el momento en que un Repartidor acepta el pedido, alcanzando este el estado ACCEPTED.

---

## 4. Flujo de Eventos

### 4.1 Flujo Básico

4.1.1 El pedido alcanza el estado WAITING_DRIVER encontrándose activo el modo de asignación AUTO_PROXIMITY, conforme a la regla RN-08.

4.1.2 El Sistema de Asignación consulta los repartidores que satisfacen simultáneamente las siguientes condiciones: encontrarse conectados (`is_online = true`), registrar un número de pedidos activos inferior a su `max_concurrent_orders` según su rango, y pertenecer a una región geográfica coincidente con la del pedido, conforme a la regla RN-06.

4.1.3 El Sistema de Asignación selecciona al candidato más adecuado entre los repartidores disponibles.

4.1.4 El sistema transiciona el pedido al estado DRIVER_ASSIGNED, asigna el identificador del candidato (`driverId`) y registra la marca de tiempo `assigned_at`.

4.1.5 El sistema notifica al candidato seleccionado, a través de FCM, incluyendo los detalles del pedido.

4.1.6 El Repartidor candidato acepta el pedido, según se describe en el paso 4.1.3 del CUS-06.

4.1.7 El sistema transiciona el pedido al estado ACCEPTED.

### 4.2 Subflujos

Ninguno.

### 4.3 Flujos Alternos

**FA-01: No hay Repartidores disponibles en la región**

En el paso 4.1.2, si no existe ningún Repartidor que satisfaga las condiciones de disponibilidad requeridas, el pedido permanece en el estado WAITING_DRIVER. El sistema genera, en este caso, una alerta de pedido huérfano (orphaned order) dirigida al Administrador a través de Telegram, con el propósito de que este intervenga manualmente.

**FA-02: El candidato designado no acepta dentro del tiempo de espera**

En el paso 4.1.6, si el candidato designado no responde dentro del tiempo establecido, el Sistema de Asignación reinicia la selección desde el paso 4.1.2, considerando al siguiente candidato disponible. Durante este reintento, el pedido permanece en el estado DRIVER_ASSIGNED, sin retroceder al estado WAITING_DRIVER, conforme a la regla RN-02. Si se agotan los candidatos disponibles, se aplica el flujo alterno FA-01.

---

## 5. Precondiciones

### 5.1 Pedido en WAITING_DRIVER

El pedido existe y su estado actual es WAITING_DRIVER.

### 5.2 Modo AUTO_PROXIMITY activo

El parámetro global de asignación tiene el valor AUTO_PROXIMITY, conforme a la regla RN-08.

---

## 6. Poscondiciones

### 6.1 Pedido en ACCEPTED

El pedido queda en estado ACCEPTED, con el `driverId`, el `assigned_at` y el `accepted_at` correspondientes registrados.

### 6.2 Repartidor notificado y comprometido

El Repartidor que aceptó el pedido ha sido notificado y este figura entre sus órdenes activas.

---

## 7. Puntos de Extensión

Ninguno en este caso de uso.

---

## 8. Requerimientos Especiales

8.1 La selección del candidato y su notificación deben ejecutarse sin intervención humana y completarse dentro de un tiempo mínimo posterior a la llegada del pedido al estado WAITING_DRIVER.

---

## 9. Información Adicional

El modelo de asignación empleado es determinístico: el Sistema de Asignación designa a un candidato específico —transicionando el pedido a DRIVER_ASSIGNED— y aguarda su aceptación explícita —transicionándolo a ACCEPTED— en lugar de recurrir a un mecanismo de difusión abierta (broadcast) en el que resulte asignado el primer repartidor en responder. Este comportamiento es consistente con la descripción del actor Sistema de Asignación establecida en el documento DS-ACT-01.

---

---

# Especificación del Caso de Uso del Sistema: CUS-05. Asignar Repartidor Manualmente

---

## 1. Actores del Sistema

### 1.1 Administrador

Actor primario. Selecciona y asigna manualmente al Repartidor que atenderá el pedido.

---

## 2. Propósito

Permitir al Administrador asignar directamente un Repartidor específico a un pedido en espera, cuando el modo de asignación activo es ADMIN_MANUAL.

---

## 3. Breve Descripción

El Administrador identifica un pedido en estado WAITING_DRIVER, consulta la lista de repartidores disponibles y selecciona a aquel que considera más adecuado. El sistema valida la disponibilidad del Repartidor seleccionado, registra la asignación y lo notifica. El caso de uso concluye en el momento en que el Repartidor ha sido asignado y notificado; el flujo posterior de aceptación y entrega continúa en el CUS-06.

---

## 4. Flujo de Eventos

### 4.1 Flujo Básico

4.1.1 El Administrador accede al panel de administración y visualiza la lista de pedidos que se encuentran en estado WAITING_DRIVER.

4.1.2 El Administrador selecciona el pedido que desea asignar.

4.1.3 El Administrador consulta la lista de repartidores disponibles para dicho pedido, es decir, aquellos que se encuentran conectados, dentro de su límite de pedidos simultáneos y con región coincidente, conforme a la regla RN-06.

4.1.4 El Administrador selecciona al Repartidor deseado y confirma la asignación.

4.1.5 El sistema valida, en tiempo real, que el Repartidor seleccionado continúe cumpliendo las condiciones de disponibilidad, conforme a la regla RN-06.

4.1.6 El sistema registra la asignación, transiciona el pedido al estado DRIVER_ASSIGNED y registra la marca de tiempo `assigned_at`.

4.1.7 El sistema notifica al Repartidor asignado, a través de FCM, incluyendo los detalles del pedido.

Sobre el flujo de estados posterior a DRIVER_ASSIGNED existe un supuesto pendiente de confirmación, consignado también en la regla RN-08: se asume que dicho flujo —la aceptación del Repartidor hacia el estado ACCEPTED y los estados subsiguientes de entrega— es idéntico al que se sigue en el modo AUTO_PROXIMITY, quedando pendiente confirmar con el equipo técnico si el Administrador cuenta con la opción de saltar directamente al estado ACCEPTED al momento de asignar manualmente.

### 4.2 Subflujos

Ninguno.

### 4.3 Flujos Alternos

**FA-01: El Repartidor ya no está disponible al confirmar**

En el paso 4.1.5, si el Repartidor seleccionado ha dejado de cumplir las condiciones de disponibilidad —por haberse desconectado, por haber alcanzado su límite de pedidos simultáneos o por haber cambiado de región— el sistema rechaza la asignación e informa al Administrador para que seleccione a otro candidato.

---

## 5. Precondiciones

### 5.1 Pedido en WAITING_DRIVER

El pedido existe y su estado actual es WAITING_DRIVER.

### 5.2 Modo ADMIN_MANUAL activo

El parámetro global de asignación tiene el valor ADMIN_MANUAL, conforme a la regla RN-08.

### 5.3 Administrador autenticado

El Administrador se encuentra autenticado y cuenta con acceso al panel de administración.

---

## 6. Poscondiciones

### 6.1 Pedido en DRIVER_ASSIGNED

El pedido queda en estado DRIVER_ASSIGNED, con el `driverId` y el `assigned_at` correspondientes registrados.

### 6.2 Repartidor notificado

El Repartidor asignado ha recibido la notificación del pedido.

---

## 7. Puntos de Extensión

Ninguno en este caso de uso.

---

## 8. Requerimientos Especiales

8.1 La lista de repartidores disponibles presentada al Administrador debe reflejar, en tiempo real, el estado de disponibilidad efectivo de cada Repartidor.

---

## 9. Información Adicional

El presente caso de uso constituye la alternativa manual al CUS-04. El Administrador tiene la facultad de alternar el modo de asignación global entre AUTO_PROXIMITY y ADMIN_MANUAL desde la configuración del sistema, conforme a la regla RN-08.

---

---

# Especificación del Caso de Uso del Sistema: CUS-06. Ejecutar Entrega

---

## 1. Actores del Sistema

### 1.1 Repartidor

Actor primario. Acepta el pedido asignado y registra cada hito del proceso de entrega.

---

## 2. Propósito

Permitir al Repartidor aceptar el pedido que le ha sido asignado y avanzarlo a través de los estados operativos de entrega hasta su conclusión, desencadenando en ese momento el registro de las comisiones correspondientes.

---

## 3. Breve Descripción

El Repartidor recibe la notificación de que le ha sido asignado un pedido y decide aceptarlo. A partir de ese instante, registra cada hito de la entrega: su llegada al local de la tienda, la recogida del pedido, su llegada al destino y la confirmación de la entrega. En cada uno de estos pasos, el sistema transiciona el pedido, notifica a los actores correspondientes y, al alcanzar el estado DELIVERED, calcula y registra en el ledger financiero las comisiones de plataforma y de Repartidor. El caso de uso concluye cuando el pedido se encuentra en estado DELIVERED y dichas comisiones han sido registradas.

---

## 4. Flujo de Eventos

### 4.1 Flujo Básico

4.1.1 El Repartidor recibe, a través de FCM, la notificación de un pedido asignado y accede a sus detalles, entre ellos la tienda, la dirección de entrega y los ítems solicitados.

4.1.2 El Repartidor revisa los detalles del pedido y decide aceptarlo.

4.1.3 El Repartidor confirma la aceptación; el sistema transiciona el pedido al estado ACCEPTED y registra la marca de tiempo `accepted_at`.

4.1.4 El sistema notifica al Cliente, a través de FCM, que el Repartidor se encuentra en camino.

4.1.5 El Repartidor llega al local de la Tienda / Dispatcher y registra su llegada en la aplicación; el sistema transiciona el pedido al estado AT_STORE y registra la marca de tiempo `arrived_at`.

4.1.6 El sistema notifica a la Tienda / Dispatcher, a través de FCM, que el Repartidor se encuentra en el local.

4.1.7 El Repartidor recoge el pedido y registra la recogida; el sistema transiciona el pedido al estado PICKED_UP y registra la marca de tiempo `picked_up_at`.

4.1.8 El sistema notifica al Cliente, a través de FCM, que el pedido ha sido recogido y se encuentra en camino.

4.1.9 El Repartidor llega a la dirección de entrega del Cliente y registra su llegada; el sistema transiciona el pedido al estado AT_CUSTOMER y registra la marca de tiempo `at_customer_at`.

4.1.10 El sistema notifica al Cliente, a través de FCM, que el Repartidor ha llegado a su ubicación.

4.1.11 El Repartidor entrega el pedido al Cliente y registra la entrega; el sistema transiciona el pedido al estado DELIVERED y registra la marca de tiempo `delivered_at`.

4.1.12 El sistema ejecuta el registro de comisiones, descrito en el subflujo SF-01.

4.1.13 El sistema notifica al Cliente, a través de FCM, que su pedido fue entregado, solicitándole a la vez una calificación del servicio recibido.

4.1.14 El sistema notifica a la Tienda / Dispatcher, a través de Telegram, que el pedido fue entregado.

### 4.2 Subflujos

**SF-01: Registro de comisiones al entregar**

4.2.1 El sistema determina el tipo de tienda (`store_type`) involucrada en el pedido, el cual puede ser OFFICIAL_10, OFFICIAL_5 o NON_OFFICIAL.

4.2.2 El sistema calcula la comisión de plataforma sobre el subtotal según el tipo de tienda determinado, conforme a la regla RN-15. Cuando el tipo es OFFICIAL_10, la comisión equivale al 10 % del subtotal. Cuando el tipo es OFFICIAL_5, se calcula primero una base equivalente al subtotal dividido entre 1.05, y la comisión corresponde al 10 % de dicha base. Cuando el tipo es NON_OFFICIAL, la base se obtiene dividiendo el subtotal entre 1.10, y la comisión equivale a la diferencia entre el subtotal y dicha base.

4.2.3 Si el tipo de tienda es OFFICIAL_10 y el método de pago empleado es CARD, el sistema registra adicionalmente un ingreso de tienda equivalente al subtotal (`store_revenue = subtotal`) en el ledger. En los pagos realizados en efectivo (CASH) no se genera esta entrada, conforme a la regla RN-15.

4.2.4 El sistema resuelve la tasa de comisión del Repartidor siguiendo la jerarquía establecida en la regla RN-16: en primer lugar el valor individual del Repartidor, en su defecto la configuración por ciudad, en su defecto la configuración global, y como último recurso el valor de reserva definido en la entidad `Settings`.

4.2.5 El sistema calcula la comisión del Repartidor mediante la expresión `comisión_driver = delivery_fee × (commission_rate / 100)`, conforme a la regla RN-17.

4.2.6 El sistema registra todas las entradas correspondientes en la tabla `financial_ledger`.

### 4.3 Flujos Alternos

**FA-01: El Repartidor no acepta el pedido dentro del tiempo de espera**

En el paso 4.1.3, si el Repartidor no acepta el pedido dentro del tiempo establecido, el Sistema de Asignación reintenta la asignación con el siguiente candidato disponible, conforme al flujo alterno FA-02 del CUS-04. En tal circunstancia, el presente caso de uso no llega a activarse para este Repartidor.

---

## 5. Precondiciones

### 5.1 Pedido en DRIVER_ASSIGNED

El pedido existe, su estado actual es DRIVER_ASSIGNED y tiene registrado el `driverId` del Repartidor correspondiente.

### 5.2 Repartidor autenticado y disponible

El Repartidor se encuentra autenticado y satisface las condiciones de disponibilidad al momento de aceptar el pedido, conforme a la regla RN-06.

---

## 6. Poscondiciones

### 6.1 Pedido en DELIVERED

El pedido queda en estado DELIVERED, con la marca de tiempo `delivered_at` y con todas las marcas de tiempo de hito registradas: `accepted_at`, `arrived_at`, `picked_up_at`, `at_customer_at` y `delivered_at`.

### 6.2 Comisiones registradas

Las comisiones de plataforma —aplicadas sobre la tienda— y del Repartidor han sido calculadas y registradas en `financial_ledger`, conforme a las reglas RN-15, RN-16 y RN-17.

### 6.3 Actores notificados

El Cliente, la Tienda / Dispatcher y el Repartidor han recibido sus respectivas notificaciones relativas a la conclusión de la entrega.

---

## 7. Puntos de Extensión

### 7.1 Solicitud de calificación del servicio

Definido en el paso 4.1.13 del flujo básico, en el momento en que el pedido alcanza el estado DELIVERED y el sistema notifica al Cliente incluyendo el aviso de calificación. Este punto extiende hacia el CUS-09 (Calificar Pedido), el cual retoma la ejecución a partir de dicho aviso.

---

## 8. Requerimientos Especiales

8.1 Las notificaciones correspondientes a cada transición de estado deben enviarse de forma inmediata, con el fin de mantener informados en tiempo real a todos los actores involucrados.

8.2 El registro de comisiones descrito en el subflujo SF-01 debe ejecutarse de forma atómica junto con la transición al estado DELIVERED, con el propósito de garantizar la consistencia del ledger financiero.

---

## 9. Información Adicional

El cálculo de comisiones se rige por las Reglas del Negocio RN-15, RN-16 y RN-17. Los tipos de tienda —OFFICIAL_10, OFFICIAL_5 y NON_OFFICIAL— constituyen atributos configurados por el Administrador para cada tienda en particular.

---

---

# Especificación del Caso de Uso del Sistema: CUS-07. Cancelar Pedido

---

## 1. Actores del Sistema

### 1.1 Cliente

Actor primario para la cancelación cuando el pedido se encuentra únicamente en estado PENDING.

### 1.2 Administrador

Actor primario para la cancelación desde cualquier estado activo posterior a PENDING.

### 1.3 Soporte

Actor primario para la cancelación desde cualquier estado activo posterior a PENDING.

---

## 2. Propósito

Registrar la cancelación de un pedido en curso, aplicando el tipo de cancelación que corresponda según el actor que la ejecuta y el estado en que se encuentra el pedido, y notificar a todos los actores involucrados.

---

## 3. Breve Descripción

Un actor autorizado solicita la cancelación de un pedido activo. El sistema valida que dicho actor cuente con permiso para cancelar según el estado actual del pedido, conforme a la regla RN-04, registra el tipo de cancelación seleccionado y aplica las consecuencias financieras definidas en la regla RN-05. A continuación, se notifica a todos los actores involucrados en el pedido. El caso de uso concluye cuando el pedido queda en estado CANCELLED, con el tipo y el motivo de cancelación debidamente registrados.

---

## 4. Flujo de Eventos

### 4.1 Flujo Básico — Cancelación por el Cliente (estado PENDING)

4.1.1 El Cliente accede al detalle de su pedido, el cual se encuentra en estado PENDING.

4.1.2 El Cliente solicita la cancelación del pedido e indica el motivo correspondiente.

4.1.3 El sistema valida que el pedido se encuentre en estado PENDING y que el solicitante sea el Cliente propietario del pedido, conforme a la regla RN-04.

4.1.4 El sistema registra el tipo de cancelación `CLIENT_FAULT` y el motivo en el campo `cancellation_reason`.

4.1.5 El sistema transiciona el pedido al estado CANCELLED.

4.1.6 El sistema aplica las consecuencias financieras correspondientes al tipo CLIENT_FAULT, registrando una penalidad económica configurable en la cuenta del Cliente, conforme a la regla RN-05. El mecanismo de cálculo y cobro de dicha penalidad queda fuera del alcance de este documento.

4.1.7 El sistema notifica a la Tienda / Dispatcher, a través de FCM, que el pedido fue cancelado.

4.1.8 El sistema confirma al Cliente la cancelación de su pedido.

### 4.2 Subflujos

**SF-01: Cancelación por Administrador o Soporte (estado CONFIRMED o posterior)**

4.2.1 El Administrador o Soporte accede al panel correspondiente y localiza el pedido que desea cancelar.

4.2.2 El actor selecciona el tipo de cancelación aplicable —CLIENT_FAULT, STORE_FAULT_BEFORE_DRIVER_PAID, STORE_FAULT_AFTER_DRIVER_PAID o DRIVER_FAULT— e ingresa el motivo de la cancelación.

4.2.3 El sistema valida que el actor cuente con el rol requerido, ADMIN o SUPPORT, y que el pedido no se encuentre en estado DELIVERED ni en estado CANCELLED, conforme a la regla RN-04.

4.2.4 El sistema registra el tipo de cancelación en el campo `cancellation_type` y el motivo en el campo `cancellation_reason`.

4.2.5 El sistema transiciona el pedido al estado CANCELLED.

4.2.6 El sistema aplica las consecuencias financieras que correspondan al tipo registrado, conforme a la regla RN-05. Si el tipo es CLIENT_FAULT, se aplica una penalidad económica configurable al Cliente, registrándose con normalidad las comisiones de tienda y de Repartidor. Si el tipo es STORE_FAULT_BEFORE_DRIVER_PAID, se produce una anulación total sin generación de entradas en el ledger. Si el tipo es STORE_FAULT_AFTER_DRIVER_PAID, el Repartidor conserva la tarifa de envío y la tienda no genera comisión. Si el tipo es DRIVER_FAULT, las comisiones se registran con normalidad y el Repartidor pierde la entrega.

4.2.7 El sistema notifica a todos los actores involucrados en el pedido: al Cliente a través de FCM, a la Tienda / Dispatcher a través de FCM, y al Repartidor a través de FCM en caso de que ya hubiera sido asignado.

### 4.3 Flujos Alternos

**FA-01: El Cliente intenta cancelar un pedido en estado CONFIRMED o posterior**

En el paso 4.1.3, si el pedido ya no se encuentra en PENDING, el sistema rechaza la acción del Cliente e informa que, para cancelar un pedido que ya ha sido confirmado, debe contactar a Soporte.

**FA-02: El pedido ya está en DELIVERED o CANCELLED**

En cualquier paso de validación, si el pedido se encuentra en estado DELIVERED o en estado CANCELLED, el sistema rechaza la acción e informa que el pedido no admite ningún cambio de estado adicional, conforme a la regla RN-02.

---

## 5. Precondiciones

### 5.1 Pedido activo

El pedido existe y su estado no es DELIVERED ni CANCELLED.

### 5.2 Actor autorizado según el estado

El actor solicitante cuenta con permiso para ejecutar la cancelación en el estado actual del pedido: el Cliente únicamente puede cancelar en el estado PENDING, mientras que el Administrador y Soporte pueden cancelar en cualquier estado activo posterior a PENDING, conforme a la regla RN-04.

---

## 6. Poscondiciones

### 6.1 Pedido en CANCELLED

El pedido queda en estado CANCELLED, con el `cancellation_type` y el `cancellation_reason` registrados.

### 6.2 Consecuencias financieras aplicadas

Las consecuencias financieras definidas en la regla RN-05 para el tipo de cancelación registrado han sido aplicadas.

### 6.3 Actores notificados

Todos los actores involucrados en el pedido al momento de la cancelación han recibido su respectiva notificación.

---

## 7. Puntos de Extensión

Ninguno en este caso de uso.

---

## 8. Requerimientos Especiales

8.1 El sistema debe validar el estado del pedido y los permisos del actor en tiempo real antes de ejecutar la cancelación, con el fin de evitar condiciones de carrera en pedidos con múltiples actores activos de manera simultánea.

---

## 9. Información Adicional

La distinción entre la cancelación, entendida como una acción del Cliente, y el rechazo, entendido como una acción de la Tienda / Dispatcher, se encuentra definida en la regla RN-04 y en el CUS-03. Ambas acciones producen el estado CANCELLED, pero constituyen acciones distintas, con actores y tipos de cancelación diferentes. Las consecuencias financieras de cada tipo de cancelación se detallan en la regla RN-05 del documento DS-RN-01.

---

---

# Especificación del Caso de Uso del Sistema: CUS-08. Consultar Estado / Seguimiento del Pedido

---

## 1. Actores del Sistema

### 1.1 Cliente

Actor primario. Consulta el estado actual de su pedido en cualquier momento durante el ciclo de vida activo de este.

---

## 2. Propósito

Permitir al Cliente conocer, en tiempo real, el estado de su pedido y, una vez que se le ha asignado un Repartidor, visualizar los hitos de la entrega conforme estos se van produciendo.

---

## 3. Breve Descripción

El Cliente accede al detalle de un pedido activo con el fin de conocer su estado actual y, de existir, la información del Repartidor asignado. El sistema muestra el estado vigente y mantiene la vista actualizada mediante notificaciones push (FCM) ante cada cambio de estado. El caso de uso puede iniciarse en cualquier momento mientras el pedido no se encuentre en estado DELIVERED ni CANCELLED, y también permite revisar pedidos que ya han concluido, a través del historial correspondiente.

---

## 4. Flujo de Eventos

### 4.1 Flujo Básico

4.1.1 El Cliente accede a la sección de pedidos activos o al historial de la aplicación.

4.1.2 El Cliente selecciona el pedido que desea consultar.

4.1.3 El sistema recupera el estado actual del pedido junto con los datos asociados: la tienda, los ítems, la tarifa, la dirección de entrega y las marcas de tiempo de los hitos ya registrados.

4.1.4 Si el pedido cuenta con un Repartidor asignado, es decir, se encuentra en estado ACCEPTED o en un estado posterior, el sistema muestra los datos del Repartidor —nombre, fotografía y rango— junto con el estado actual de la entrega.

4.1.5 El sistema presenta al Cliente la vista de detalle con el estado vigente del pedido.

4.1.6 Mientras el pedido permanezca activo, el sistema actualiza automáticamente la vista del Cliente ante cada cambio de estado, a partir de las notificaciones FCM recibidas en segundo plano.

### 4.2 Subflujos

Ninguno.

### 4.3 Flujos Alternos

**FA-01: El pedido no tiene Repartidor asignado aún**

En el paso 4.1.4, si el pedido se encuentra en estado PENDING, CONFIRMED o WAITING_DRIVER, el sistema omite la sección correspondiente al Repartidor e informa al Cliente que su pedido se encuentra en proceso de preparación o de búsqueda de repartidor, según corresponda.

---

## 5. Precondiciones

### 5.1 Cliente autenticado

El Cliente se encuentra autenticado en el sistema.

### 5.2 Pedido existente y propio

El pedido consultado existe y pertenece al Cliente solicitante.

---

## 6. Poscondiciones

### 6.1 Sin cambio de estado

Este caso de uso es de solo lectura; su ejecución no produce ninguna transición de estado sobre el pedido ni sobre ninguna otra entidad del sistema.

---

## 7. Puntos de Extensión

Ninguno en este caso de uso.

---

## 8. Requerimientos Especiales

8.1 La vista de estado debe reflejar el estado real del pedido en el momento de la consulta; los cambios que se produzcan con posterioridad se comunican mediante notificaciones FCM, sin requerir ninguna acción adicional por parte del Cliente.

---

## 9. Información Adicional

Las actualizaciones automáticas de estado —CONFIRMED, ACCEPTED, PICKED_UP, AT_CUSTOMER y DELIVERED— son iniciadas por las notificaciones FCM que el sistema emite en cada transición, descritas en los flujos básicos del CUS-02, del CUS-04 o CUS-05, y del CUS-06.

---

---

# Especificación del Caso de Uso del Sistema: CUS-09. Calificar Pedido

---

## 1. Actores del Sistema

### 1.1 Cliente

Actor primario. Emite una calificación del servicio recibido una vez concluida la entrega del pedido.

---

## 2. Propósito

Permitir al Cliente registrar una calificación del servicio de entrega una vez que su pedido ha sido marcado como DELIVERED, contribuyendo con ello a la evaluación del desempeño del sistema y de los actores involucrados.

---

## 3. Breve Descripción

Tras recibir la notificación de que su pedido fue entregado, el sistema presenta al Cliente un aviso (prompt) de calificación. El Cliente puede emitir una puntuación y, de manera opcional, un comentario. El caso de uso concluye cuando la calificación queda registrada, o bien cuando el Cliente decide omitirla.

---

## 4. Flujo de Eventos

### 4.1 Flujo Básico

4.1.1 El sistema notifica al Cliente, a través de FCM, que su pedido ha sido entregado —esto es, que ha alcanzado el estado DELIVERED— incluyendo en dicha notificación un aviso para calificar el servicio, conforme al paso 4.1.13 del CUS-06.

4.1.2 El Cliente accede al aviso de calificación desde la notificación recibida o desde el detalle del pedido.

4.1.3 El Cliente selecciona una puntuación para el servicio recibido.

4.1.4 El Cliente puede, de manera opcional, agregar un comentario de texto libre.

4.1.5 El Cliente envía la calificación.

4.1.6 El sistema registra la calificación asociada al pedido y actualiza los indicadores de desempeño correspondientes.

4.1.7 El sistema confirma al Cliente que su calificación ha sido registrada.

### 4.2 Subflujos

Ninguno.

### 4.3 Flujos Alternos

**FA-01: El Cliente omite la calificación**

En el paso 4.1.2 o en el paso 4.1.5, si el Cliente cierra el aviso sin haber enviado una calificación, el sistema registra dicha omisión sin generar ningún error. El aviso puede volver a presentarse al Cliente la próxima vez que este acceda al detalle del pedido, siempre que la calificación no haya sido registrada previamente.

---

## 5. Precondiciones

### 5.1 Pedido en DELIVERED

El pedido existe, pertenece al Cliente y su estado actual es DELIVERED.

### 5.2 Calificación no registrada previamente

El pedido no cuenta aún con una calificación registrada por parte del Cliente.

### 5.3 Cliente autenticado

El Cliente se encuentra autenticado en el sistema.

---

## 6. Poscondiciones

### 6.1 Calificación registrada

La puntuación, junto con el comentario opcional en caso de haberse proporcionado, queda registrada y asociada al pedido correspondiente.

---

## 7. Puntos de Extensión

Ninguno en este caso de uso.

---

## 8. Requerimientos Especiales

8.1 La calificación debe poder registrarse una única vez por pedido y por parte del Cliente; los intentos posteriores deben ser rechazados, informando al respecto.

---

## 9. Información Adicional

El aviso de calificación es iniciado por el sistema como parte del cierre del ciclo del pedido, según lo descrito en el paso 4.1.13 del CUS-06. La calificación es de carácter opcional; su omisión no bloquea ningún flujo posterior del Cliente.
