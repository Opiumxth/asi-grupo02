# DeliverySuite

# Proyecto: DeliverySuite

# Documento de Arquitectura de Software — Modelo de Vistas 4+1

## Versión 1.0

Identificador del documento: DS-ARQ-01

---

# Historial de Revisiones

| Fecha | Versión | Descripción | Autor |
|---|---|---|---|
| 05/Jul/26 | 1.0 | Primera versión — documento de arquitectura de software bajo el modelo de vistas 4+1 | Flores Hoyos, Mathias |
| 05/Jul/26 | 1.0 | Primera versión — documento de arquitectura de software bajo el modelo de vistas 4+1 | Príncipe Caballa, Aida |
| 05/Jul/26 | 1.0 | Primera versión — documento de arquitectura de software bajo el modelo de vistas 4+1 | Ochoa Torres, Yoant Alnor |

---

# Tabla de Contenidos

- 1. Introducción
  - 1.1 Propósito
  - 1.2 Alcance
  - 1.3 Definiciones, Acrónimos y Abreviaturas
  - 1.4 Documentos Relacionados
- 2. Representación Arquitectónica
- 3. Vista de Casos de Uso (+1)
- 4. Vista Lógica
- 5. Vista de Procesos
- 6. Vista de Desarrollo
- 7. Vista Física
- 8. Restricciones Arquitectónicas y Elementos Fuera de Alcance

---

# 1. Introducción

## 1.1 Propósito

El presente documento tiene como propósito describir la arquitectura de software del sistema DeliverySuite mediante el modelo de vistas 4+1, propuesto por Philippe Kruchten dentro del marco del Rational Unified Process. Bajo este modelo, la arquitectura de un sistema se documenta desde cuatro perspectivas complementarias —lógica, de procesos, de desarrollo y física— las cuales se organizan y justifican en torno a una quinta perspectiva adicional, constituida por los escenarios de uso más significativos del sistema, que en el presente documento corresponden a los nueve casos de uso especificados en el documento DS-CUS-01-09. El objetivo de esta descripción es exponer las decisiones estructurales que sustentan la construcción de DeliverySuite, de modo que sirvan de guía para el equipo de desarrollo y de fundamento para la evaluación de la solución por parte del curso.

Es importante precisar que el presente documento constituye una descripción íntegramente textual de la arquitectura del sistema. No contiene diagramas de ningún tipo, ya sean diagramas UML, diagramas de bloques o esquemas gráficos equivalentes. Los diagramas UML correspondientes a cada una de las vistas aquí descritas —diagramas de casos de uso, de clases, de secuencia, de componentes y de despliegue— se entregan de manera independiente en el archivo del modelo Rational Rose `DeliverySuite.mdl`, el cual constituye el artefacto gráfico complementario y obligatorio de este documento. Toda referencia a elementos gráficos, distribución de cajas, flechas o disposición espacial de componentes debe entenderse remitida a dicho modelo, y no forma parte del contenido del presente texto.

## 1.2 Alcance

La arquitectura descrita en este documento cubre el sistema DeliverySuite en su totalidad, entendido como el ecosistema de aplicaciones que da soporte al ciclo completo de un pedido: su creación por parte del Cliente, su confirmación o rechazo por parte de la Tienda / Dispatcher, la asignación de un Repartidor —ya sea de forma automática o manual—, la ejecución de la entrega, la eventual cancelación del pedido en cualquiera de sus estados, la consulta de su estado por parte del Cliente y, finalmente, la calificación del servicio recibido. El documento describe la arquitectura a nivel de las cinco vistas del modelo 4+1 —casos de uso, lógica, procesos, desarrollo y física— sin descender al nivel de diseño detallado de clases o de esquemas de base de datos, los cuales quedan representados en el modelo Rational Rose adjunto.

## 1.3 Definiciones, Acrónimos y Abreviaturas

