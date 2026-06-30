# DeliverySuite

# Proyecto: DeliverySuite

# Especificaciones de Caso de Uso del Sistema

## Versión 1.0

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

---

---

# Especificación del Caso de Uso del Sistema: CUS-01. Crear Pedido

---

## 1. Actores del Sistema

### 1.1 Cliente
Actor primario. Inicia el caso de uso al seleccionar una tienda y sus productos para realizar un pedido.

---

## 2. Propósito

Permitir al Cliente crear un nuevo pedido seleccionando productos de una tienda afiliada activa, especificando la dirección de entrega y el método de pago, de modo que el sistema registre el pedido y notifique a la Tienda / Dispatcher.

---

## 3. Breve Descripción

El Cliente selecciona una tienda y uno o más productos de su catálogo, indica la dirección de entrega con coordenadas geográficas y elige un método de pago. El sistema calcula la tarifa de envío, valida los datos y registra el pedido en estado PENDING con un snapshot inmutable de los precios de los ítems. Al concluir, notifica a la Tienda / Dispatcher para que inicie su revisión. El caso de uso termina cuando el pedido queda registrado y la notificación ha sido enviada.

---

## 4. Flujo de Eventos

### 4.1 Flujo Básico

4.1.1 El Cliente selecciona una tienda afiliada activa y visible en la plataforma.

4.1.2 El Cliente selecciona uno o más productos del catálogo de la tienda, indicando cantidad y, si aplica, variante.

4.1.3 El Cliente especifica la dirección de entrega con coordenadas geográficas.

4.1.4 El sistema ejecuta el cálculo de la tarifa de envío (SF-01).

4.1.5 El Cliente selecciona el método de pago (CASH, CARD o WALLET).

4.1.6 El Cliente revisa el resumen del pedido (ítems, subtotal, tarifa de envío, total) y lo confirma.

4.1.7 El sistema valida que el pedido cumpla con las condiciones mínimas: al menos un ítem, dirección con coordenadas válidas, tienda activa y Cliente autenticado (RN-01).

4.1.8 El sistema registra el pedido en estado PENDING, guardando el snapshot inmutable de nombre, precio unitario, subtotal y variante de cada ítem (RN-18).

4.1.9 El sistema envía una notificación a la Tienda / Dispatcher informando la nueva orden (FCM y, si está configurado, Telegram).

4.1.10 El sistema retorna al Cliente la confirmación del pedido creado con su identificador y estado PENDING.

### 4.2 Subflujos

**SF-01: Cálculo de tarifa de envío**

4.2.1 El sistema calcula la distancia en kilómetros entre las coordenadas de la tienda (o sucursal, si ya está definida) y el punto de entrega del Cliente usando la fórmula de Haversine.

4.2.2 Si el modo de cálculo activo es ZONE (`fee_mode = 'zone'`): el sistema evalúa cada zona de entrega activa y determina si el punto de entrega cae dentro de su polígono GeoJSON. Cuando se encuentra una zona coincidente, calcula `tarifa = tarifa_base + (distancia_km × tarifa_km_extra)` y aplica el rango `[tarifa_mínima, tarifa_máxima]` de esa zona (RN-11). Si ninguna zona coincide, aplica el fallback de S/ 2.00 por kilómetro.

4.2.3 Si el modo de cálculo activo es DISTANCE (`fee_mode = 'distance'`): el sistema calcula `tarifa = distancia_km × precio_por_km` y aplica el rango `[tarifa_mínima, tarifa_máxima]` configurados globalmente (RN-12).

4.2.4 El sistema aplica las reglas de tarifa activas (`delivery_fee_rules`) en orden ascendente de prioridad, evaluando para cada regla si aplica según ciudad, día de la semana y ventana horaria vigente (RN-13).

4.2.5 El sistema multiplica la tarifa resultante por el factor de la tienda (`fee_multiplier`) y redondea al medio sol más cercano hacia arriba (RN-14).

4.2.6 El valor final queda fijo en el campo `delivery_fee` del pedido y no se recalcula en estados posteriores (RN-09).

### 4.3 Flujos Alternos

**FA-01: La tienda está cerrada o inactiva**

En el paso 4.1.1, si la tienda seleccionada tiene `is_open = false` o `is_hidden = true`, el sistema informa al Cliente que la tienda no está disponible en este momento y no permite continuar con el pedido.

**FA-02: El carrito no contiene ítems**

En el paso 4.1.6, si el Cliente intenta confirmar sin haber seleccionado ningún producto, el sistema rechaza la solicitud e informa que el pedido debe contener al menos un ítem.

**FA-03: Dirección de entrega sin coordenadas válidas**

En el paso 4.1.7, si la dirección de entrega no posee coordenadas geográficas válidas, el sistema rechaza la solicitud e indica al Cliente que corrija la dirección.

---

## 5. Precondiciones

### 5.1 Cliente autenticado
El Cliente debe estar registrado y autenticado en el sistema antes de iniciar el caso de uso.

