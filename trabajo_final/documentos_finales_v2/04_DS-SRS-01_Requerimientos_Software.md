# DeliverySuite

# Proyecto: DeliverySuite

# Especificación de Requerimientos de Software

## Versión 1.0

Identificador del documento: DS-SRS-01

---

# Historial de Versiones

| Fecha | Versión | Descripción | Autor |
|---|---|---|---|
| 29/Jun/26 | 1.0 | Primera versión — requerimientos del ciclo completo de pedido | Flores Hoyos, Mathias |
| 29/Jun/26 | 1.0 | Primera versión — requerimientos del ciclo completo de pedido | Príncipe Caballa, Aida |
| 29/Jun/26 | 1.0 | Primera versión — requerimientos del ciclo completo de pedido | Ochoa Torres, Yoant Alnor |

---

# Tabla de Contenidos

- 1. Funcionalidad
  - 1.1 Asociados a los casos de uso
  - 1.2 Asociados a aspectos generales
- 2. Usabilidad
- 3. Confiabilidad
- 4. Rendimiento
- 5. Soporte
- 6. Restricciones de Diseño
- 7. Documentación de Usuario y Sistema de Ayuda
- 8. Componentes Adquiridos
- 9. Interfases
  - 9.1 Interfases de Usuarios
  - 9.2 Interfases de Hardware
  - 9.3 Interfases de Software
  - 9.4 Interfases de Comunicaciones
- 10. Licenciamiento
- 11. Requerimientos Legales y de Derecho de Autor
- 12. Estándares Aplicables

---

# Especificación de Requerimientos de Software

El presente documento recoge, de manera integral, los requerimientos de software del sistema DeliverySuite, complementando en un solo cuerpo normativo las especificaciones funcionales derivadas de los nueve casos de uso del sistema (documento DS-CUS-01-09) con los requerimientos no funcionales de usabilidad, confiabilidad, rendimiento, soporte, diseño, documentación, componentes adquiridos, interfaces, licenciamiento, aspectos legales y estándares aplicables. Su organización sigue la estructura recomendada por el estándar IEEE 830-1998, adaptada a la plantilla del curso basada en el Rational Unified Process.

## 1. Funcionalidad

Esta sección describe los requerimientos funcionales del sistema, organizados en dos grupos: aquellos que se derivan directamente de los casos de uso especificados en el documento DS-CUS-01-09, y aquellos que responden a aspectos transversales del comportamiento del sistema y no se circunscriben a un único caso de uso.

### 1.1 Asociados a los casos de uso

**1.1.1 RF-01 — Creación de pedido (CUS-01)**
El sistema debe permitir al Cliente autenticado crear un pedido seleccionando una tienda activa, uno o más productos con su respectiva cantidad y variante, una dirección de entrega con coordenadas geográficas, y un método de pago que puede ser efectivo (CASH), tarjeta (CARD) o billetera (WALLET). Al confirmarse el pedido, el sistema debe registrarlo en estado PENDING conservando un snapshot inmutable de nombre, precio unitario, subtotal y variante de cada ítem, calcular y fijar la tarifa de envío, y notificar a la Tienda / Dispatcher correspondiente.

**1.1.2 RF-02 — Cálculo de tarifa de envío al crear el pedido (CUS-01 / SF-01)**
El sistema debe calcular la tarifa de envío en el momento de creación del pedido, aplicando el modo de cálculo activo —ZONE o DISTANCE—, las reglas de tarifa configuradas en orden de prioridad, el multiplicador de tienda y el redondeo al medio sol superior. El valor calculado debe quedar fijo en el pedido y no debe recalcularse en los estados posteriores del ciclo de vida.

