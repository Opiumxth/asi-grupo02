# Contenido para Exposición — DeliverySuite

Duración objetivo: 15–20 min. ~14 slides (≈ 1–1.5 min por slide).
Fuente: los 5 documentos `.md` (DS-ACT-01, DS-RN-01, DS-CUS-01-09, DS-SRS-01, DS-ARQ-01) + `DeliverySuite.mdl`.

Cada bloque de abajo = 1 slide. Contenido en bullets, listo para pasar a PowerPoint/Slides.

---

## Slide 1 — Portada

- **Proyecto:** DeliverySuite
- **Integrantes:**
  - Flores Hoyos, Mathias
  - Príncipe Caballa, Aida
  - Ochoa Torres, Yoant Alnor
- **Curso:** Análisis de Sistemas de Información *(ajustar nombre exacto/código si difiere)*
- **Fecha de exposición:** [completar]

---

## Slide 2 — Contexto: ¿Qué es DeliverySuite?

- Plataforma de **delivery a domicilio**: conecta Cliente, Tienda y Repartidor.
- Cubre el **ciclo completo del pedido**: creación → confirmación → asignación → entrega → cancelación/calificación.
- Inspirada en una plataforma real: **Musuq V2** (mismo modelo de datos y roles: `VENDOR`, `DRIVER`, `ADMIN`, `SUPPORT`).
- Arquitectura: backend único (NestJS) + apps móviles (Flutter) + web (React).

---

## Slide 3 — Alcance del Proyecto

**Incluye:**
- Ciclo completo del pedido: creación, confirmación/rechazo, asignación (auto/manual), entrega, cancelación, calificación
- Cálculo de tarifa de envío (ZONE / DISTANCE) y de comisiones
- Seguimiento del pedido en tiempo real

**Fuera de alcance** *(exclusiones definidas en DS-ARQ-01):*
- Cupones y descuentos promocionales
- Billetera / saldo prepagado (wallet)
- Programa de referidos
- Pedidos B2B por canales externos
- Múltiples sucursales como unidades de negocio independientes
- Operación simultánea en múltiples países

---

## Slide 4 — Actores del Sistema (6)

- **Cliente** — crea el pedido, hace seguimiento, puede cancelar en PENDING y califica la entrega.
- **Tienda / Dispatcher** (`VENDOR`) — confirma o rechaza pedidos y gestiona su catálogo/sucursales.
- **Repartidor** (`DRIVER`) — recoge y entrega el pedido, avanzando por los estados operativos.
- **Administrador** (`ADMIN`) — configura el sistema, asigna manualmente, cancela y gestiona tiendas/repartidores.
- **Soporte** (`SUPPORT`) — cancela pedidos en curso ante incidencias (disputas, pedidos estancados).
- **Sistema de Asignación** — actor automatizado que asigna repartidor por proximidad.

---

## Slide 5 — Reglas del Negocio más relevantes (6 de 18)

- **RN-02** — Secuencia de estados estricta y unidireccional (PENDING → … → DELIVERED / CANCELLED).
- **RN-05** — El tipo de cancelación define el tratamiento financiero (CLIENT_FAULT, STORE_FAULT_*, DRIVER_FAULT).
- **RN-07** — Límite de pedidos simultáneos según el rango del repartidor (BRONZE=1 … LEGEND=5).
- **RN-08** — Modo de asignación (AUTO_PROXIMITY / ADMIN_MANUAL) es global, solo lo cambia el Administrador.
- **RN-14** — Tarifa final = tarifa × factor de tienda, redondeada al medio sol superior.
- **RN-16** — Comisión del repartidor por jerarquía: individual → ciudad → global → valor de reserva.

---

## Slide 6 — Casos de Uso del Sistema (9)

- **CUS-01** Crear Pedido
- **CUS-02** Confirmar Pedido
- **CUS-03** Rechazar Pedido
- **CUS-04** Asignar Repartidor Automáticamente
- **CUS-05** Asignar Repartidor Manualmente
- **CUS-06** Ejecutar Entrega
- **CUS-07** Cancelar Pedido
- **CUS-08** Consultar Estado / Seguimiento del Pedido
- **CUS-09** Calificar Pedido

**Resumen visual — ciclo de vida del pedido:**

```
PENDING → CONFIRMED → WAITING_DRIVER → DRIVER_ASSIGNED → ACCEPTED
        → AT_STORE → PICKED_UP → AT_CUSTOMER → DELIVERED   (o → CANCELLED)
```