### 5.2 Tienda activa
La tienda seleccionada debe existir, estar activa (`is_open = true`) y no estar oculta (`is_hidden = false`).

---

## 6. Poscondiciones

### 6.1 Pedido registrado en PENDING
El pedido queda persistido en la base de datos con estado PENDING, incluyendo el snapshot de ítems, la tarifa de envío calculada y la marca de tiempo `created_at`.

### 6.2 Snapshot de precios inmutable
Los precios, nombres y variantes de los ítems quedan registrados en `order_items` y no se modifican ante cambios posteriores en el catálogo (RN-18).

### 6.3 Tienda / Dispatcher notificada
La Tienda / Dispatcher ha recibido la notificación del nuevo pedido para iniciar su revisión.

---

## 7. Puntos de Extensión

Ninguno en este caso de uso.

---

## 8. Requerimientos Especiales

8.1 El cálculo de la tarifa de envío debe completarse en tiempo de respuesta de la solicitud de creación del pedido, sin retrasos perceptibles para el Cliente.

8.2 La notificación a la Tienda / Dispatcher debe enviarse de forma inmediata tras la creación del pedido.

---

## 9. Información Adicional

El cálculo de la tarifa de envío se rige por las Reglas del Negocio RN-09 a RN-14. El modo de cálculo activo (ZONE o DISTANCE) es un parámetro global configurable por el Administrador (RN-10).

---

---

# Especificación del Caso de Uso del Sistema: CUS-02. Confirmar Pedido

---

## 1. Actores del Sistema

### 1.1 Tienda / Dispatcher
Actor primario. Recibe la notificación del pedido y ejecuta la confirmación.

---

## 2. Propósito

Permitir a la Tienda / Dispatcher revisar un pedido recibido y aceptarlo para su preparación, indicando la sucursal de despacho y el tiempo estimado de preparación, de modo que el sistema inicie la búsqueda de repartidor.

---

## 3. Breve Descripción

La Tienda / Dispatcher recibe la notificación de un nuevo pedido en estado PENDING, revisa sus detalles y decide confirmarlo. Al confirmar, selecciona la sucursal de despacho y estima el tiempo de preparación en minutos. El sistema transiciona el pedido a CONFIRMED, notifica al Cliente y activa automáticamente la búsqueda de repartidor (WAITING\_DRIVER). El caso de uso termina cuando el pedido queda en estado CONFIRMED y el proceso de asignación ha sido iniciado.

---

## 4. Flujo de Eventos

### 4.1 Flujo Básico

4.1.1 La Tienda / Dispatcher recibe la notificación de nuevo pedido (FCM y/o Telegram) y accede a sus detalles en la aplicación.

4.1.2 La Tienda / Dispatcher revisa los ítems del pedido, la dirección de entrega y las instrucciones del Cliente.

4.1.3 La Tienda / Dispatcher selecciona la sucursal de despacho (`storeLocationId`) desde las sucursales activas de su tienda.

4.1.4 La Tienda / Dispatcher ingresa el tiempo estimado de preparación en minutos (`estimatedPrepTime`).

4.1.5 La Tienda / Dispatcher confirma el pedido.

4.1.6 El sistema valida que el pedido se encuentre en estado PENDING (RN-03).

4.1.7 El sistema transiciona el pedido a CONFIRMED y registra `confirmed_at`.

4.1.8 El sistema notifica al Cliente vía FCM que su pedido ha sido confirmado.

4.1.9 El sistema transiciona automáticamente el pedido a WAITING\_DRIVER e inicia el proceso de asignación de repartidor según el modo activo (ver CUS-04 o CUS-05).

### 4.2 Subflujos

Ninguno.

### 4.3 Flujos Alternos

**FA-01: El pedido ya no está en PENDING**

En el paso 4.1.6, si el pedido fue cancelado por el Cliente antes de que la Tienda / Dispatcher respondiera, el sistema rechaza la acción e informa a la Tienda / Dispatcher que el pedido ya no está disponible.

---

## 5. Precondiciones

### 5.1 Pedido en PENDING
El pedido existe y su estado actual es PENDING.

### 5.2 Tienda / Dispatcher autenticada
La Tienda / Dispatcher está autenticada y el pedido pertenece a su tienda.

### 5.3 Sucursal activa disponible
Existe al menos una sucursal activa asociada a la tienda para seleccionar como punto de despacho.

---

## 6. Poscondiciones

### 6.1 Pedido en CONFIRMED
El pedido queda en estado CONFIRMED con `storeLocationId`, `estimatedPrepTime` y `confirmed_at` registrados.

### 6.2 Cliente notificado
El Cliente ha recibido la notificación de confirmación de su pedido.

### 6.3 Proceso de asignación iniciado
El pedido ha transitado a WAITING\_DRIVER y el proceso de búsqueda de repartidor ha comenzado (CUS-04 o CUS-05).

---

## 7. Puntos de Extensión

Ninguno en este caso de uso.

---

## 8. Requerimientos Especiales

