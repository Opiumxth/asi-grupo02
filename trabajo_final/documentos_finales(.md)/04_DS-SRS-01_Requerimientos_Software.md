# DeliverySuite

# Proyecto: DeliverySuite

# Especificación de Requerimientos de Software

## Versión 1.0

---

**Historial de Versiones**

| Fecha | Versión | Descripción | Autor |
|---|---|---|---|
| 29/Jun/26 | 1.0 | Primera versión — requerimientos del ciclo completo de pedido | Flores Hoyos, Mathias |
| 29/Jun/26 | 1.0 | Primera versión — requerimientos del ciclo completo de pedido | Príncipe Caballa, Aida |
| 29/Jun/26 | 1.0 | Primera versión — requerimientos del ciclo completo de pedido | Ochoa Torres, Yoant Alnor |

---

## Tabla de Contenidos

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

## 1. Funcionalidad

### 1.1 Asociados a los casos de uso

**1.1.1 RF-01 — Creación de pedido (CUS-01)**
El sistema debe permitir al Cliente autenticado crear un pedido seleccionando una tienda activa, uno o más productos con cantidad y variante, una dirección de entrega con coordenadas geográficas y un método de pago (CASH, CARD o WALLET). Al confirmar, el sistema debe registrar el pedido en estado PENDING con un snapshot inmutable de nombre, precio unitario, subtotal y variante de cada ítem, calcular y fijar la tarifa de envío, y notificar a la Tienda / Dispatcher.

**1.1.2 RF-02 — Cálculo de tarifa de envío al crear el pedido (CUS-01 / SF-01)**
El sistema debe calcular la tarifa de envío en el momento de creación del pedido aplicando el modo de cálculo activo (ZONE o DISTANCE), las reglas de tarifa configuradas en orden de prioridad, el multiplicador de tienda y el redondeo al medio sol superior. El valor calculado debe quedar fijo en el pedido y no recalcularse en estados posteriores.

**1.1.3 RF-03 — Confirmación de pedido por la Tienda / Dispatcher (CUS-02)**
El sistema debe permitir a la Tienda / Dispatcher autenticada confirmar un pedido en estado PENDING, seleccionando la sucursal de despacho e indicando el tiempo estimado de preparación en minutos. Al confirmar, el sistema debe transicionar el pedido a CONFIRMED, notificar al Cliente y activar automáticamente la búsqueda de repartidor.

**1.1.4 RF-04 — Rechazo de pedido por la Tienda / Dispatcher (CUS-03)**
El sistema debe permitir a la Tienda / Dispatcher rechazar un pedido en estado PENDING. Al rechazarlo, el sistema debe registrar el tipo de cancelación STORE\_FAULT\_BEFORE\_DRIVER\_PAID, transicionar el pedido a CANCELLED, notificar al Cliente y no generar entradas en el ledger financiero.

**1.1.5 RF-05 — Asignación automática de Repartidor (CUS-04)**
En modo AUTO\_PROXIMITY, el sistema debe seleccionar automáticamente un Repartidor que cumpla las condiciones de disponibilidad (online, dentro de su límite de pedidos simultáneos según rango, región coincidente), transicionar el pedido a DRIVER\_ASSIGNED y notificar al candidato. Si el candidato no acepta en el tiempo establecido, el sistema debe reintentar con el siguiente candidato disponible manteniendo el estado DRIVER\_ASSIGNED. Si no hay candidatos disponibles, debe generar una alerta al Administrador.

**1.1.6 RF-06 — Asignación manual de Repartidor por el Administrador (CUS-05)**
En modo ADMIN\_MANUAL, el sistema debe permitir al Administrador autenticado seleccionar un Repartidor disponible de la lista filtrada por condiciones de disponibilidad, validar su estado en tiempo real al confirmar la asignación, registrar la asignación transicionando el pedido a DRIVER\_ASSIGNED y notificar al Repartidor.

**1.1.7 RF-07 — Ejecución de entrega por el Repartidor (CUS-06)**
El sistema debe permitir al Repartidor asignado aceptar el pedido (→ ACCEPTED), registrar su llegada al local (→ AT\_STORE), registrar la recogida del pedido (→ PICKED\_UP), registrar su llegada al destino (→ AT\_CUSTOMER) y confirmar la entrega (→ DELIVERED). En cada transición el sistema debe registrar el timestamp correspondiente y notificar a los actores involucrados.

