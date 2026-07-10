# Guion de Video — CUS-05: Asignar Repartidor Manualmente

> Nota de producción: este caso de uso sí cuenta con grabación de pantalla. **El flujo narrado a continuación corrige la lógica del documento original**: el modo manual no consiste en que el Administrador asigne al repartidor, sino en que son los propios Repartidores quienes ven los pedidos disponibles y se auto-asignan aquellos que pueden atender.

---

Ahora vamos a ver "Asignar Repartidor Manualmente", que es la alternativa al modo automático que vimos antes. Este caso de uso ocurre en el mismo punto del ciclo del pedido: justo cuando un pedido ya fue confirmado por la tienda y está esperando a que alguien lo recoja. La diferencia está en quién decide y cómo se decide.

Aquí el actor principal es el Repartidor. Cuando el sistema tiene activado el modo manual en lugar del modo automático, no es el sistema el que busca y designa a alguien: son los propios repartidores los que, desde su aplicación, ven la lista de pedidos disponibles para tomar y eligen ellos mismos cuáles creen que pueden llevar.

Veamos cómo se da el flujo. Con el modo manual activo, un repartidor conectado abre su lista de pedidos disponibles, la que vamos a mostrar ahora en pantalla. Ahí aparecen únicamente los pedidos que están esperando repartidor y que corresponden a su misma región geográfica. El repartidor revisa esa lista, evalúa qué pedido le conviene o cree que puede cumplir, y selecciona uno para auto-asignárselo. En ese momento, el sistema valida en tiempo real que el repartidor todavía cumpla las condiciones necesarias, es decir, que siga conectado y que no haya superado su límite de pedidos simultáneos. Si todo está en orden, el sistema registra la asignación, transiciona el pedido al estado de repartidor asignado y guarda la marca de tiempo correspondiente.

A partir de ahí, el flujo continúa exactamente igual que en el modo automático: el repartidor debe aceptar explícitamente el pedido para que este pase al estado de aceptado, y desde ese punto arranca todo el proceso de entrega que vamos a revisar en el siguiente caso de uso.

¿Y si algo falla en el camino? Puede pasar que, justo cuando el repartidor intenta tomar el pedido, otro repartidor se le haya adelantado, o que él mismo ya no cumpla las condiciones de disponibilidad, por ejemplo, si mientras tanto llegó a su límite de pedidos. En ese caso, el sistema rechaza la auto-asignación y el pedido sigue disponible en la lista para que otro repartidor lo tome.

Como resultado final, este caso de uso concluye con un pedido que quedó en estado de repartidor asignado, con el repartidor correcto vinculado y notificado, listo para iniciar la entrega. La diferencia clave frente al modo automático es que aquí la decisión de quién lleva cada pedido la toman los propios repartidores, y no un algoritmo del sistema.

[Duración estimada: 2 min 45 seg]