- **RUP:** Rational Unified Process, proceso de desarrollo de software sobre el cual se estructura la documentación del proyecto.
- **Vista 4+1:** modelo de representación arquitectónica compuesto por las vistas lógica, de procesos, de desarrollo y física, organizadas en torno a los escenarios de casos de uso.
- **CUS:** Caso de Uso del Sistema, conforme a la numeración establecida en el documento DS-CUS-01-09.
- **RN:** Regla del Negocio, conforme a la numeración establecida en el documento DS-RN-01.
- **RF:** Requerimiento Funcional, conforme a la numeración establecida en el documento DS-SRS-01.
- **FCM:** Firebase Cloud Messaging, servicio empleado para el envío de notificaciones push.
- **WebSocket:** protocolo de comunicación bidireccional persistente empleado para la actualización en tiempo real de las aplicaciones cliente.
- **API REST:** interfaz de programación de aplicaciones basada en el estilo arquitectónico REST, expuesta por el backend de DeliverySuite.
- **.mdl:** extensión del archivo de modelo de Rational Rose, herramienta empleada para la construcción de los diagramas UML del proyecto.

## 1.4 Documentos Relacionados

| Título | Identificador |
|---|---|
| Actores del Sistema | DS-ACT-01 |
| Reglas del Negocio | DS-RN-01 |
| Especificaciones de Caso de Uso del Sistema | DS-CUS-01-09 |
| Especificación de Requerimientos de Software | DS-SRS-01 |
| Modelo de diagramas UML (Rational Rose) | `DeliverySuite.mdl` |

---

# 2. Representación Arquitectónica

La arquitectura de DeliverySuite se organiza siguiendo el modelo de vistas 4+1. Cada una de las cuatro vistas principales atiende a un conjunto distinto de interesados y a un conjunto distinto de preocupaciones: la vista lógica interesa principalmente a los analistas y diseñadores, en tanto describe las abstracciones del dominio del negocio; la vista de procesos interesa a quienes deben garantizar el correcto funcionamiento dinámico y la consistencia del sistema en ejecución; la vista de desarrollo interesa al equipo de programación, en tanto organiza el código fuente en módulos y aplicaciones independientes; y la vista física interesa a quienes despliegan y operan el sistema en producción, en tanto describe la infraestructura sobre la cual este se ejecuta. La quinta vista, la de casos de uso, no constituye una vista adicional en el mismo sentido que las anteriores, sino el hilo conductor que valida y motiva las decisiones tomadas en las otras cuatro: los nueve casos de uso especificados para DeliverySuite son, precisamente, los escenarios que ponen a prueba y justifican cada elección estructural descrita en las secciones siguientes.

Bajo este enfoque, la arquitectura de DeliverySuite responde a un conjunto de decisiones centrales que atraviesan las cinco vistas: la concentración de toda la lógica de negocio en un backend único construido sobre NestJS, que actúa como única fuente de verdad del sistema; la separación estricta entre dicho backend y las aplicaciones cliente, ya sean móviles o web, las cuales se limitan a consumir la API expuesta por el backend; el uso de una secuencia de estados estrictamente unidireccional para modelar el ciclo de vida del pedido, tal como establece la regla RN-02; y el uso combinado de notificaciones push y de comunicación en tiempo real mediante WebSockets para mantener sincronizados a todos los actores durante dicho ciclo de vida.

---

# 3. Vista de Casos de Uso (+1)

La vista de casos de uso reúne los escenarios arquitectónicamente significativos de DeliverySuite, es decir, aquellos casos de uso cuyo cumplimiento exige decisiones estructurales concretas y cuya omisión comprometería la validez de la arquitectura propuesta. Los nueve casos de uso especificados en el documento DS-CUS-01-09 cumplen íntegramente esta condición, en la medida en que, en conjunto, recorren el ciclo de vida completo del pedido y exigen del sistema capacidades de validación, de automatización, de notificación en tiempo real y de consistencia transaccional.