**1.1.3 RF-03 — Confirmación de pedido por la Tienda / Dispatcher (CUS-02)**
El sistema debe permitir a la Tienda / Dispatcher autenticada confirmar un pedido que se encuentre en estado PENDING, seleccionando la sucursal de despacho e indicando el tiempo estimado de preparación en minutos. Al ejecutarse la confirmación, el sistema debe transicionar el pedido a CONFIRMED, notificar al Cliente y activar automáticamente la búsqueda de repartidor.

**1.1.4 RF-04 — Rechazo de pedido por la Tienda / Dispatcher (CUS-03)**
El sistema debe permitir a la Tienda / Dispatcher rechazar un pedido que se encuentre en estado PENDING. Al rechazarlo, el sistema debe registrar el tipo de cancelación STORE_FAULT_BEFORE_DRIVER_PAID, transicionar el pedido a CANCELLED, notificar al Cliente y abstenerse de generar entradas en el ledger financiero.

**1.1.5 RF-05 — Asignación automática de Repartidor (CUS-04)**
En el modo AUTO_PROXIMITY, el sistema debe seleccionar automáticamente a un Repartidor que satisfaga las condiciones de disponibilidad establecidas —esto es, encontrarse conectado, permanecer dentro de su límite de pedidos simultáneos según rango, y pertenecer a una región coincidente— transicionar el pedido a DRIVER_ASSIGNED y notificar al candidato seleccionado. Si dicho candidato no acepta el pedido dentro del tiempo establecido, el sistema debe reintentar la asignación con el siguiente candidato disponible; si no existiera ningún candidato disponible, el sistema debe generar una alerta dirigida al Administrador.

**1.1.6 RF-06 — Asignación manual de Repartidor por el Administrador (CUS-05)**
En el modo ADMIN_MANUAL, el sistema debe permitir al Administrador autenticado seleccionar un Repartidor disponible de la lista filtrada según las condiciones de disponibilidad, validar su estado en tiempo real al momento de confirmar la asignación, registrar dicha asignación transicionando el pedido a DRIVER_ASSIGNED, y notificar al Repartidor correspondiente.

**1.1.7 RF-07 — Ejecución de entrega por el Repartidor (CUS-06)**
El sistema debe permitir al Repartidor asignado aceptar el pedido, transicionándolo a ACCEPTED; registrar su llegada al local, transicionándolo a AT_STORE; registrar la recogida del pedido, transicionándolo a PICKED_UP; registrar su llegada al destino, transicionándolo a AT_CUSTOMER; y confirmar la entrega, transicionándolo a DELIVERED. En cada una de estas transiciones, el sistema debe registrar la marca de tiempo correspondiente y notificar a los actores involucrados.

**1.1.8 RF-08 — Registro de comisiones al entregar (CUS-06 / SF-01)**
Al transicionar el pedido al estado DELIVERED, el sistema debe calcular y registrar en el ledger financiero la comisión de plataforma aplicada a la tienda según su tipo —OFFICIAL_10, OFFICIAL_5 o NON_OFFICIAL—, el ingreso de tienda cuando el tipo es OFFICIAL_10 y el pago se realizó mediante CARD, y la comisión del Repartidor, calculada sobre la tarifa de envío empleando la tasa resuelta según la jerarquía de configuración establecida.

**1.1.9 RF-09 — Cancelación de pedido (CUS-07)**
El sistema debe permitir la cancelación de pedidos activos conforme a las siguientes reglas de autorización: el Cliente puede cancelar únicamente mientras el pedido se encuentra en PENDING; el Administrador y Soporte pueden cancelar desde cualquier estado activo posterior a PENDING. Toda cancelación debe registrar el tipo correspondiente —CLIENT_FAULT, STORE_FAULT_BEFORE_DRIVER_PAID, STORE_FAULT_AFTER_DRIVER_PAID o DRIVER_FAULT—, aplicar las consecuencias financieras que dicho tipo define, y notificar a todos los actores involucrados.

