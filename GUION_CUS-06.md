# Guion de Video — CUS-06: Ejecutar Entrega

> Nota de producción: este caso de uso cuenta con grabación de pantalla completa. El guion puede incluir referencias directas a lo que se muestra en el video.

---

Llegamos a "Ejecutar Entrega", el corazón operativo de todo el proceso: es el caso de uso donde el repartidor efectivamente lleva el pedido desde la tienda hasta el cliente, y donde vamos a poder seguir, paso a paso, como se muestra ahora en pantalla, todo el recorrido de la aplicación del repartidor.

El actor principal aquí es el Repartidor, quien es el encargado de aceptar el pedido asignado y de registrar cada hito del proceso de entrega a medida que va ocurriendo.

Como pueden ver en la pantalla, todo comienza cuando el repartidor recibe una notificación de que se le ha asignado un pedido, con los detalles: la tienda, la dirección de entrega y los ítems solicitados. El repartidor revisa esa información y confirma que acepta el pedido; en ese momento el sistema lo marca como aceptado y, como vemos aquí, se le avisa al cliente que su repartidor ya va en camino.

A partir de ahí, el repartidor va registrando cada etapa de su recorrido directamente desde la app, que es justo lo que estamos viendo ahora. Primero, cuando llega al local de la tienda, presiona el botón correspondiente y el pedido cambia al estado de "en tienda". Segundo, cuando recoge el pedido, lo marca como recogido, y en ese instante el sistema le avisa al cliente que su pedido va en camino. Tercero, cuando el repartidor llega a la dirección del cliente, registra su llegada, y el cliente recibe la notificación de que el repartidor ya está en su ubicación. Y finalmente, cuando hace la entrega en mano, la marca como completada, lo que ustedes pueden ver reflejado aquí en la pantalla con el cambio de estado a "entregado".

Ese último paso no es solo un cambio de estado: en ese mismo instante, el sistema calcula y registra automáticamente las comisiones de la plataforma y del repartidor, según el tipo de tienda y la tarifa de envío, y guarda ese registro en el libro financiero del sistema. Además, se dispara una notificación al cliente pidiéndole que califique el servicio, y otra a la tienda confirmando que el pedido ya fue entregado.

¿Y si el repartidor no llega a aceptar el pedido a tiempo? En ese caso, como vimos en el caso de uso anterior, el sistema simplemente lo reintenta con otro candidato, y este caso de uso nunca llega a activarse para ese repartidor en particular.

Con esto, el pedido queda finalmente en estado entregado, con todos los tiempos de cada hito registrados, las comisiones calculadas correctamente, y los tres actores involucrados —cliente, tienda y repartidor— debidamente notificados. Así concluye el ciclo completo de la entrega.

[Duración estimada: 2 min 55 seg]