El caso de uso CUS-01, Crear Pedido, es significativo porque exige que el sistema centralice en un único punto de entrada —el backend— tanto la validación de las condiciones mínimas del pedido como el cálculo de la tarifa de envío, descrito en el subflujo SF-01. Esta exigencia motiva la decisión arquitectónica de que ninguna aplicación cliente calcule tarifas por su cuenta: el cálculo, que depende de reglas configurables y de datos geoespaciales, debe residir exclusivamente en el backend para garantizar que el valor cobrado al Cliente sea siempre el mismo con independencia de la aplicación desde la cual se origine el pedido. Asimismo, la necesidad de conservar un snapshot inmutable de los ítems, conforme a la regla RN-18, impacta directamente en el diseño de la vista lógica, al exigir que la entidad ÍtemPedido copie los datos del producto en el momento de la creación, en lugar de referenciarlos de manera dinámica.

Los casos de uso CUS-02 y CUS-03, Confirmar Pedido y Rechazar Pedido, son significativos porque introducen la necesidad de un mecanismo de notificación confiable y de baja latencia hacia la Tienda / Dispatcher y, en sentido inverso, hacia el Cliente. Esta necesidad motiva la incorporación de un módulo de notificaciones desacoplado de la lógica de negocio, capaz de enviar mensajes push a través de FCM inmediatamente después de cada transición de estado relevante.

Los casos de uso CUS-04 y CUS-05, Asignar Repartidor Automáticamente y Asignar Repartidor Manualmente, son quizás los más significativos desde el punto de vista arquitectónico, en tanto exigen que el sistema soporte dos estrategias de asignación intercambiables sobre el mismo modelo de estados, gobernadas por un parámetro de configuración global (RN-08). Esta exigencia motiva la decisión de aislar la lógica de selección de repartidores en un servicio de dominio independiente —el Sistema de Asignación descrito en DS-ACT-01— que encapsula tanto el algoritmo de selección por proximidad y disponibilidad como el mecanismo de reintento ante la falta de aceptación del candidato designado. De no aislarse esta lógica, la alternancia entre los modos AUTO_PROXIMITY y ADMIN_MANUAL obligaría a duplicar las validaciones de disponibilidad del Repartidor (RN-06) en dos lugares distintos del sistema.

El caso de uso CUS-06, Ejecutar Entrega, es significativo porque concentra la mayor cantidad de transiciones de estado consecutivas del ciclo de vida del pedido y porque, en su culminación, exige la ejecución atómica del registro de comisiones financieras (subflujo SF-01) junto con la transición al estado DELIVERED. Esta exigencia de atomicidad, recogida en el requerimiento RC-03, motiva que el diseño de la vista de procesos trate la finalización de la entrega y la liquidación de comisiones como una única unidad de trabajo dentro del backend, evitando así estados inconsistentes en los que un pedido figure como entregado sin que sus comisiones hayan sido registradas.

El caso de uso CUS-07, Cancelar Pedido, es significativo porque introduce múltiples actores con distintos niveles de autorización sobre una misma operación, en función del estado del pedido (RN-04), y porque cada tipo de cancelación conlleva un tratamiento financiero distinto (RN-05). Esta variabilidad motiva que la vista lógica del sistema module la cancelación como una operación parametrizada por tipo, en lugar de un conjunto de operaciones independientes por cada actor.

El caso de uso CUS-08, Consultar Estado / Seguimiento del Pedido, es significativo porque, a diferencia de los anteriores, no modifica el estado del sistema, pero exige una actualización continua y de baja latencia hacia el Cliente. Este requerimiento motiva la incorporación de un canal de comunicación en tiempo real —descrito con mayor detalle en la vista de procesos— que complementa a las notificaciones push discretas con una vía adicional de sincronización mientras la aplicación del Cliente permanece activa.