**1.1.10 RF-10 — Consulta de estado y seguimiento del pedido (CUS-08)**
El sistema debe permitir al Cliente autenticado consultar el estado actual de cualquiera de sus pedidos, ya sean activos o concluidos, mostrando los datos del Repartidor asignado cuando el pedido se encuentre en estado ACCEPTED o en un estado posterior. La vista debe mantenerse actualizada mediante notificaciones FCM ante cada cambio de estado, sin requerir ninguna acción adicional por parte del Cliente.

**1.1.11 RF-11 — Calificación del pedido (CUS-09)**
El sistema debe permitir al Cliente calificar el servicio de entrega una vez que su pedido haya alcanzado el estado DELIVERED, siempre que no haya emitido previamente una calificación para ese pedido. La calificación debe admitir una puntuación y, opcionalmente, un comentario. Al ser de carácter opcional, su omisión no debe bloquear ningún flujo posterior del Cliente.

---

### 1.2 Asociados a aspectos generales

**1.2.1 RF-12 — Secuencia de estados unidireccional**
El sistema debe garantizar que los estados de un pedido avancen exclusivamente conforme a la secuencia definida —PENDING, CONFIRMED, WAITING_DRIVER, DRIVER_ASSIGNED, ACCEPTED, AT_STORE, PICKED_UP, AT_CUSTOMER y DELIVERED— sin permitir saltos ni reversiones, con la única excepción de la reiteración en el estado DRIVER_ASSIGNED durante el reintento de asignación.

**1.2.2 RF-13 — Snapshot inmutable de ítems**
El sistema debe registrar en la tabla `order_items` el nombre, el precio unitario, el subtotal y la variante de cada ítem en el momento de creación del pedido. Este snapshot no debe modificarse ante cambios posteriores en el catálogo de la tienda.

**1.2.3 RF-14 — Modo de asignación global configurable**
El sistema debe soportar dos modos de asignación, AUTO_PROXIMITY y ADMIN_MANUAL, como un parámetro global de tipo singleton. Solamente un modo puede permanecer activo en un momento dado, y el Administrador debe poder alternar el modo activo desde el panel de administración.

**1.2.4 RF-15 — Modo de cálculo de tarifa global configurable**
El sistema debe soportar dos modos de cálculo de la tarifa de envío, ZONE y DISTANCE, como un parámetro de alcance global. El Administrador debe poder configurar el modo activo, los valores propios de cada zona —polígonos, tarifas base, mínimos y máximos— y las reglas de tarifa adicionales, incluyendo su tipo, valor y filtros de ciudad, día y horario.

**1.2.5 RF-16 — Control de disponibilidad del Repartidor**
El sistema debe permitir al Repartidor activar y desactivar su estado de disponibilidad, alternando entre conectado y desconectado. El sistema debe impedir, asimismo, la asignación de pedidos a repartidores que no satisfagan simultáneamente las siguientes condiciones: encontrarse conectados, no haber alcanzado su límite de pedidos simultáneos según su rango, y tener una región coincidente con la del pedido.

**1.2.6 RF-17 — Notificaciones en tiempo real a todos los actores**
El sistema debe enviar notificaciones push mediante FCM a los actores correspondientes en cada transición de estado del pedido: al Cliente, ante la confirmación, la asignación de repartidor, la recogida, la llegada al destino, la entrega y la cancelación; a la Tienda / Dispatcher, ante una nueva orden, la llegada del repartidor y la entrega; y al Repartidor, ante su asignación y ante una cancelación posterior a dicha asignación.

**1.2.7 RF-18 — Alertas operativas al Administrador**
El sistema debe generar alertas automáticas dirigidas al Administrador, mediante Telegram, ante las siguientes situaciones: un pedido que permanece en estado CONFIRMED sin avanzar dentro del tiempo configurado, un pedido que permanece en estado PICKED_UP sin completarse dentro del tiempo esperado, un pedido en estado WAITING_DRIVER sin repartidor asignado —constituyendo un pedido huérfano— y cancelaciones que permanecen sin resolver.