8.1 La notificación al Cliente debe enviarse de forma inmediata tras la confirmación.

---

## 9. Información Adicional

La confirmación del pedido es el evento que desencadena la búsqueda de repartidor. Si la tienda tiene una sola sucursal activa, el sistema puede preseleccionarla; si tiene varias, la selección es obligatoria por parte de la Tienda / Dispatcher.

---

---

# Especificación del Caso de Uso del Sistema: CUS-03. Rechazar Pedido

---

## 1. Actores del Sistema

### 1.1 Tienda / Dispatcher
Actor primario. Decide no atender el pedido recibido.

---

## 2. Propósito

Permitir a la Tienda / Dispatcher declinar un pedido en estado PENDING que no puede ser atendido, registrando la cancelación con el tipo correspondiente y notificando al Cliente.

---

## 3. Breve Descripción

La Tienda / Dispatcher revisa un pedido en PENDING y decide no atenderlo. El sistema registra la cancelación con tipo STORE\_FAULT\_BEFORE\_DRIVER\_PAID, transiciona el pedido a CANCELLED y notifica al Cliente. No se generan entradas en el ledger financiero. El caso de uso termina cuando el pedido queda en estado CANCELLED y el Cliente ha sido notificado.

---

## 4. Flujo de Eventos

### 4.1 Flujo Básico

4.1.1 La Tienda / Dispatcher accede a los detalles del pedido en estado PENDING.

4.1.2 La Tienda / Dispatcher decide rechazarlo e ingresa el motivo de rechazo.

4.1.3 La Tienda / Dispatcher confirma la acción de rechazo.

4.1.4 El sistema valida que el pedido se encuentre en estado PENDING (RN-03).

4.1.5 El sistema registra `cancellation_type = STORE_FAULT_BEFORE_DRIVER_PAID` y el motivo indicado en `cancellation_reason`.

4.1.6 El sistema transiciona el pedido a CANCELLED.

4.1.7 El sistema notifica al Cliente vía FCM que su pedido fue rechazado por la tienda.

4.1.8 El sistema no genera ninguna entrada en el ledger financiero (RN-05: anulación total).

### 4.2 Subflujos

Ninguno.

### 4.3 Flujos Alternos

**FA-01: El pedido ya no está en PENDING**

En el paso 4.1.4, si el pedido ya no se encuentra en PENDING (fue cancelado por el Cliente o transitó a otro estado), el sistema rechaza la acción e informa a la Tienda / Dispatcher.

---

## 5. Precondiciones

### 5.1 Pedido en PENDING
El pedido existe y su estado actual es PENDING.

### 5.2 Tienda / Dispatcher autenticada
La Tienda / Dispatcher está autenticada y el pedido pertenece a su tienda.

---

## 6. Poscondiciones

### 6.1 Pedido en CANCELLED
El pedido queda en estado CANCELLED con `cancellation_type = STORE_FAULT_BEFORE_DRIVER_PAID` y `cancellation_reason` registrados.

### 6.2 Cliente notificado
El Cliente ha recibido la notificación de rechazo.

### 6.3 Sin entradas financieras
No se han generado entradas de comisión ni de ingreso en el ledger financiero.

---

## 7. Puntos de Extensión

Ninguno en este caso de uso.

---

## 8. Requerimientos Especiales

8.1 La notificación al Cliente debe enviarse de forma inmediata tras el rechazo.

---

## 9. Información Adicional

Este caso de uso representa la acción de "rechazo" por parte de la Tienda / Dispatcher, que es distinta de la "cancelación" que puede ejecutar el Cliente (ver CUS-07 y RN-04). Ambas producen el estado CANCELLED pero con tipos de cancelación y actores diferentes.

---

---

# Especificación del Caso de Uso del Sistema: CUS-04. Asignar Repartidor Automáticamente

---

## 1. Actores del Sistema

### 1.1 Sistema de Asignación
Actor primario (secundario automatizado). Se activa automáticamente cuando el pedido alcanza WAITING\_DRIVER en modo AUTO\_PROXIMITY.

---

## 2. Propósito

Seleccionar automáticamente al Repartidor disponible más adecuado en la región del pedido, notificarlo y registrar la asignación, habilitando al Repartidor para aceptar el pedido.

---

## 3. Breve Descripción

Una vez que el pedido llega a WAITING\_DRIVER con el modo AUTO\_PROXIMITY activo, el Sistema de Asignación consulta los Repartidores disponibles en la región, selecciona un candidato y lo designa como responsable del pedido transicionando a DRIVER\_ASSIGNED. El candidato designado debe aceptar explícitamente. Si no responde en el tiempo establecido, el Sistema reintenta con el siguiente candidato disponible. El caso de uso termina cuando un Repartidor acepta el pedido (estado ACCEPTED).

---

## 4. Flujo de Eventos

### 4.1 Flujo Básico

4.1.1 El pedido alcanza el estado WAITING\_DRIVER y el modo de asignación activo es AUTO\_PROXIMITY (RN-08).