Finalmente, el caso de uso CUS-09, Calificar Pedido, si bien es de menor complejidad estructural que los anteriores, es significativo porque cierra el ciclo de vida del pedido y confirma que el diseño del sistema contempla, además de la ejecución operativa, la recolección de información para la evaluación del servicio, sin que dicha recolección bloquee ningún flujo posterior del Cliente.

---

# 4. Vista Lógica

La vista lógica describe las abstracciones principales del dominio de DeliverySuite y las responsabilidades que cada una asume dentro del sistema. Estas abstracciones se representan como clases y relaciones en el modelo Rational Rose adjunto; en el presente documento se describen en prosa sus responsabilidades y su forma de relacionarse entre sí.

La entidad central del dominio es el **Pedido**, que representa una solicitud de compra realizada por un Cliente a una Tienda determinada. El Pedido es responsable de mantener su propio estado a lo largo de todo su ciclo de vida, conforme a la secuencia unidireccional establecida en la regla RN-02, y de conservar los datos que resultan relevantes para su trazabilidad: la dirección de entrega con sus coordenadas geográficas, el método de pago seleccionado, la tarifa de envío calculada y fijada en el momento de su creación, y las marcas de tiempo correspondientes a cada hito relevante —confirmación, asignación, aceptación, llegada a la tienda, recogida, llegada al cliente y entrega—. El Pedido se relaciona con el Cliente que lo origina, con la Tienda a la que se dirige, con el Repartidor que eventualmente lo atiende, y con el conjunto de Ítems de Pedido que lo componen. Adicionalmente, el Pedido es responsable de exponer, ante una eventual cancelación, el tipo de cancelación y el motivo asociado, conforme a las reglas RN-04 y RN-05.

El **Cliente** representa a la persona que utiliza la plataforma para realizar pedidos. Su responsabilidad principal dentro del modelo lógico es la de originar Pedidos y mantener asociada a su perfil la información necesaria para ello, entre otras su identidad autenticada y sus direcciones de entrega. El Cliente se relaciona con el Pedido en una relación de uno a muchos, en tanto un mismo Cliente puede haber originado múltiples pedidos a lo largo del tiempo, y es también el actor responsable de emitir la calificación asociada a un Pedido concluido, conforme al CUS-09.

La **Tienda** representa al establecimiento afiliado que ofrece productos para su entrega a domicilio. Su responsabilidad central es la de mantener su catálogo de productos y decidir, para cada Pedido dirigido a ella, si lo confirma o lo rechaza. La Tienda mantiene, además, los atributos que determinan su tratamiento financiero —su tipo de tienda, del cual depende el cálculo de la comisión de plataforma conforme a la regla RN-15— y los atributos que determinan la tarifa de envío que le resulta aplicable, tales como su multiplicador de tarifa. La Tienda se relaciona con el Pedido en una relación de uno a muchos, y con sus propias sucursales de despacho, las cuales representan los puntos físicos desde los que puede efectivizarse la preparación y entrega de un pedido; esta relación entre Tienda y sucursal es la que sustenta la selección de `storeLocationId` exigida por la regla RN-03 al momento de confirmar un pedido.

El **Repartidor** representa a la persona encargada de recoger el pedido en la tienda y entregarlo en la dirección indicada por el Cliente. Su responsabilidad principal es la de mantener su propio estado de disponibilidad —conectado o desconectado— y de avanzar el Pedido que le ha sido asignado a través de los estados operativos de entrega descritos en el CUS-06. El Repartidor mantiene, asimismo, los atributos que determinan su capacidad operativa: su rango, del cual se deriva su límite de pedidos simultáneos conforme a la regla RN-07, y su tasa de comisión, la cual puede encontrarse configurada de manera individual o resolverse por la jerarquía de configuración descrita en la regla RN-16. El Repartidor se relaciona con el Pedido en una relación de uno a muchos limitada por su propio límite de concurrencia, lo cual constituye una particularidad relevante del dominio: a diferencia de la Tienda, que puede recibir un número no acotado de pedidos activos, el Repartidor se encuentra sujeto a una restricción de capacidad que el sistema debe validar antes de cada asignación, conforme a la regla RN-06.