---

## 2. Usabilidad

Esta sección recoge los requerimientos que inciden directamente sobre la facilidad de uso del sistema por parte de sus actores.

**2.1 RU-01 — Retroalimentación inmediata ante acciones del usuario**
El sistema debe mostrar al usuario una respuesta visual inmediata ante cada acción significativa —creación de pedido, confirmación, cambio de estado— evitando tiempos de espera superiores a los percibidos como habituales en aplicaciones móviles de delivery.

**2.2 RU-02 — Flujo de creación de pedido en tres pasos**
El flujo de creación de pedido en la aplicación del Cliente debe completarse en no más de tres pasos principales: selección de productos, configuración de entrega —dirección y método de pago— y confirmación. La tarifa de envío debe mostrarse al Cliente antes de la confirmación final.

**2.3 RU-03 — Visibilidad del estado del pedido sin navegación adicional**
El estado actual del pedido, así como la información del Repartidor asignado cuando corresponda, deben ser accesibles desde la pantalla principal de pedidos activos del Cliente, sin requerir navegación hacia submenús adicionales.

---

## 3. Confiabilidad

Esta sección establece los requerimientos que garantizan la disponibilidad y la consistencia del sistema frente a fallos.

**3.1 RC-01 — Disponibilidad del servicio**
El sistema debe permanecer disponible para los actores operativos —Cliente, Tienda / Dispatcher y Repartidor— durante el horario de operación de las tiendas afiliadas, con una disponibilidad objetivo del 99 % dentro de dicho período.

**3.2 RC-02 — Consistencia del estado del pedido**
El sistema debe garantizar que las transiciones de estado de un pedido sean atómicas: ante un fallo ocurrido en mitad de una transición, el pedido debe permanecer en su estado anterior, sin que se registren estados intermedios inválidos.

**3.3 RC-03 — Consistencia del ledger financiero**
El registro de comisiones en el ledger financiero debe ejecutarse de forma atómica junto con la transición al estado DELIVERED. Un fallo producido durante dicho registro no debe dejar al pedido en estado DELIVERED sin sus correspondientes entradas financieras.

**3.4 RC-04 — Tiempo medio entre fallos y tiempo medio de reparación**
El sistema debe operar con un tiempo medio entre fallos (MTBF) objetivo no inferior a 720 horas de operación continua, y un tiempo medio de reparación (MTTR) objetivo no superior a 2 horas para incidencias que afecten la disponibilidad del backend o de la base de datos. Ambos indicadores deben monitorearse de manera continua a partir de los registros de operación del sistema.

**3.5 RC-05 — Clasificación de severidad de errores**
Toda incidencia registrada sobre el sistema debe clasificarse según su severidad en una de las siguientes categorías: crítica, cuando impide la creación de pedidos, el registro de comisiones o produce pérdida de datos del ciclo de vida del pedido; significativa, cuando degrada una funcionalidad sin impedir la operación general, tal como un retraso relevante en las notificaciones; y menor, cuando afecta aspectos cosméticos o de usabilidad sin comprometer la operación del negocio. La resolución de incidencias críticas debe priorizarse sobre las demás categorías.

---

## 4. Rendimiento

Esta sección especifica los tiempos de respuesta y la capacidad que el sistema debe sostener bajo condiciones normales de operación.

**4.1 RP-01 — Tiempo de respuesta en creación de pedido**
La creación de un pedido, incluyendo el cálculo de la tarifa de envío y la notificación a la Tienda / Dispatcher, debe completarse en menos de tres segundos bajo condiciones normales de carga.

**4.2 RP-02 — Tiempo de respuesta en transiciones de estado**
Cada transición de estado de un pedido —confirmación, asignación, hitos de entrega— debe procesarse y notificarse a los actores involucrados en menos de dos segundos bajo condiciones normales de carga.