**1.1.8 RF-08 — Registro de comisiones al entregar (CUS-06 / SF-01)**
Al transicionar a DELIVERED, el sistema debe calcular y registrar en el ledger financiero la comisión de plataforma sobre la tienda según su tipo (OFFICIAL\_10, OFFICIAL\_5, NON\_OFFICIAL), el ingreso de tienda cuando el tipo es OFFICIAL\_10 y el pago es CARD, y la comisión del Repartidor calculada sobre la tarifa de envío usando la tasa resuelta por jerarquía de configuración.

**1.1.9 RF-09 — Cancelación de pedido (CUS-07)**
El sistema debe permitir la cancelación de pedidos activos según las siguientes reglas de autorización: el Cliente puede cancelar únicamente desde PENDING; el Administrador y Soporte pueden cancelar desde cualquier estado activo posterior a PENDING. La cancelación debe registrar el tipo correspondiente (CLIENT\_FAULT, STORE\_FAULT\_BEFORE\_DRIVER\_PAID, STORE\_FAULT\_AFTER\_DRIVER\_PAID o DRIVER\_FAULT), aplicar las consecuencias financieras definidas por ese tipo y notificar a todos los actores involucrados.

**1.1.10 RF-10 — Consulta de estado y seguimiento del pedido (CUS-08)**
El sistema debe permitir al Cliente autenticado consultar el estado actual de cualquiera de sus pedidos (activos o concluidos), mostrando los datos del Repartidor asignado cuando el pedido esté en estado ACCEPTED o posterior. La vista debe mantenerse actualizada mediante notificaciones FCM ante cada cambio de estado, sin requerir acción adicional del Cliente.

**1.1.11 RF-11 — Calificación del pedido (CUS-09)**
El sistema debe permitir al Cliente calificar el servicio de entrega una vez que su pedido haya alcanzado el estado DELIVERED, siempre que no haya emitido una calificación previa para ese pedido. La calificación debe aceptar una puntuación y un comentario opcional. La calificación es opcional; su omisión no debe bloquear ningún flujo posterior.

---

### 1.2 Asociados a aspectos generales

**1.2.1 RF-12 — Secuencia de estados unidireccional**
El sistema debe garantizar que los estados de un pedido avancen exclusivamente en la secuencia definida (PENDING → CONFIRMED → WAITING\_DRIVER → DRIVER\_ASSIGNED → ACCEPTED → AT\_STORE → PICKED\_UP → AT\_CUSTOMER → DELIVERED), sin permitir saltos ni reversiones, con la única excepción de la reiteración en DRIVER\_ASSIGNED durante el reintento de asignación.

**1.2.2 RF-13 — Snapshot inmutable de ítems**
El sistema debe registrar en `order_items` el nombre, precio unitario, subtotal y variante de cada ítem en el momento de creación del pedido. Este snapshot no debe modificarse ante cambios posteriores en el catálogo de la tienda.

**1.2.3 RF-14 — Modo de asignación global configurable**
El sistema debe soportar dos modos de asignación (AUTO\_PROXIMITY y ADMIN\_MANUAL) como parámetro global singleton. Solo un modo puede estar activo a la vez. El Administrador debe poder cambiar el modo activo desde el panel de administración.

**1.2.4 RF-15 — Modo de cálculo de tarifa global configurable**
El sistema debe soportar dos modos de cálculo de tarifa de envío (ZONE y DISTANCE) como parámetro global. El Administrador debe poder configurar el modo activo, los valores de zona (polígonos, tarifas base, mínimos y máximos) y las reglas de tarifa adicionales (tipo, valor, filtros de ciudad, día y horario).

**1.2.5 RF-16 — Control de disponibilidad del Repartidor**
El sistema debe permitir al Repartidor activar y desactivar su estado de disponibilidad (online/offline). El sistema debe impedir la asignación de pedidos a Repartidores que no cumplan simultáneamente: estar online, no haber alcanzado su límite de pedidos simultáneos según rango y tener la región coincidente con el pedido.