El **Ítem de Pedido** representa cada línea de producto que compone un Pedido. Su responsabilidad distintiva dentro del modelo lógico es la de actuar como una copia inmutable —un snapshot— de los datos del producto vigentes en el momento en que el Pedido fue creado, conforme a la regla RN-18: el nombre del producto, su precio unitario, su variante si corresponde, y el subtotal resultante. Esta responsabilidad de "congelar" la información distingue al Ítem de Pedido de una simple referencia al catálogo de la Tienda, y resulta central para garantizar que las modificaciones posteriores del catálogo no alteren el valor de los pedidos ya creados. El Ítem de Pedido se relaciona con el Pedido en una relación de muchos a uno, siendo cada Ítem de Pedido parte constitutiva de un único Pedido.

De manera transversal a estas cinco entidades principales, el modelo lógico contempla una entidad de apoyo destinada a la trazabilidad, el Evento de Pedido, la cual registra cada transición de estado ocurrida sobre un Pedido —el estado anterior, el estado nuevo, el actor que la ejecutó y la marca de tiempo correspondiente— en cumplimiento del requerimiento de auditoría RS-01. Esta entidad no participa del flujo operativo del negocio, pero resulta indispensable para sostener la trazabilidad exigida por el sistema.

---

# 5. Vista de Procesos

La vista de procesos describe el comportamiento dinámico del sistema en tiempo de ejecución, con particular énfasis en el ciclo de vida del Pedido y en el mecanismo mediante el cual dicho ciclo de vida se comunica, en tiempo real, a los distintos actores involucrados.

El proceso central de DeliverySuite es el ciclo de vida del Pedido, gobernado por la secuencia de estados establecida en la regla RN-02: PENDING, CONFIRMED, WAITING_DRIVER, DRIVER_ASSIGNED, ACCEPTED, AT_STORE, PICKED_UP, AT_CUSTOMER y DELIVERED, con CANCELLED como único estado terminal alternativo. Cada transición de este proceso es disparada por la acción de un actor específico —el Cliente al crear el pedido, la Tienda / Dispatcher al confirmarlo o rechazarlo, el Sistema de Asignación o el Administrador al asignar un Repartidor, y el Repartidor al ejecutar cada hito de la entrega— y es procesada de manera sincrónica por el backend, el cual valida las precondiciones correspondientes antes de efectivizar el cambio de estado. Esta naturaleza sincrónica de la validación resulta esencial para el requerimiento de consistencia RC-02: ante un fallo ocurrido en mitad de una transición, el proceso debe garantizar que el Pedido permanezca en su estado anterior, sin registrar estados intermedios inválidos.

Dentro de este proceso general existe un subproceso de particular complejidad, correspondiente a la asignación automática de Repartidor (CUS-04). Cuando el Pedido alcanza el estado WAITING_DRIVER bajo el modo AUTO_PROXIMITY, se activa un proceso de selección que se ejecuta de manera asíncrona respecto de la solicitud original del Cliente: el Sistema de Asignación evalúa a los repartidores disponibles, designa a un candidato y aguarda su aceptación dentro de una ventana de tiempo determinada. Este proceso incorpora un mecanismo de reintento: si el candidato designado no responde dentro del plazo establecido, el proceso vuelve a ejecutarse sobre el siguiente candidato disponible, sin que el Pedido retroceda a un estado anterior. Si el proceso se agota sin encontrar un candidato disponible, deriva en un proceso de alerta operativa, mediante el cual el sistema notifica al Administrador la existencia de un pedido huérfano, conforme al requerimiento RF-18.