**4.3 RP-03 — Capacidad de pedidos simultáneos**
El sistema debe soportar la operación simultánea de múltiples pedidos activos en distintos estados, sin que se produzca una degradación perceptible del rendimiento para ninguno de los actores.

**4.4 RP-04 — Throughput mínimo de transacciones**
El backend debe sostener un mínimo de 50 transacciones de escritura por segundo (creación de pedidos, transiciones de estado y registros en el ledger financiero, en conjunto) bajo condiciones normales de carga, sin degradación del tiempo de respuesta definido en RP-01 y RP-02.

**4.5 RP-05 — Modo de degradación ante sobrecarga**
Ante una condición de sobrecarga que impida sostener los tiempos de respuesta establecidos en RP-01 y RP-02, el sistema debe priorizar el procesamiento de las operaciones que modifican el estado del pedido —confirmación, asignación y hitos de entrega— por sobre las operaciones de sola consulta, tales como la descrita en el CUS-08, de modo que el ciclo de vida del pedido no se vea interrumpido.

---

## 5. Soporte

Esta sección describe los requerimientos orientados a facilitar la operación, la auditoría y el mantenimiento del sistema.

**5.1 RS-01 — Registro de auditoría de transiciones**
Toda transición de estado de un pedido debe quedar registrada en la tabla `order_events`, incluyendo el estado anterior, el estado nuevo, el actor que la ejecutó —o el valor SYSTEM cuando la transición es automática— y la marca de tiempo correspondiente. Este registro es inmutable y no puede ser modificado ni eliminado por ningún actor.

**5.2 RS-02 — Trazabilidad de cancelaciones**
Toda cancelación debe registrar el tipo de cancelación (`cancellation_type`) y el motivo textual (`cancellation_reason`), vinculados al pedido correspondiente, de modo que el Administrador o Soporte puedan revisarlos posteriormente.

**5.3 RS-03 — Configuración operativa sin redespliegue**
Los parámetros operativos principales —modo de asignación, modo de cálculo de tarifa, reglas de tarifa y configuración de comisiones— deben poder modificarse desde el panel de administración sin requerir un redespliegue del sistema.

---

## 6. Restricciones de Diseño

Esta sección recoge las decisiones de diseño que han sido establecidas como obligatorias para la construcción del sistema.

**6.1 RD-01 — API REST como interfaz principal del backend**
Toda la lógica de negocio del sistema debe centralizarse en el backend, expuesta a través de una API REST construida sobre NestJS. Las aplicaciones cliente, tanto móviles como web, no deben replicar lógica de negocio; deben limitarse a consumir los endpoints del backend para toda operación que modifique el estado del sistema.

**6.2 RD-02 — Zona horaria fija America/Lima**
Todos los timestamps registrados por el sistema deben expresarse en la zona horaria America/Lima como referencia operativa, con independencia de la ubicación geográfica del actor que origina la acción.

**6.3 RD-03 — Sin triggers nativos de base de datos**
Toda la lógica de automatización —transiciones de estado, cálculos, notificaciones y tareas programadas de monitoreo— debe implementarse en la capa de aplicación, mediante servicios de NestJS. No deben emplearse triggers nativos de PostgreSQL para la implementación de lógica de negocio.

**6.4 RD-04 — Autenticación mediante JWT**
Todos los endpoints que modifiquen el estado del sistema o que accedan a datos sensibles deben requerir un token JWT válido, firmado por el backend. El control de acceso por rol debe aplicarse mediante guardias (guards) en cada endpoint, en función del rol del actor —CLIENT, DRIVER, VENDOR, ADMIN o SUPPORT.

---

## 7. Documentación de Usuario y Sistema de Ayuda

Esta sección describe los requerimientos de documentación orientada a los actores operativos del sistema.

