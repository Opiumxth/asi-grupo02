# Guion de Video — CUS-04: Asignar Repartidor Automáticamente

> Nota de producción: este caso de uso **no cuenta con grabación de pantalla**, ya que la asignación la ejecuta el propio sistema sin intervención humana visible. El guion es puramente explicativo, narrado en off, sin referencias a "lo que se ve en pantalla".

---

Vamos a hablar de "Asignar Repartidor Automáticamente", uno de los pasos clave dentro del ciclo de vida de un pedido en DeliverySuite. Este caso de uso ocurre justo después de que un pedido ha sido confirmado por la tienda: el pedido queda en espera de repartidor, y aquí es donde el sistema entra a resolver ese problema por sí mismo.

Los actores que participan en este caso de uso son dos. El primero es el Sistema de Asignación, un actor automatizado que se activa solo, sin que nadie tenga que presionar un botón, en el momento en que el pedido llega al estado de espera de repartidor y el modo de asignación configurado es el automático por proximidad. El segundo actor es el Repartidor, quien participa como candidato: es él quien, al aceptar explícitamente el pedido, permite que este avance al siguiente estado.

¿Cómo funciona el flujo? Cuando un pedido queda esperando repartidor bajo este modo automático, el Sistema de Asignación revisa qué repartidores cumplen tres condiciones al mismo tiempo: que estén conectados, que no hayan superado su límite de pedidos simultáneos según su rango, y que se encuentren en la misma región geográfica que el pedido. De ese grupo, el sistema elige al candidato más adecuado y lo designa como responsable, notificándolo de inmediato a través de una notificación push con los detalles del pedido. El repartidor designado tiene que aceptar explícitamente; solo cuando lo hace, el pedido pasa al estado de aceptado, y con eso concluye este caso de uso.

Es importante mencionar que este modelo no es del tipo "el primero que responde se lo lleva". El sistema designa a un candidato específico y espera su respuesta, en lugar de ofrecer el pedido a todos a la vez.

Ahora bien, ¿qué pasa si algo no sale como se espera? Hay dos situaciones alternas. Si no hay ningún repartidor disponible en la región, el pedido se queda esperando y el sistema genera una alerta de pedido huérfano hacia el Administrador, para que intervenga manualmente. Y si el candidato designado no responde dentro del tiempo establecido, el sistema simplemente reintenta con el siguiente candidato disponible, sin que el pedido retroceda de estado; si se agotan todos los candidatos, se cae en la misma alerta de pedido huérfano.

En resumen, este caso de uso permite que la plataforma resuelva automáticamente y sin intervención humana quién va a llevar cada pedido, dejando como resultado final un pedido con un repartidor asignado, notificado y comprometido con la entrega, listo para pasar a la siguiente etapa del proceso.

[Duración estimada: 2 min 50 seg]