Otro proceso relevante es el de liquidación financiera asociado a la conclusión de la entrega (CUS-06, subflujo SF-01). Este proceso se ejecuta en el mismo instante en que el Pedido transiciona al estado DELIVERED y comprende la determinación del tipo de tienda, el cálculo de la comisión de plataforma, la eventual determinación del ingreso de tienda según el método de pago, la resolución de la tasa de comisión del Repartidor conforme a la jerarquía establecida en la regla RN-16, y el registro de las entradas resultantes en el ledger financiero. Este proceso debe ejecutarse como una unidad atómica junto con la transición de estado, conforme al requerimiento RC-03, de modo que no pueda producirse un Pedido marcado como entregado cuyas comisiones no hayan sido registradas.

El proceso de actualización en tiempo real constituye el mecanismo transversal que conecta al backend con las aplicaciones cliente durante todo el ciclo de vida del Pedido, y combina dos canales complementarios. El primero de ellos son las notificaciones push, entregadas mediante FCM: en cada transición de estado relevante, el backend emite de manera asíncrona un mensaje push dirigido al actor o actores que deben ser informados de dicho cambio —el Cliente ante la confirmación, la asignación, la recogida, la llegada y la entrega; la Tienda / Dispatcher ante la llegada del repartidor y la conclusión de la entrega; y el Repartidor ante su propia asignación—, conforme al requerimiento RF-17. Este canal resulta apropiado para informar a un usuario incluso cuando su aplicación no se encuentra activa en primer plano, dado que el sistema operativo del dispositivo entrega la notificación de forma independiente del estado de la aplicación. El segundo canal es la comunicación en tiempo real mediante WebSockets, empleada mientras la aplicación del Cliente, de la Tienda o del Repartidor permanece activa y en primer plano: a través de este canal, el backend difunde eventos de cambio de estado que las aplicaciones cliente utilizan para refrescar de inmediato sus vistas —por ejemplo, la vista de seguimiento del Pedido descrita en el CUS-08— sin requerir que el usuario recargue manualmente la información ni que la aplicación dependa exclusivamente de la llegada de una notificación push. La combinación de ambos canales garantiza que el requerimiento de rendimiento RP-02, que exige que cada transición de estado se notifique a los actores involucrados en menos de dos segundos, se satisfaga tanto para los usuarios con la aplicación activa como para aquellos que la tienen en segundo plano.

---

# 6. Vista de Desarrollo

La vista de desarrollo describe cómo se organiza el código fuente de DeliverySuite en aplicaciones y módulos independientes, reflejando la restricción de diseño RD-01, según la cual toda la lógica de negocio debe residir en el backend y las aplicaciones cliente deben limitarse a consumirla.

El sistema se organiza en tres grandes componentes de desarrollo, claramente separados entre sí. El primero es el **backend**, una única aplicación construida sobre el framework NestJS, que constituye la única fuente de verdad del negocio: expone una API REST sobre HTTPS mediante la cual todas las aplicaciones cliente leen y modifican el estado del sistema, conforme al requerimiento de interfaz RI-S-01, y expone además un espacio de nombres de WebSocket destinado a la difusión de eventos en tiempo real, conforme al requerimiento RI-S-02. El segundo componente son las **aplicaciones móviles**, desarrolladas en Flutter, que constituyen la vía principal de interacción para el Cliente y para el Repartidor. El tercer componente son las **aplicaciones web**, desarrolladas en React, que ofrecen una vía alternativa de acceso para determinados actores, en particular para el Administrador, que opera exclusivamente a través de un panel web, y para la Tienda / Dispatcher, que cuenta con una vía web complementaria a su aplicación móvil.