4.1.2 El Sistema de Asignación consulta los Repartidores que cumplen simultáneamente: `is_online = true`, número de pedidos activos menor que `max_concurrent_orders` según su rango, y región geográfica coincidente con la del pedido (RN-06).

4.1.3 El Sistema de Asignación selecciona al candidato más adecuado entre los disponibles.

4.1.4 El sistema transiciona el pedido a DRIVER\_ASSIGNED, asigna el `driverId` del candidato y registra `assigned_at`.

4.1.5 El sistema notifica al candidato vía FCM con los detalles del pedido.

4.1.6 El Repartidor candidato acepta el pedido (ver CUS-06, paso 4.1.3).

4.1.7 El sistema transiciona el pedido a ACCEPTED.

### 4.2 Subflujos

Ninguno.

### 4.3 Flujos Alternos

**FA-01: No hay Repartidores disponibles en la región**

En el paso 4.1.2, si no existe ningún Repartidor que cumpla las condiciones de disponibilidad, el pedido permanece en WAITING\_DRIVER. El sistema genera una alerta de pedido huérfano (orphaned order) al Administrador vía Telegram para que intervenga manualmente.

**FA-02: El candidato designado no acepta dentro del tiempo de espera**

En el paso 4.1.6, si el candidato no responde en el tiempo establecido, el Sistema de Asignación reinicia la selección desde el paso 4.1.2 con el siguiente candidato disponible. El pedido permanece en estado DRIVER\_ASSIGNED durante el reintento, sin retroceder a WAITING\_DRIVER (RN-02). Si se agotan los candidatos disponibles, se aplica FA-01.

---

## 5. Precondiciones

### 5.1 Pedido en WAITING_DRIVER
El pedido existe y su estado actual es WAITING\_DRIVER.

### 5.2 Modo AUTO_PROXIMITY activo
El parámetro global de asignación es AUTO\_PROXIMITY (RN-08).

---

## 6. Poscondiciones

### 6.1 Pedido en ACCEPTED
El pedido queda en estado ACCEPTED con el `driverId`, `assigned_at` y `accepted_at` registrados.

### 6.2 Repartidor notificado y comprometido
El Repartidor que aceptó ha sido notificado y el pedido figura entre sus órdenes activas.

---

## 7. Puntos de Extensión

Ninguno en este caso de uso.

---

## 8. Requerimientos Especiales

8.1 La selección del candidato y la notificación deben ejecutarse sin intervención humana y completarse en un tiempo mínimo tras la llegada del pedido a WAITING\_DRIVER.

---

## 9. Información Adicional

El modelo de asignación es determinístico: el Sistema designa a un candidato específico (→ DRIVER\_ASSIGNED) y espera su aceptación explícita (→ ACCEPTED), en lugar de un broadcast abierto en que gana el primero que responde. Esto es consistente con la descripción del actor "Sistema de Asignación" en DS-ACT-01.

---

---

# Especificación del Caso de Uso del Sistema: CUS-05. Asignar Repartidor Manualmente

---

## 1. Actores del Sistema

### 1.1 Administrador
Actor primario. Selecciona y asigna manualmente al Repartidor que atenderá el pedido.

---

## 2. Propósito

Permitir al Administrador asignar directamente un Repartidor específico a un pedido en espera cuando el modo de asignación activo es ADMIN\_MANUAL.

---

## 3. Breve Descripción

El Administrador identifica un pedido en estado WAITING\_DRIVER, consulta la lista de Repartidores disponibles y selecciona al que considera más adecuado. El sistema valida la disponibilidad del Repartidor seleccionado, registra la asignación y lo notifica. El caso de uso termina cuando el Repartidor ha sido asignado y notificado. El flujo posterior (aceptación y entrega) continúa en CUS-06.

---

## 4. Flujo de Eventos

### 4.1 Flujo Básico

4.1.1 El Administrador accede al panel de administración y visualiza la lista de pedidos en estado WAITING\_DRIVER.

4.1.2 El Administrador selecciona el pedido que desea asignar.

4.1.3 El Administrador consulta la lista de Repartidores disponibles para ese pedido (online, dentro de su límite de simultáneas, región coincidente — RN-06).

4.1.4 El Administrador selecciona al Repartidor deseado y confirma la asignación.

4.1.5 El sistema valida en tiempo real que el Repartidor seleccionado aún cumpla las condiciones de disponibilidad (RN-06).

4.1.6 El sistema registra la asignación, transiciona el pedido a DRIVER\_ASSIGNED y registra `assigned_at`.

4.1.7 El sistema notifica al Repartidor asignado vía FCM con los detalles del pedido.

> **Supuesto a confirmar (ver RN-08):** se asume que el flujo de estados tras DRIVER\_ASSIGNED (aceptación del Repartidor → ACCEPTED, y los estados subsiguientes de entrega) es idéntico al del modo AUTO\_PROXIMITY. Confirmar con el equipo técnico si el Administrador tiene la opción de saltar directamente a ACCEPTED al asignar manualmente.

### 4.2 Subflujos