**7.1 RDoc-01 — Manual de operación para la Tienda / Dispatcher**
El sistema debe acompañarse de una guía de operación destinada a la Tienda / Dispatcher, que describa el flujo de gestión de pedidos: recepción de notificaciones, confirmación, selección de sucursal y seguimiento hasta la entrega.

**7.2 RDoc-02 — Guía de inicio para el Repartidor**
El sistema debe incluir una guía de inicio dirigida al Repartidor, que explique la activación del estado de conexión, la recepción y aceptación de pedidos, y el registro de cada hito de la entrega.

---

## 8. Componentes Adquiridos

Esta sección identifica los componentes de terceros de los que depende el funcionamiento del sistema.

**8.1 RCA-01 — Firebase Cloud Messaging (FCM)**
El sistema utiliza Firebase Cloud Messaging para el envío de notificaciones push a las aplicaciones móviles y web de todos los actores. Este componente es provisto por Google Firebase y debe configurarse con las credenciales correspondientes al proyecto.

**8.2 RCA-02 — Servicio de geolocalización y mapas**
El sistema utiliza la API de Google Maps en las aplicaciones móviles para la visualización del mapa de seguimiento del Repartidor y para la selección de direcciones de entrega. Su disponibilidad está sujeta a los términos de uso de Google Maps Platform.

**8.3 RCA-03 — Almacenamiento de imágenes (Cloudinary)**
El sistema utiliza Cloudinary para el almacenamiento y el servicio de imágenes correspondientes a tiendas, productos y perfiles de usuario. La disponibilidad de dichas imágenes depende de la disponibilidad del servicio externo.

---

## 9. Interfases

Esta sección define las interfaces que el sistema debe soportar, agrupadas según se trate de interfaces de usuario, de hardware, de software o de comunicaciones.

### 9.1 Interfases de Usuarios

**9.1.1 RI-U-01 — Aplicación móvil del Cliente (mobile_customer)**
Interfaz desarrollada en Flutter para dispositivos iOS y Android, que permite al Cliente explorar tiendas, crear pedidos, consultar el estado en tiempo real y calificar las entregas recibidas. Requiere autenticación mediante código de un solo uso (OTP) enviado al número de teléfono del usuario.

**9.1.2 RI-U-02 — Aplicación móvil del Repartidor (mobile_driver)**
Interfaz desarrollada en Flutter para dispositivos Android e iOS, que permite al Repartidor gestionar su disponibilidad, recibir y aceptar los pedidos que le son asignados, y registrar los hitos de la entrega.

**9.1.3 RI-U-03 — Aplicación móvil de la Tienda / Dispatcher (mobile_store)**
Interfaz desarrollada en Flutter para dispositivos Android e iOS, que permite a la Tienda / Dispatcher recibir notificaciones de nuevas órdenes, confirmar o rechazar pedidos, y realizar el seguimiento de la entrega.

**9.1.4 RI-U-04 — Panel web de administración (web_admin)**
Aplicación web desarrollada en React, que permite al Administrador gestionar pedidos, repartidores, tiendas, usuarios, la configuración de los modos de asignación, las reglas de tarifa y las comisiones.

**9.1.5 RI-U-05 — Panel web de la Tienda / Dispatcher (web_store)**
Aplicación web desarrollada en React, alternativa a la aplicación móvil correspondiente, que permite a la Tienda / Dispatcher gestionar sus pedidos y configurar su información desde un navegador.

---

### 9.2 Interfases de Hardware

**9.2.1 RI-H-01 — GPS del dispositivo del Repartidor**
La aplicación móvil del Repartidor debe acceder al GPS del dispositivo con el fin de permitir el seguimiento de su posición en tiempo real, tanto por parte del Cliente como del sistema.

**9.2.2 RI-H-02 — Conectividad de red**
Todas las aplicaciones cliente requieren conectividad a internet, ya sea mediante WiFi o datos móviles, para comunicarse con el backend. Dichas aplicaciones deben informar al usuario ante la pérdida de conectividad.

---