Dentro del backend, el código se organiza en módulos independientes, cada uno responsable de un subconjunto acotado de la funcionalidad del sistema, en consonancia con los actores y casos de uso descritos en los documentos DS-ACT-01 y DS-CUS-01-09. El módulo de autenticación es responsable de la identificación de los usuarios y de la emisión de los tokens que habilitan el acceso a los demás módulos, conforme a la restricción de diseño RD-04. El módulo de usuarios mantiene los datos comunes a todos los actores humanos del sistema y su rol dentro de él. El módulo de repartidores extiende la información del usuario con los atributos propios del Repartidor: su rango, su límite de pedidos simultáneos, su estado de disponibilidad y su tasa de comisión. El módulo de tiendas mantiene el catálogo, las sucursales y los atributos financieros de cada Tienda / Dispatcher. El módulo de productos administra el catálogo de productos ofrecidos por cada tienda, incluyendo sus variantes. El módulo de pedidos constituye el núcleo del sistema: concentra las entidades Pedido, Ítem de Pedido y Evento de Pedido, y aloja tanto el servicio que gobierna el ciclo de vida completo del Pedido como el servicio de asignación que implementa la lógica descrita en los casos de uso CUS-04 y CUS-05. El módulo de notificaciones encapsula el envío de mensajes push mediante FCM, siendo consumido por el módulo de pedidos en cada transición de estado relevante. El módulo de alertas operativas encapsula el envío de avisos automáticos al Administrador mediante bots de Telegram, conforme al requerimiento RF-18. Finalmente, el módulo de configuración mantiene los parámetros globales del sistema —el modo de asignación, el modo de cálculo de tarifa, las reglas de tarifa y la configuración de comisiones— de forma centralizada, permitiendo que el Administrador los modifique sin necesidad de un nuevo despliegue del sistema, conforme al requerimiento RS-03.

Las tres aplicaciones móviles —dirigida al Cliente, al Repartidor y a la Tienda / Dispatcher, respectivamente— comparten una misma base tecnológica en Flutter y una misma forma de comunicarse con el backend: mediante un cliente HTTP que adjunta el token de autenticación en cada solicitud, y mediante un cliente de WebSocket que se suscribe a los eventos relevantes para el rol del usuario que la utiliza. Cada aplicación organiza su propio código en función de las pantallas y funcionalidades que le resultan propias según el actor al que sirve: la aplicación del Cliente se organiza en torno a la exploración de tiendas, la construcción del pedido y el seguimiento de su estado; la aplicación del Repartidor se organiza en torno a la gestión de su disponibilidad, la visualización de los pedidos que le son ofrecidos o asignados, y el registro de los hitos de la entrega; y la aplicación de la Tienda / Dispatcher se organiza en torno a la recepción y confirmación de pedidos y a la gestión de su catálogo.

Las aplicaciones web, por su parte, siguen el mismo principio de consumir exclusivamente la API del backend, sin duplicar lógica de negocio. La aplicación web dirigida al Administrador concentra la gestión integral de pedidos, repartidores, tiendas, usuarios y la configuración de los parámetros globales del sistema. La aplicación web dirigida a la Tienda / Dispatcher ofrece una alternativa de escritorio a su aplicación móvil para las mismas funciones de gestión de pedidos y de catálogo.

---

# 7. Vista Física

La vista física describe la disposición de los elementos de hardware y de los servicios de infraestructura sobre los cuales se ejecuta DeliverySuite, así como la forma en que dichos elementos se comunican entre sí en el entorno de producción.

El componente central de la infraestructura es el **servidor de la API del backend**, sobre el cual se despliega la aplicación NestJS que concentra toda la lógica de negocio del sistema. Este servidor expone dos superficies de comunicación hacia las aplicaciones cliente: la interfaz REST sobre HTTPS, empleada para todas las operaciones de lectura y de escritura del estado del sistema, y la interfaz de WebSocket, empleada para la difusión de eventos en tiempo real conforme a lo descrito en la vista de procesos. El servidor de la API constituye, asimismo, el único componente de la infraestructura autorizado a comunicarse con la base de datos y con los servicios externos del sistema, en cumplimiento de la restricción de diseño RD-01, que reserva al backend la exclusividad sobre la lógica de negocio.