Ninguno.

### 4.3 Flujos Alternos

**FA-01: El Repartidor ya no está disponible al confirmar**

En el paso 4.1.5, si el Repartidor seleccionado ya no cumple las condiciones de disponibilidad (se puso offline, alcanzó su límite o su región cambió), el sistema rechaza la asignación e informa al Administrador para que seleccione otro candidato.

---

## 5. Precondiciones

### 5.1 Pedido en WAITING_DRIVER
El pedido existe y su estado actual es WAITING\_DRIVER.

### 5.2 Modo ADMIN_MANUAL activo
El parámetro global de asignación es ADMIN\_MANUAL (RN-08).

### 5.3 Administrador autenticado
El Administrador está autenticado y tiene acceso al panel de administración.

---

## 6. Poscondiciones

### 6.1 Pedido en DRIVER_ASSIGNED
El pedido queda en estado DRIVER\_ASSIGNED con el `driverId` y `assigned_at` registrados.

### 6.2 Repartidor notificado
El Repartidor asignado ha recibido la notificación del pedido.

---

## 7. Puntos de Extensión

Ninguno en este caso de uso.

---

## 8. Requerimientos Especiales

8.1 La lista de Repartidores disponibles presentada al Administrador debe reflejar el estado en tiempo real de disponibilidad de cada Repartidor.

---

## 9. Información Adicional

Este caso de uso es la alternativa manual a CUS-04. El Administrador puede cambiar el modo de asignación global entre AUTO\_PROXIMITY y ADMIN\_MANUAL desde la configuración del sistema (RN-08).

---

---

# Especificación del Caso de Uso del Sistema: CUS-06. Ejecutar Entrega

---

## 1. Actores del Sistema

### 1.1 Repartidor
Actor primario. Acepta el pedido asignado y registra cada hito del proceso de entrega.

---

## 2. Propósito

Permitir al Repartidor aceptar el pedido asignado y avanzar el pedido a través de los estados operativos de entrega hasta su conclusión, desencadenando el registro de comisiones al completarse.

---

## 3. Breve Descripción

El Repartidor recibe la notificación de que le ha sido asignado un pedido y decide aceptarlo. A partir de ese momento, registra cada hito de la entrega: llegada al local de la tienda, recogida del pedido, llegada al destino y confirmación de entrega. El sistema transiciona el pedido en cada paso, notifica a los actores correspondientes y, al alcanzar DELIVERED, calcula y registra las comisiones de plataforma y Repartidor en el ledger financiero. El caso de uso termina cuando el pedido queda en estado DELIVERED y las comisiones han sido registradas.

---

## 4. Flujo de Eventos

### 4.1 Flujo Básico

4.1.1 El Repartidor recibe la notificación de pedido asignado vía FCM y accede a sus detalles (tienda, dirección de entrega, ítems).

4.1.2 El Repartidor revisa los detalles del pedido y decide aceptarlo.

4.1.3 El Repartidor confirma la aceptación. El sistema transiciona el pedido a ACCEPTED y registra `accepted_at`.

4.1.4 El sistema notifica al Cliente vía FCM que el Repartidor está en camino.

4.1.5 El Repartidor llega al local de la Tienda / Dispatcher y registra su llegada en la aplicación. El sistema transiciona el pedido a AT\_STORE y registra `arrived_at`.

4.1.6 El sistema notifica a la Tienda / Dispatcher vía FCM que el Repartidor se encuentra en el local.

4.1.7 El Repartidor recoge el pedido y registra la recogida. El sistema transiciona el pedido a PICKED\_UP y registra `picked_up_at`.

4.1.8 El sistema notifica al Cliente vía FCM que el pedido ha sido recogido y está en camino.

4.1.9 El Repartidor llega a la dirección de entrega del Cliente y registra su llegada. El sistema transiciona el pedido a AT\_CUSTOMER y registra `at_customer_at`.

4.1.10 El sistema notifica al Cliente vía FCM que el Repartidor ha llegado.

4.1.11 El Repartidor entrega el pedido al Cliente y registra la entrega. El sistema transiciona el pedido a DELIVERED y registra `delivered_at`.

4.1.12 El sistema ejecuta el registro de comisiones (SF-01).

4.1.13 El sistema notifica al Cliente vía FCM que su pedido fue entregado y solicita una calificación del servicio.

4.1.14 El sistema notifica a la Tienda / Dispatcher vía Telegram que el pedido fue entregado.

### 4.2 Subflujos

**SF-01: Registro de comisiones al entregar**

4.2.1 El sistema determina el tipo de tienda (`store_type`: OFFICIAL\_10, OFFICIAL\_5 o NON\_OFFICIAL).

4.2.2 El sistema calcula la comisión de plataforma sobre el subtotal según el tipo de tienda (RN-15):
- OFFICIAL\_10: `comisión = subtotal × 10%`.
- OFFICIAL\_5: `base = subtotal / 1.05`; `comisión = base × 10%`.
- NON\_OFFICIAL: `base = subtotal / 1.10`; `comisión = subtotal − base`.