**1.2.6 RF-17 — Notificaciones en tiempo real a todos los actores**
El sistema debe enviar notificaciones push (FCM) a los actores correspondientes en cada transición de estado del pedido: al Cliente ante confirmación, asignación de repartidor, recogida, llegada al destino, entrega y cancelación; a la Tienda / Dispatcher ante nueva orden, llegada del repartidor y entrega; al Repartidor ante asignación y cancelación posterior a su asignación.

**1.2.7 RF-18 — Alertas operativas al Administrador**
El sistema debe generar alertas automáticas al Administrador (vía Telegram) ante las siguientes situaciones: pedido en CONFIRMED sin avanzar más del tiempo configurado, pedido en PICKED\_UP sin completarse en el tiempo esperado, pedido en WAITING\_DRIVER sin repartidor asignado (pedido huérfano) y cancelaciones no resueltas.

---

## 2. Usabilidad

**2.1 RU-01 — Retroalimentación inmediata ante acciones del usuario**
El sistema debe mostrar al usuario una respuesta visual inmediata ante cada acción significativa (creación de pedido, confirmación, cambio de estado), sin tiempos de espera superiores a los percibidos como normales en aplicaciones móviles de delivery.

**2.2 RU-02 — Flujo de creación de pedido en tres pasos**
El flujo de creación de pedido en la aplicación del Cliente debe completarse en no más de tres pasos principales: selección de productos, configuración de entrega (dirección y método de pago) y confirmación. La tarifa de envío debe mostrarse al Cliente antes de la confirmación final.

**2.3 RU-03 — Visibilidad del estado del pedido sin navegación adicional**
El estado actual del pedido y la información del Repartidor asignado (cuando aplique) deben ser accesibles desde la pantalla principal de pedidos activos del Cliente, sin requerir navegación a submenús.

---

## 3. Confiabilidad

**3.1 RC-01 — Disponibilidad del servicio**
El sistema debe estar disponible para los actores operativos (Cliente, Tienda / Dispatcher, Repartidor) durante el horario de operación de las tiendas afiliadas, con una disponibilidad objetivo de 99 % en ese período.

**3.2 RC-02 — Consistencia del estado del pedido**
El sistema debe garantizar que las transiciones de estado de un pedido sean atómicas: ante un fallo en mitad de una transición, el pedido debe permanecer en su estado anterior sin registrar estados intermedios inválidos.

**3.3 RC-03 — Consistencia del ledger financiero**
El registro de comisiones en el ledger financiero debe ejecutarse de forma atómica con la transición a DELIVERED. Un fallo durante ese registro no debe dejar el pedido en DELIVERED sin sus entradas financieras correspondientes.

---

## 4. Rendimiento

**4.1 RP-01 — Tiempo de respuesta en creación de pedido**
La creación de un pedido, incluyendo el cálculo de tarifa de envío y la notificación a la Tienda / Dispatcher, debe completarse en menos de 3 segundos bajo condiciones normales de carga.

**4.2 RP-02 — Tiempo de respuesta en transiciones de estado**
Cada transición de estado de un pedido (confirmación, asignación, hitos de entrega) debe procesarse y notificarse a los actores involucrados en menos de 2 segundos bajo condiciones normales de carga.

**4.3 RP-03 — Capacidad de pedidos simultáneos**
El sistema debe soportar la operación simultánea de múltiples pedidos activos en distintos estados sin degradación perceptible del rendimiento para ningún actor.

---

## 5. Soporte

**5.1 RS-01 — Registro de auditoría de transiciones**
Toda transición de estado de un pedido debe quedar registrada en `order_events` con el estado anterior, el estado nuevo, el actor que la ejecutó (o SYSTEM si es automática) y el timestamp. Este registro es inmutable y no puede ser modificado ni eliminado por ningún actor.

**5.2 RS-02 — Trazabilidad de cancelaciones**
Toda cancelación debe registrar el tipo de cancelación (`cancellation_type`) y el motivo textual (`cancellation_reason`), vinculados al pedido, para permitir la revisión posterior por parte del Administrador o Soporte.

