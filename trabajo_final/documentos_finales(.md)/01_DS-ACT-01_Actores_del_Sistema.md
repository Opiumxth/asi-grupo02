# DeliverySuite

# Proyecto: DeliverySuite

# Actores del Sistema

## Versión 1.0

---

**Historial de Revisiones**

| Fecha | Versión | Descripción | Autor |
|---|---|---|---|
| 28/Jun/26 | 1.0 | Primera versión — identificación de actores del ciclo completo de pedido | Flores Hoyos, Mathias |
| 28/Jun/26 | 1.0 | Primera versión — identificación de actores del ciclo completo de pedido | Príncipe Caballa, Aida |
| 28/Jun/26 | 1.0 | Primera versión — identificación de actores del ciclo completo de pedido | Ochoa Torres, Yoant Alnor |

---

## Tabla de Contenidos

## 1. Cliente
## 2. Tienda / Dispatcher
## 3. Repartidor
## 4. Administrador
## 5. Soporte
## 6. Sistema de Asignación

---

## Actores del Sistema

### 1. Cliente

Persona que utiliza la plataforma para realizar pedidos de productos a las tiendas afiliadas. Es el actor que inicia el ciclo de vida de un pedido: selecciona productos, proporciona la dirección de entrega, elige el método de pago y confirma la orden. Durante el proceso puede consultar el estado de su pedido en tiempo real. Tiene la capacidad de cancelar un pedido mientras se encuentra en estado PENDING. Al concluir la entrega, puede calificar el servicio recibido.

---

### 2. Tienda / Dispatcher

Establecimiento afiliado a la plataforma que ofrece productos para entrega a domicilio. En Musuq V2 este rol corresponde al rol interno `VENDOR`. Recibe notificaciones de nuevas órdenes, las revisa y decide confirmarlas (indicando la sucursal de despacho y el tiempo estimado de preparación) o rechazarlas. Gestiona su catálogo de productos, horarios de atención y la información de su local. La confirmación de la Tienda / Dispatcher es el evento que desencadena la búsqueda de repartidor. También puede ser bloqueada por el sistema ante deuda financiera pendiente.

---

### 3. Repartidor

Persona encargada de recoger el pedido en la tienda y entregarlo en la dirección indicada por el cliente. En Musuq V2 este rol corresponde al rol interno `DRIVER`. Recibe notificaciones de pedidos disponibles en su región, acepta la asignación y avanza el pedido a través de los estados operativos: AT\_STORE (llegó al local), PICKED\_UP (recogió el pedido), AT\_CUSTOMER (llegó al destino) y DELIVERED (entregado). Mantiene activo su estado de disponibilidad (online/offline) y puede gestionar múltiples pedidos simultáneos según su rango. Su comisión se calcula sobre la tarifa de envío de cada pedido entregado.

---

### 4. Administrador

Usuario interno con acceso total al sistema encargado de supervisar y configurar la operación. En Musuq V2 este rol corresponde al rol interno `ADMIN`. Sus responsabilidades dentro del alcance acordado incluyen: configurar el modo de asignación de repartidores (AUTO\_PROXIMITY o ADMIN\_MANUAL), asignar manualmente un repartidor a un pedido cuando el modo manual está activo, gestionar tiendas y repartidores (incluyendo su estado financiero), cancelar pedidos en cualquier estado y ajustar las reglas de tarifa de envío y comisiones por tipo de tienda. Accede al sistema a través del panel web de administración.

---

### 5. Soporte

Personal interno encargado de la atención de incidencias operativas. En Musuq V2 este rol corresponde al rol interno `SUPPORT`. Interviene cuando se presentan situaciones excepcionales dentro del ciclo de un pedido, como cancelaciones problemáticas, disputas entre cliente y tienda, o pedidos atascados en un estado. Puede cancelar pedidos que ya no se encuentren en estado PENDING y registrar el tipo de cancelación correspondiente (CLIENT\_FAULT, STORE\_FAULT, DRIVER\_FAULT) para que el sistema aplique el tratamiento financiero correcto. Requiere visibilidad en tiempo real del estado de los pedidos activos.

---

### 6. Sistema de Asignación

Actor secundario automatizado que representa el componente `AssignmentService` del backend de DeliverySuite. No es un usuario humano; es activado automáticamente cuando un pedido alcanza el estado CONFIRMED y el modo de asignación activo es AUTO\_PROXIMITY. Opera siguiendo un modelo de asignación determinística en dos fases: primero selecciona un candidato específico entre los repartidores disponibles (online, no ocupado, dentro de la región geográfica del pedido) y transiciona el pedido al estado DRIVER\_ASSIGNED notificando a ese candidato; a continuación, el candidato designado debe aceptar explícitamente la orden para que el pedido avance al estado ACCEPTED. Si el candidato no acepta dentro del tiempo de espera, el Sistema de Asignación reintenta el proceso seleccionando al siguiente candidato disponible. El modo activo (AUTO\_PROXIMITY o ADMIN\_MANUAL) es configurable por el Administrador.