4.2.3 Si el tipo es OFFICIAL\_10 y el método de pago es CARD, el sistema registra adicionalmente `store_revenue = subtotal` en el ledger. En pagos CASH no se genera esa entrada (RN-15).

4.2.4 El sistema resuelve la tasa de comisión del Repartidor siguiendo la jerarquía: override individual → config por ciudad → config global → fallback en Settings (RN-16).

4.2.5 El sistema calcula la comisión del Repartidor: `comisión_driver = delivery_fee × (commission_rate / 100)` (RN-17).

4.2.6 El sistema registra todas las entradas correspondientes en `financial_ledger`.

### 4.3 Flujos Alternos

**FA-01: El Repartidor no acepta el pedido dentro del tiempo de espera**

En el paso 4.1.3, si el Repartidor no acepta dentro del tiempo establecido, el Sistema de Asignación reintenta la asignación con el siguiente candidato disponible (ver CUS-04, FA-02). El presente caso de uso no se activa para este Repartidor.

---

## 5. Precondiciones

### 5.1 Pedido en DRIVER_ASSIGNED
El pedido existe y su estado actual es DRIVER\_ASSIGNED, con el `driverId` del Repartidor registrado.

### 5.2 Repartidor autenticado y disponible
El Repartidor está autenticado y cumple las condiciones de disponibilidad al momento de aceptar (RN-06).

---

## 6. Poscondiciones

### 6.1 Pedido en DELIVERED
El pedido queda en estado DELIVERED con `delivered_at` y todos los timestamps de hito registrados (`accepted_at`, `arrived_at`, `picked_up_at`, `at_customer_at`, `delivered_at`).

### 6.2 Comisiones registradas
Las comisiones de plataforma (sobre la tienda) y del Repartidor han sido calculadas y registradas en `financial_ledger` (RN-15, RN-16, RN-17).

### 6.3 Actores notificados
El Cliente, la Tienda / Dispatcher y el Repartidor han recibido sus respectivas notificaciones de conclusión de la entrega.

---

## 7. Puntos de Extensión

Ninguno en este caso de uso.

---

## 8. Requerimientos Especiales

8.1 Las notificaciones en cada transición de estado deben enviarse de forma inmediata para mantener informados a todos los actores en tiempo real.

8.2 El registro de comisiones (SF-01) debe ejecutarse de forma atómica con la transición a DELIVERED para garantizar la consistencia del ledger financiero.

---

## 9. Información Adicional

El cálculo de comisiones se rige por las Reglas del Negocio RN-15, RN-16 y RN-17. Los tipos de tienda (OFFICIAL\_10, OFFICIAL\_5, NON\_OFFICIAL) son atributos configurados en cada tienda por el Administrador.

---

---

# Especificación del Caso de Uso del Sistema: CUS-07. Cancelar Pedido

---

## 1. Actores del Sistema

### 1.1 Cliente
Actor primario para cancelación desde estado PENDING únicamente.

### 1.2 Administrador
Actor primario para cancelación desde cualquier estado activo posterior a PENDING.

### 1.3 Soporte
Actor primario para cancelación desde cualquier estado activo posterior a PENDING.

---

## 2. Propósito

Registrar la cancelación de un pedido en curso, aplicando el tipo de cancelación correspondiente según el actor que la ejecuta y el estado actual del pedido, y notificar a todos los actores involucrados.

---

## 3. Breve Descripción

Un actor autorizado solicita la cancelación de un pedido activo. El sistema valida que el actor tenga permiso para cancelar según el estado actual del pedido (RN-04), registra el tipo de cancelación seleccionado y aplica las consecuencias financieras definidas en RN-05. Se notifica a todos los actores involucrados en el pedido. El caso de uso termina cuando el pedido queda en estado CANCELLED con el tipo y motivo de cancelación registrados.

---

## 4. Flujo de Eventos

### 4.1 Flujo Básico — Cancelación por el Cliente (estado PENDING)

4.1.1 El Cliente accede al detalle de su pedido en estado PENDING.

4.1.2 El Cliente solicita la cancelación e indica el motivo.

4.1.3 El sistema valida que el pedido se encuentre en estado PENDING y que el solicitante sea el Cliente propietario del pedido (RN-04).

4.1.4 El sistema registra `cancellation_type = CLIENT_FAULT` y el motivo en `cancellation_reason`.

4.1.5 El sistema transiciona el pedido a CANCELLED.

4.1.6 El sistema aplica las consecuencias financieras correspondientes al tipo CLIENT\_FAULT: se registra una penalidad económica configurable en la cuenta del Cliente (RN-05). El mecanismo de cálculo y cobro de dicha penalidad está fuera del alcance de este documento.

4.1.7 El sistema notifica a la Tienda / Dispatcher vía FCM que el pedido fue cancelado.

4.1.8 El sistema confirma al Cliente la cancelación del pedido.

### 4.2 Subflujos

**SF-01: Cancelación por Administrador o Soporte (estado CONFIRMED o posterior)**