**5.3 RS-03 — Configuración operativa sin redespliegue**
Los parámetros operativos principales (modo de asignación, modo de cálculo de tarifa, reglas de tarifa, configuración de comisiones) deben poder modificarse desde el panel de administración sin necesidad de redespliegue del sistema.

---

## 6. Restricciones de Diseño

**6.1 RD-01 — API REST como interfaz principal del backend**
Toda la lógica de negocio del sistema debe estar centralizada en el backend (API REST sobre NestJS). Las aplicaciones cliente (móvil y web) no deben replicar lógica de negocio; deben consumir los endpoints del backend para toda operación que modifique estado.

**6.2 RD-02 — Zona horaria fija America/Lima**
Todos los timestamps registrados por el sistema deben utilizar la zona horaria America/Lima como referencia operativa, independientemente de la ubicación geográfica del actor que genere la acción.

**6.3 RD-03 — Sin triggers nativos de base de datos**
Toda la lógica de automatización (transiciones de estado, cálculos, notificaciones, cron jobs de monitoreo) debe implementarse en la capa de aplicación (NestJS Services). No se deben usar triggers nativos de PostgreSQL para lógica de negocio.

**6.4 RD-04 — Autenticación mediante JWT**
Todos los endpoints que modifiquen estado o accedan a datos sensibles deben requerir un token JWT válido firmado por el backend. El control de acceso por rol debe aplicarse mediante guards en cada endpoint según el rol del actor (CLIENT, DRIVER, VENDOR, ADMIN, SUPPORT).

---

## 7. Documentación de Usuario y Sistema de Ayuda

**7.1 RDoc-01 — Manual de operación para la Tienda / Dispatcher**
El sistema debe acompañarse de una guía de operación para la Tienda / Dispatcher que describa el flujo de gestión de pedidos: recepción de notificaciones, confirmación, selección de sucursal y seguimiento hasta la entrega.

**7.2 RDoc-02 — Guía de inicio para el Repartidor**
El sistema debe incluir una guía de inicio para el Repartidor que explique la activación del estado online, la recepción y aceptación de pedidos, y el registro de cada hito de la entrega.

---

## 8. Componentes Adquiridos

**8.1 RCA-01 — Firebase Cloud Messaging (FCM)**
El sistema utiliza Firebase Cloud Messaging para el envío de notificaciones push a las aplicaciones móviles y web de todos los actores. Este componente es provisto por Google Firebase y debe estar configurado con las credenciales correspondientes al proyecto.

**8.2 RCA-02 — Servicio de geolocalización y mapas**
El sistema utiliza la API de Google Maps en las aplicaciones móviles para la visualización del mapa de seguimiento del Repartidor y la selección de direcciones de entrega. Su disponibilidad está sujeta a los términos de uso de Google Maps Platform.

**8.3 RCA-03 — Almacenamiento de imágenes (Cloudinary)**
El sistema utiliza Cloudinary para el almacenamiento y servicio de imágenes de tiendas, productos y perfiles de usuarios. La disponibilidad de imágenes depende de la disponibilidad del servicio externo.

---

## 9. Interfases

### 9.1 Interfases de Usuarios

**9.1.1 RI-U-01 — Aplicación móvil del Cliente (mobile_customer)**
Interfaz Flutter para dispositivos iOS y Android que permite al Cliente explorar tiendas, crear pedidos, consultar el estado en tiempo real y calificar entregas. Requiere autenticación por OTP (teléfono).

**9.1.2 RI-U-02 — Aplicación móvil del Repartidor (mobile_driver)**
Interfaz Flutter para dispositivos Android e iOS que permite al Repartidor gestionar su disponibilidad, recibir y aceptar pedidos asignados, y registrar los hitos de la entrega.

**9.1.3 RI-U-03 — Aplicación móvil de la Tienda / Dispatcher (mobile_store)**
Interfaz Flutter para dispositivos Android e iOS que permite a la Tienda / Dispatcher recibir notificaciones de nuevas órdenes, confirmar o rechazar pedidos, y hacer seguimiento de la entrega.