El segundo componente central es la **base de datos PostgreSQL**, gestionada de forma administrada y desplegada de manera independiente del servidor de la API, con el cual se comunica mediante una conexión privada. En esta base de datos se persisten todas las entidades descritas en la vista lógica del presente documento —el Pedido, el Cliente, la Tienda, el Repartidor y el Ítem de Pedido, junto con las entidades de configuración y de auditoría que les dan soporte—. La totalidad de la lógica de automatización del sistema, incluyendo el cálculo de tarifas, la asignación de repartidores y el registro de comisiones, se ejecuta en la capa de aplicación del backend y no mediante mecanismos nativos de la base de datos, conforme a la restricción de diseño RD-03.

Además de estos dos componentes centrales, la arquitectura física de DeliverySuite incorpora un conjunto de **servicios externos** con los cuales el servidor de la API se comunica a través de internet. El primero de estos servicios es Firebase Cloud Messaging, empleado para la entrega de notificaciones push a los dispositivos móviles y a los navegadores de todos los actores del sistema, conforme al requerimiento RCA-01; este servicio recibe del backend la orden de envío junto con el identificador del dispositivo destinatario, y se encarga de entregar la notificación a través de la infraestructura de Google, con independencia de si la aplicación cliente se encuentra activa en ese momento. El segundo servicio externo es el de mapas y geolocalización, empleado directamente por las aplicaciones móviles —en particular por la del Repartidor y la del Cliente— para la visualización de mapas y el seguimiento de la posición del Repartidor en tiempo real mediante el GPS del dispositivo, conforme al requerimiento RI-H-01; este servicio es consumido de manera distribuida por cada aplicación móvil, sin que el servidor de la API intermedie en la renderización del mapa, aunque las coordenadas geográficas relevantes para el cálculo de la tarifa de envío sí son procesadas en el backend, conforme a lo descrito en la vista de procesos. Un tercer servicio externo, empleado por el módulo de alertas operativas, es la plataforma de mensajería utilizada para notificar al Administrador ante situaciones excepcionales, conforme al requerimiento RI-C-02, para lo cual el servidor de la API mantiene conectividad saliente hacia dicho servicio.

Sobre este conjunto de componentes, las tres aplicaciones móviles y las aplicaciones web se despliegan de manera independiente del servidor de la API y de la base de datos: las aplicaciones móviles se distribuyen a los dispositivos de los usuarios a través de las tiendas de aplicaciones correspondientes a cada plataforma, mientras que las aplicaciones web se sirven como aplicaciones de una sola página (single-page applications) desde un servicio de alojamiento de contenido estático, y se comunican con el servidor de la API exclusivamente a través de internet, empleando los mismos canales REST y WebSocket que emplean las aplicaciones móviles. Esta separación física entre los dispositivos cliente, el servidor de la API y la base de datos permite que cada componente escale de manera independiente y que el mantenimiento o la actualización de uno de ellos no exija necesariamente la interrupción de los demás.

---

# 8. Restricciones Arquitectónicas y Elementos Fuera de Alcance

La arquitectura descrita en este documento se circunscribe estrictamente al ciclo de vida del Pedido y a los nueve casos de uso especificados en el documento DS-CUS-01-09. En consecuencia, quedan expresamente fuera del alcance de las cinco vistas presentadas los siguientes elementos, con independencia de que puedan existir en desarrollos relacionados o en versiones futuras del sistema: los mecanismos de cupones y descuentos promocionales; el sistema de billetera o saldo prepagado del usuario; el programa de referidos entre usuarios; los pedidos de tipo B2B originados por canales externos a las aplicaciones propias de la plataforma; el soporte de múltiples sucursales operando como unidades de negocio independientes entre sí, más allá de la simple selección de la sucursal de despacho contemplada en la regla RN-03; y el soporte de operación simultánea en múltiples países. Ninguno de estos elementos ha sido considerado en la Vista de Casos de Uso, en la Vista Lógica, en la Vista de Procesos, en la Vista de Desarrollo ni en la Vista Física del presente documento, y su eventual incorporación futura requeriría una revisión específica de la arquitectura aquí descrita.