4.2.1 El Administrador o Soporte accede al panel correspondiente y localiza el pedido a cancelar.

4.2.2 El actor selecciona el tipo de cancelación (CLIENT\_FAULT, STORE\_FAULT\_BEFORE\_DRIVER\_PAID, STORE\_FAULT\_AFTER\_DRIVER\_PAID o DRIVER\_FAULT) e ingresa el motivo de la cancelación.

4.2.3 El sistema valida que el actor tenga el rol requerido (ADMIN o SUPPORT) y que el pedido no se encuentre en estado DELIVERED ni CANCELLED (RN-04).

4.2.4 El sistema registra el tipo de cancelación en `cancellation_type` y el motivo en `cancellation_reason`.

4.2.5 El sistema transiciona el pedido a CANCELLED.

4.2.6 El sistema aplica las consecuencias financieras según el tipo registrado (RN-05):
- CLIENT\_FAULT: penalidad económica configurable al Cliente; comisiones de tienda y Repartidor se registran.
- STORE\_FAULT\_BEFORE\_DRIVER\_PAID: anulación total; sin entradas en el ledger.
- STORE\_FAULT\_AFTER\_DRIVER\_PAID: el Repartidor conserva la tarifa de envío; la tienda no genera comisión.
- DRIVER\_FAULT: comisiones se registran con normalidad; el Repartidor pierde la entrega.

4.2.7 El sistema notifica a todos los actores involucrados en el pedido: Cliente vía FCM, Tienda / Dispatcher vía FCM, y Repartidor vía FCM si ya había sido asignado.

### 4.3 Flujos Alternos

**FA-01: El Cliente intenta cancelar un pedido en estado CONFIRMED o posterior**

En el paso 4.1.3, si el pedido ya no está en PENDING, el sistema rechaza la acción del Cliente e informa que para cancelar un pedido ya confirmado debe contactar a Soporte.

**FA-02: El pedido ya está en DELIVERED o CANCELLED**

En cualquier paso de validación, si el pedido se encuentra en DELIVERED o CANCELLED, el sistema rechaza la acción e informa que el pedido no admite más cambios de estado (RN-02).

---

## 5. Precondiciones

### 5.1 Pedido activo
El pedido existe y su estado no es DELIVERED ni CANCELLED.

### 5.2 Actor autorizado según el estado
El actor solicitante tiene permiso para ejecutar la cancelación en el estado actual del pedido: el Cliente solo puede cancelar en PENDING; el Administrador y Soporte pueden cancelar en cualquier estado activo posterior a PENDING (RN-04).

---

## 6. Poscondiciones

### 6.1 Pedido en CANCELLED
El pedido queda en estado CANCELLED con `cancellation_type` y `cancellation_reason` registrados.

### 6.2 Consecuencias financieras aplicadas
Las consecuencias financieras definidas en RN-05 para el tipo de cancelación registrado han sido aplicadas.

### 6.3 Actores notificados
Todos los actores involucrados en el pedido al momento de la cancelación han recibido su respectiva notificación.

---

## 7. Puntos de Extensión

Ninguno en este caso de uso.

---

## 8. Requerimientos Especiales

8.1 El sistema debe validar el estado del pedido y los permisos del actor en tiempo real antes de ejecutar la cancelación, para evitar condiciones de carrera en pedidos con múltiples actores activos simultáneamente.

---

## 9. Información Adicional

La distinción entre "cancelación" (acción del Cliente) y "rechazo" (acción de la Tienda / Dispatcher) está definida en RN-04 y en CUS-03. Ambas producen el estado CANCELLED pero son acciones distintas con actores y tipos diferentes. Las consecuencias financieras de cada tipo de cancelación están detalladas en RN-05 del documento DS-RN-01.

---

---

# Especificación del Caso de Uso del Sistema: CUS-08. Consultar Estado / Seguimiento del Pedido

---

## 1. Actores del Sistema

### 1.1 Cliente
Actor primario. Consulta el estado actual de su pedido en cualquier momento durante el ciclo de vida activo.

---

## 2. Propósito

Permitir al Cliente conocer en tiempo real el estado de su pedido y, cuando un Repartidor ha sido asignado, visualizar los hitos de la entrega conforme avanza el ciclo.

---

## 3. Breve Descripción

El Cliente accede al detalle de un pedido activo para ver su estado actual y la información del Repartidor asignado si ya existe. El sistema muestra el estado vigente y mantiene la vista actualizada mediante notificaciones push (FCM) ante cada cambio de estado. El caso de uso puede iniciarse en cualquier momento mientras el pedido no esté en DELIVERED ni CANCELLED, y también permite revisar pedidos ya concluidos en el historial.

---

## 4. Flujo de Eventos

### 4.1 Flujo Básico

4.1.1 El Cliente accede a la sección de pedidos activos o historial de la aplicación.

4.1.2 El Cliente selecciona el pedido que desea consultar.