**9.1.4 RI-U-04 — Panel web de administración (web_admin)**
Aplicación web React que permite al Administrador gestionar pedidos, repartidores, tiendas, usuarios, configuración de modos de asignación, reglas de tarifa y comisiones.

**9.1.5 RI-U-05 — Panel web de la Tienda / Dispatcher (web_store)**
Aplicación web React alternativa a la app móvil para que la Tienda / Dispatcher gestione pedidos y configure su información desde un navegador.

---

### 9.2 Interfases de Hardware

**9.2.1 RI-H-01 — GPS del dispositivo del Repartidor**
La aplicación móvil del Repartidor debe acceder al GPS del dispositivo para permitir el seguimiento de su posición en tiempo real por parte del Cliente y el sistema.

**9.2.2 RI-H-02 — Conectividad de red**
Todas las aplicaciones cliente requieren conectividad a internet (WiFi o datos móviles) para comunicarse con el backend. Las aplicaciones deben informar al usuario ante la pérdida de conectividad.

---

### 9.3 Interfases de Software

**9.3.1 RI-S-01 — API REST del backend (NestJS)**
El backend expone una API REST sobre HTTPS que constituye la única fuente de verdad del negocio. Todas las aplicaciones cliente consumen esta API para leer y modificar el estado del sistema.

**9.3.2 RI-S-02 — WebSocket (Socket.io)**
El backend expone un namespace de WebSocket (`/orders`) para la actualización en tiempo real del estado de pedidos, la asignación de repartidor y el chat entre Repartidor y Cliente.

**9.3.3 RI-S-03 — Firebase Authentication**
El sistema utiliza Firebase Auth para la verificación OTP por número de teléfono en la aplicación móvil del Cliente.

---

### 9.4 Interfases de Comunicaciones

**9.4.1 RI-C-01 — Notificaciones push vía FCM**
El backend se comunica con las aplicaciones cliente mediante Firebase Cloud Messaging para el envío de notificaciones de cambio de estado de pedidos, asignación de repartidor y confirmaciones.

**9.4.2 RI-C-02 — Alertas operativas vía Telegram Bot**
El backend envía alertas operativas (pedidos lentos, pedidos huérfanos, cancelaciones) al Administrador y a las tiendas configuradas mediante bots de Telegram. Requiere conectividad saliente del servidor hacia la API de Telegram.

---

## 10. Licenciamiento

**10.1 RL-01 — Licencias de componentes de terceros**
El sistema incorpora componentes de terceros bajo licencias open source (NestJS / MIT, TypeORM / MIT, Flutter / BSD) y servicios de pago por uso (Firebase, Google Maps, Cloudinary). La operación del sistema requiere que las licencias y suscripciones de los servicios externos estén vigentes y dentro de los límites de uso contratados.

---

## 11. Requerimientos Legales y de Derecho de Autor

**11.1 RLeg-01 — Protección de datos personales**
El sistema almacena datos personales de los usuarios (nombre, teléfono, correo, dirección, coordenadas GPS). El tratamiento de estos datos debe cumplir con la Ley N.º 29733 de Protección de Datos Personales del Perú y su reglamento, incluyendo el consentimiento informado del usuario al registrarse.

**11.2 RLeg-02 — Derechos sobre el software**
El código fuente de DeliverySuite es propiedad de sus autores. Queda prohibida su reproducción, distribución o uso comercial sin autorización expresa.

---

## 12. Estándares Aplicables

**12.1 REst-01 — IEEE 830 — Especificación de Requerimientos de Software**
Este documento sigue la estructura recomendada por el estándar IEEE 830-1998 para la especificación de requerimientos de software, adaptada a la plantilla RUP del curso.

**12.2 REst-02 — RUP (Rational Unified Process)**
El proceso de desarrollo sigue las prácticas del Rational Unified Process, con documentos de Visión de Negocio, Actores del Sistema, Reglas del Negocio, Casos de Uso y este SRS como artefactos formales de la fase de Inicio/Elaboración.

**12.3 REst-03 — RFC 3986 — URI y endpoints REST**
Los endpoints de la API REST del backend deben seguir las convenciones de diseño REST, con rutas que identifiquen recursos y verbos HTTP que expresen la operación (GET, POST, PATCH, DELETE).
