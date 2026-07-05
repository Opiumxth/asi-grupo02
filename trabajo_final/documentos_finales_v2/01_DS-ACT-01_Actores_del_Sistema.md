# DeliverySuite

# Proyecto: DeliverySuite

# Actores del Sistema

## Versión 1.0

Identificador del documento: DS-ACT-01

---

# Historial de Revisiones

| Fecha | Versión | Descripción | Autor |
|---|---|---|---|
| 28/Jun/26 | 1.0 | Primera versión — identificación de actores del ciclo completo de pedido | Flores Hoyos, Mathias |
| 28/Jun/26 | 1.0 | Primera versión — identificación de actores del ciclo completo de pedido | Príncipe Caballa, Aida |
| 28/Jun/26 | 1.0 | Primera versión — identificación de actores del ciclo completo de pedido | Ochoa Torres, Yoant Alnor |

---

# Tabla de Contenidos

## 1. Cliente
## 2. Tienda / Dispatcher
## 3. Repartidor
## 4. Administrador
## 5. Soporte
## 6. Sistema de Asignación

---

# Actores del Sistema

El presente documento identifica y describe a los actores del sistema DeliverySuite, entendidos como los roles que interactúan directamente con el sistema para obtener un resultado de valor observable. Cada actor puede corresponder a una persona, a un grupo de personas que desempeñan la misma función frente al sistema, o a un componente automatizado que actúa en representación del negocio. La identificación de estos actores constituye la base sobre la cual se construyen posteriormente las Reglas del Negocio y las Especificaciones de Caso de Uso del Sistema.

## 1. Cliente

El Cliente es la persona que utiliza la plataforma para realizar pedidos de productos ofrecidos por las tiendas afiliadas a DeliverySuite. Es el actor que inicia el ciclo de vida de un pedido: selecciona los productos deseados, proporciona la dirección de entrega, elige el método de pago y confirma la orden. Durante el desarrollo del proceso, el Cliente puede consultar el estado de su pedido en tiempo real, obteniendo visibilidad sobre cada hito relevante de la entrega. Asimismo, conserva la potestad de cancelar el pedido mientras este se encuentre en el estado PENDING, es decir, antes de que la tienda inicie su preparación. Una vez concluida la entrega, el Cliente tiene la posibilidad de calificar el servicio recibido, aportando información que contribuye a la evaluación del desempeño del sistema y de los demás actores involucrados.

---

## 2. Tienda / Dispatcher

La Tienda / Dispatcher es el establecimiento afiliado a la plataforma que ofrece productos para su entrega a domicilio. Dentro del sistema Musuq V2, del cual DeliverySuite toma como referencia su modelo de datos y reglas operativas, este rol corresponde al rol interno `VENDOR`. La Tienda / Dispatcher recibe notificaciones de las nuevas órdenes generadas por los Clientes, revisa su contenido y decide si confirmarlas —indicando en ese momento la sucursal de despacho correspondiente y el tiempo estimado de preparación— o si rechazarlas cuando no pueda atenderlas. Además de esta función central, la Tienda / Dispatcher es responsable de gestionar su catálogo de productos, sus horarios de atención y la información general de su local. La confirmación emitida por este actor constituye el evento que desencadena la búsqueda de un repartidor disponible para atender el pedido. Cabe señalar que la Tienda / Dispatcher puede ser bloqueada por el sistema en caso de mantener una deuda financiera pendiente frente a la plataforma.

---

## 3. Repartidor

El Repartidor es la persona encargada de recoger el pedido en la tienda correspondiente y entregarlo en la dirección indicada por el Cliente. Este actor corresponde, dentro de Musuq V2, al rol interno `DRIVER`. El Repartidor recibe notificaciones de los pedidos disponibles en su región geográfica de operación, acepta la asignación que se le propone y, a partir de ese momento, avanza el pedido a través de una secuencia de estados operativos claramente definida: AT_STORE, que indica su llegada al local de la tienda; PICKED_UP, que indica la recogida del pedido; AT_CUSTOMER, que indica su llegada al destino de entrega; y DELIVERED, que indica la conclusión de la entrega. El Repartidor mantiene de forma activa su estado de disponibilidad, pudiendo alternar entre las condiciones de conectado (online) y desconectado (offline), y tiene la posibilidad de gestionar simultáneamente varios pedidos según el rango que ostente dentro del sistema de clasificación de repartidores. Por cada pedido entregado, su comisión se calcula sobre la tarifa de envío correspondiente a dicho pedido.

---

## 4. Administrador

El Administrador es el usuario interno que cuenta con acceso total al sistema y que se encarga de supervisar y configurar la operación integral de la plataforma. Este actor corresponde al rol interno `ADMIN` dentro de Musuq V2. Sus responsabilidades, dentro del alcance acordado para DeliverySuite, comprenden la configuración del modo de asignación de repartidores, ya sea en su modalidad automática por proximidad (AUTO_PROXIMITY) o en su modalidad manual (ADMIN_MANUAL); la asignación manual de un repartidor a un pedido específico cuando el modo manual se encuentra activo; la gestión integral de tiendas y repartidores, incluyendo el control de su estado financiero; la cancelación de pedidos en cualquier estado del ciclo de vida; y el ajuste de las reglas que determinan la tarifa de envío y las comisiones aplicables según el tipo de tienda. El Administrador accede al sistema a través del panel web de administración dispuesto para tal fin.

---

## 5. Soporte

El actor Soporte representa al personal interno encargado de la atención de incidencias operativas que surgen durante el ciclo de vida de los pedidos. Corresponde al rol interno `SUPPORT` dentro de Musuq V2. Este actor interviene en situaciones excepcionales, tales como cancelaciones que revisten alguna complejidad particular, disputas suscitadas entre el Cliente y la Tienda, o pedidos que permanecen inmovilizados en un determinado estado por un periodo anómalo. Entre sus atribuciones se encuentra la de cancelar pedidos que ya no se encuentren en estado PENDING, registrando en cada caso el tipo de cancelación que corresponda —CLIENT_FAULT, STORE_FAULT o DRIVER_FAULT— de modo que el sistema pueda aplicar el tratamiento financiero adecuado a cada situación. Para desempeñar adecuadamente su función, el actor Soporte requiere visibilidad en tiempo real sobre el estado de los pedidos activos dentro del sistema.

---

## 6. Sistema de Asignación

El Sistema de Asignación es un actor secundario de naturaleza automatizada que representa al componente `AssignmentService` del backend de DeliverySuite. A diferencia de los actores descritos anteriormente, no corresponde a un usuario humano, sino a un proceso que se activa automáticamente cuando un pedido alcanza el estado CONFIRMED y el modo de asignación vigente en el sistema es AUTO_PROXIMITY. Este actor opera conforme a un modelo de asignación determinístico organizado en dos fases sucesivas. En la primera fase, selecciona a un candidato específico entre los repartidores disponibles —entendiéndose por tales aquellos que se encuentran conectados, no ocupados y ubicados dentro de la región geográfica correspondiente al pedido— y transiciona el pedido al estado DRIVER_ASSIGNED, notificando de inmediato a dicho candidato. En la segunda fase, el candidato designado debe aceptar explícitamente la orden para que el pedido pueda avanzar al estado ACCEPTED. Si el candidato seleccionado no manifiesta su aceptación dentro del tiempo de espera establecido, el Sistema de Asignación reinicia el proceso, seleccionando al siguiente candidato disponible según el mismo criterio. El modo de asignación activo, sea este AUTO_PROXIMITY o ADMIN_MANUAL, es un parámetro configurable exclusivamente por el Administrador.