4.1.3 El sistema recupera el estado actual del pedido y los datos asociados (tienda, ítems, tarifa, dirección de entrega, timestamps de hito registrados).

4.1.4 Si el pedido tiene un Repartidor asignado (estado ACCEPTED o posterior), el sistema muestra los datos del Repartidor (nombre, foto, rango) y el estado actual de la entrega.

4.1.5 El sistema presenta la vista de detalle al Cliente con el estado vigente.

4.1.6 Mientras el pedido permanezca activo, el sistema actualiza automáticamente la vista del Cliente ante cada cambio de estado mediante notificaciones FCM recibidas en segundo plano.

### 4.2 Subflujos

Ninguno.

### 4.3 Flujos Alternos

**FA-01: El pedido no tiene Repartidor asignado aún**

En el paso 4.1.4, si el pedido se encuentra en PENDING, CONFIRMED o WAITING\_DRIVER, el sistema omite la sección de Repartidor e informa al Cliente que su pedido está en proceso de preparación o búsqueda de repartidor según corresponda.

---

## 5. Precondiciones

### 5.1 Cliente autenticado
El Cliente está autenticado en el sistema.

### 5.2 Pedido existente y propio
El pedido consultado existe y pertenece al Cliente solicitante.

---

## 6. Poscondiciones

### 6.1 Sin cambio de estado
Este caso de uso es de solo lectura; no produce ninguna transición de estado en el pedido ni en ninguna otra entidad del sistema.

---

## 7. Puntos de Extensión

Ninguno en este caso de uso.

---

## 8. Requerimientos Especiales

8.1 La vista de estado debe reflejar el estado real del pedido en el momento de la consulta; los cambios posteriores se comunican mediante notificaciones FCM sin requerir acción adicional del Cliente.

---

## 9. Información Adicional

Las actualizaciones automáticas de estado (CONFIRMED, ACCEPTED, PICKED\_UP, AT\_CUSTOMER, DELIVERED) son iniciadas por las notificaciones FCM que el sistema emite en cada transición, descritas en los flujos básicos de CUS-02, CUS-04/05 y CUS-06.

---

---

# Especificación del Caso de Uso del Sistema: CUS-09. Calificar Pedido

---

## 1. Actores del Sistema

### 1.1 Cliente
Actor primario. Emite una calificación del servicio recibido tras la entrega del pedido.

---

## 2. Propósito

Permitir al Cliente registrar una calificación del servicio de entrega una vez que su pedido ha sido marcado como DELIVERED, contribuyendo a la evaluación del desempeño del sistema y los actores involucrados.

---

## 3. Breve Descripción

Tras recibir la notificación de que su pedido fue entregado, el sistema presenta al Cliente un prompt de calificación. El Cliente puede emitir una puntuación y, opcionalmente, un comentario. El caso de uso termina cuando la calificación queda registrada o el Cliente decide omitirla.

---

## 4. Flujo de Eventos

### 4.1 Flujo Básico

4.1.1 El sistema notifica al Cliente vía FCM que su pedido ha sido entregado (estado DELIVERED) e incluye un prompt para calificar el servicio (paso 4.1.13 de CUS-06).

4.1.2 El Cliente accede al prompt de calificación desde la notificación o desde el detalle del pedido.

4.1.3 El Cliente selecciona una puntuación para el servicio recibido.

4.1.4 El Cliente puede agregar opcionalmente un comentario de texto libre.

4.1.5 El Cliente envía la calificación.

4.1.6 El sistema registra la calificación asociada al pedido y actualiza los indicadores de desempeño correspondientes.

4.1.7 El sistema confirma al Cliente que su calificación fue registrada.

### 4.2 Subflujos

Ninguno.

### 4.3 Flujos Alternos

**FA-01: El Cliente omite la calificación**

En el paso 4.1.2 o 4.1.5, si el Cliente cierra el prompt sin enviar una calificación, el sistema registra la omisión sin generar error. El prompt puede volver a presentarse al Cliente la próxima vez que acceda al detalle del pedido, mientras la calificación no haya sido registrada.

---

## 5. Precondiciones

### 5.1 Pedido en DELIVERED
El pedido existe, pertenece al Cliente y su estado actual es DELIVERED.

### 5.2 Calificación no registrada previamente
El pedido no tiene aún una calificación registrada por parte del Cliente.

### 5.3 Cliente autenticado
El Cliente está autenticado en el sistema.

---

## 6. Poscondiciones

### 6.1 Calificación registrada
La puntuación (y comentario opcional) quedan registrados y asociados al pedido correspondiente.

---

## 7. Puntos de Extensión

Ninguno en este caso de uso.

---

## 8. Requerimientos Especiales

8.1 La calificación debe poder registrarse solo una vez por pedido por parte del Cliente; intentos posteriores deben ser rechazados con información al respecto.

---

## 9. Información Adicional

El prompt de calificación es iniciado por el sistema como parte del cierre del ciclo de pedido en CUS-06 (paso 4.1.13). La calificación es opcional; su omisión no bloquea ningún flujo posterior del Cliente.