### 9.3 Interfases de Software

**9.3.1 RI-S-01 — API REST del backend (NestJS)**
El backend expone una API REST sobre HTTPS, la cual constituye la única fuente de verdad del negocio. Todas las aplicaciones cliente consumen esta API para leer y modificar el estado del sistema.

**9.3.2 RI-S-02 — WebSocket (Socket.io)**
El backend expone un espacio de nombres (namespace) de WebSocket, `/orders`, destinado a la actualización en tiempo real del estado de los pedidos, la asignación de repartidor y el chat entre el Repartidor y el Cliente.

**9.3.3 RI-S-03 — Firebase Authentication**
El sistema utiliza Firebase Auth para la verificación mediante código de un solo uso (OTP) por número de teléfono, en la aplicación móvil del Cliente.

---

### 9.4 Interfases de Comunicaciones

**9.4.1 RI-C-01 — Notificaciones push vía FCM**
El backend se comunica con las aplicaciones cliente mediante Firebase Cloud Messaging, para el envío de notificaciones de cambio de estado de los pedidos, de asignación de repartidor y de confirmaciones.

**9.4.2 RI-C-02 — Alertas operativas vía Telegram Bot**
El backend envía alertas operativas —pedidos lentos, pedidos huérfanos, cancelaciones— al Administrador y a las tiendas configuradas, mediante bots de Telegram. Este mecanismo requiere conectividad saliente del servidor hacia la API de Telegram.

---

## 10. Licenciamiento

Esta sección identifica las licencias que rigen los componentes de terceros incorporados al sistema.

**10.1 RL-01 — Licencias de componentes de terceros**
El sistema incorpora componentes de terceros bajo licencias de código abierto —NestJS y TypeORM bajo licencia MIT, Flutter bajo licencia BSD— así como servicios de pago por uso, entre ellos Firebase, Google Maps y Cloudinary. La operación continua del sistema exige que las licencias y suscripciones de dichos servicios externos se mantengan vigentes y dentro de los límites de uso contratados.

---

## 11. Requerimientos Legales y de Derecho de Autor

Esta sección recoge las obligaciones legales que el sistema debe observar en el tratamiento de datos personales y en la protección del software desarrollado.

**11.1 RLeg-01 — Protección de datos personales**
El sistema almacena datos personales de los usuarios, tales como nombre, teléfono, correo electrónico, dirección y coordenadas GPS. El tratamiento de dichos datos debe cumplir con la Ley N.º 29733 de Protección de Datos Personales del Perú y su reglamento, lo cual incluye el consentimiento informado del usuario al momento de su registro.

**11.2 RLeg-02 — Derechos sobre el software**
El código fuente de DeliverySuite es propiedad de sus autores. Queda prohibida su reproducción, distribución o uso comercial sin la autorización expresa de estos.

---

## 12. Estándares Aplicables

Esta sección enumera los estándares y procesos formales sobre los cuales se ha construido la documentación y el desarrollo del sistema.

**12.1 REst-01 — IEEE 830 — Especificación de Requerimientos de Software**
El presente documento sigue la estructura recomendada por el estándar IEEE 830-1998 para la especificación de requerimientos de software, adaptada a la plantilla del Rational Unified Process empleada en el curso.

**12.2 REst-02 — RUP (Rational Unified Process)**
El proceso de desarrollo del sistema sigue las prácticas del Rational Unified Process, empleando como artefactos formales de las fases de Inicio y Elaboración los documentos de Visión de Negocio, Actores del Sistema, Reglas del Negocio, Especificaciones de Caso de Uso y el presente documento de requerimientos de software.

**12.3 REst-03 — RFC 3986 — URI y endpoints REST**
Los endpoints de la API REST del backend deben seguir las convenciones de diseño REST, empleando rutas que identifiquen recursos y verbos HTTP que expresen con claridad la operación correspondiente —GET, POST, PATCH o DELETE.