*(En vivo: mostrar el diagrama de estados real — Slide 11, "Ciclo de vida del Pedido").*

---

## Slide 7 — Detalle: CUS-01 Crear Pedido

- Cliente selecciona tienda activa + productos (cantidad, variante).
- Indica dirección de entrega (coordenadas) y método de pago.
- Sistema calcula la tarifa de envío (ZONE/DISTANCE + reglas + factor tienda + redondeo).
- Sistema valida condiciones mínimas (RN-01) y registra el pedido en **PENDING** con snapshot de precios (RN-18).
- Sistema notifica a la Tienda / Dispatcher.

---

## Slide 8 — Detalle: CUS-04 Asignación Automática de Repartidor

- Se activa al llegar a **WAITING_DRIVER** con modo AUTO_PROXIMITY.
- Filtra repartidores: online, dentro de su límite (RN-07) y misma región (RN-06).
- Selecciona candidato → **DRIVER_ASSIGNED** → notificación.
- Candidato acepta → **ACCEPTED**; si no acepta a tiempo, reintenta con el siguiente.
- Sin candidatos disponibles → alerta de "pedido huérfano" al Administrador.

---

## Slide 9 — Arquitectura 4+1 (I): Casos de Uso + Vista Lógica

- **Vista de Casos de Uso (+1):** los 9 CUS validan la arquitectura — ej. CUS-04/05 exigen aislar la asignación en un servicio propio; CUS-06 exige atomicidad entre entrega y comisiones.
- **Vista Lógica:** **Pedido** (entidad central) — Cliente, Tienda, Repartidor — Ítem de Pedido (snapshot inmutable) — Evento de Pedido (auditoría).

---

## Slide 10 — Arquitectura 4+1 (II): Procesos, Desarrollo y Física

- **Vista de Procesos:** ciclo del pedido síncrono y transaccional + asignación automática asíncrona (con reintentos) + tiempo real vía FCM y WebSocket.
- **Vista de Desarrollo:** backend NestJS único + apps Flutter (cliente/repartidor/tienda) + apps React (admin/tienda); módulos clave: pedidos, asignación, tarifas, comisiones.
- **Vista Física:** servidor API ↔ PostgreSQL, más servicios externos (FCM, Google Maps, Telegram).

---

## Slide 11 — Modelo Rational Rose: diagramas en `DeliverySuite.mdl`

*(nombres exactos del modelo — usar estos títulos al abrir el .mdl en vivo)*

- **Casos de Uso:** `DCU-01`
- **Estados:** `Ciclo de vida del Pedido`
- **Clases:** `DC-01`
- **Secuencia:** `DS-01` Crear Pedido · `DS-02` Confirmar + Asignación · `DS-03` Ejecutar Entrega · `DS-04` Cancelar Pedido
- **Componentes:** `DCOMP-01`
- **Despliegue:** `Deployment View`

*(el resto de diagramas del modelo son plantillas RUP por defecto, vacías — no se muestran)*

---

## Slide 12 — Prototipo

*(completar con lo realmente implementado; no se encontró evidencia de prototipo en los documentos entregados — usar como guía DS-SRS-01 §9.1)*

- App móvil Cliente — [pantallas/flujos implementados]
- App móvil Repartidor — [pantallas/flujos implementados]
- App móvil Tienda — [pantallas/flujos implementados]
- Panel web Admin — [pantallas/flujos implementados]

---

## Slide 13 — Conclusiones

- DeliverySuite documenta un ciclo de pedido completo y consistente, de extremo a extremo (9 CUS, 18 RN, 18 RF).
- El modelo de estados unidireccional (RN-02) es la columna vertebral de todo el diseño.
- La arquitectura 4+1 separa negocio (backend) de presentación (apps cliente), y aísla asignación/tarifas/comisiones en servicios de dominio independientes.
- El modelo UML (Rational Rose) complementa la documentación textual con los diagramas reales listados en la Slide 11.

---

## Slide 14 — Trabajo Futuro

*(explícitamente fuera de alcance, según DS-ARQ-01 §8)*

- Cupones y descuentos promocionales
- Billetera / saldo (wallet)
- Programa de referidos
- Pedidos B2B por canales externos
- Múltiples sucursales como unidades de negocio independientes
- Operación multi-país
