# Arquitectura Core — Musuq V2
> ⚠️ **ACTUALIZACIÓN 2026-06:** las liquidaciones fueron ELIMINADAS y reemplazadas por **billeteras** (`backend/src/wallets/`). Ver `FINANZAS_BILLETERAS.md`. Las menciones a finance/settlements/deposits en este documento son históricas.


> Plataforma de delivery de comida y courier multi-app, multi-país.  
> Monorepo con 7 aplicaciones: 1 backend, 3 móviles Flutter, 3 webs React.

---

## 1. Stack Tecnológico

### Backend
| Capa | Tecnología | Versión |
|---|---|---|
| Framework | NestJS | 11.0.1 |
| Lenguaje | TypeScript | 5.7.3 |
| ORM | TypeORM | 0.3.28 |
| Base de datos | PostgreSQL | (via Railway) |
| WebSockets | Socket.io + NestJS Gateway | 4.8.3 |
| Autenticación | JWT + Passport + Firebase Auth | — |
| Push Notifications | Firebase Admin SDK (FCM) | 13.6.0 |
| Imágenes | Cloudinary | 2.9.0 |
| Alertas operativas | Telegram Bot API (axios) | — |
| Geoespacial | TurfJS | 7.3.4 |
| Logger | Pino | 10.3.1 |
| Timezone | America/Lima (hardcoded en main.ts) | — |

### Aplicaciones Móviles (Flutter)
| App | Paquete | Versión |
|---|---|---|
| mobile_customer | com.musuq.app | 1.0.6+13 |
| mobile_driver | com.musuq.driver | 1.0.4+4 |
| mobile_store | com.musuq.store | 1.0.4+4 |

**Dependencias compartidas Flutter:**
- `flutter_riverpod` — State management (providers + notifiers)
- `dio` — HTTP client con interceptores JWT
- `firebase_core` + `firebase_messaging` — FCM push notifications
- `firebase_auth` — Auth OTP por teléfono
- `socket_io_client` — WebSocket para actualizaciones en tiempo real
- `geolocator` + `google_maps_flutter` — GPS y mapas
- `shared_preferences` + `flutter_secure_storage` — Persistencia local
- `permission_handler` — Permisos de sistema

### Aplicaciones Web (React + Vite)
| App | Propósito | Estado |
|---|---|---|
| web_customer | Tienda pública para clientes | SPA con React Router DOM 7 |
| web_admin | Panel admin monolítico | Single `App.tsx` (433KB) |
| web_store | Panel de gestión para locales | Single `App.tsx` (110KB) |

**Stack web:** React 18.3.1 · TypeScript 5.6.2 · Vite 5.4.10 · socket.io-client 4.8.3 · Firebase 12.10.0

---

## 2. Ciclo de Vida de un Pedido

```
CLIENTE                    BACKEND                      LOCAL (VENDOR)             MOTORIZADO
  │                           │                               │                         │
  │── POST /orders ──────────>│                               │                         │
  │   (items, coords,         │ Crea Order{status:PENDING}    │                         │
  │    payment, coupon)       │ Calcula fee + comisión        │                         │
  │                           │ Guarda OrderItems (snapshot)  │                         │
  │<── Order creada ──────────│                               │                         │
  │                           │── FCM + Telegram ────────────>│                         │
  │                           │   (nueva orden para el local) │                         │
  │                           │                               │                         │
  │                           │                    [Local revisa la orden]               │
  │                           │<── PATCH /orders/:id/status ──│                         │
  │                           │    {status: CONFIRMED,        │                         │
  │                           │     storeLocationId,          │                         │
  │                           │     estimatedPrepTime}        │                         │
  │                           │ status → CONFIRMED            │                         │
  │<── FCM (confirmado) ──────│                               │                         │
  │                           │                               │                         │
  │                           │  [Modo AUTO_PROXIMITY]        │                         │
  │                           │  AssignmentService busca       │                         │
  │                           │  driver disponible por zona    │                         │
  │                           │  status → WAITING_DRIVER       │                         │
  │                           │── FCM broadcast drivers ─────────────────────────────>  │
  │                           │   (orden disponible)           │                        │
  │                           │                                │                        │
  │                           │<── POST /orders/:id/accept ─────────────────────────── │
  │                           │ status → ACCEPTED              │                        │
  │                           │ driverId asignado              │                        │
  │                           │ Socket emit order.driver_assigned                       │
  │<── FCM (driver en camino)─│                                │                        │
  │                           │                                │                        │
  │                           │<── POST /orders/:id/arrive-store ──────────────────────│
  │                           │ status → AT_STORE              │                        │
  │                           │── FCM (driver en local) ──────>│                        │
  │                           │                                │                        │
  │                           │<── POST /orders/:id/pickup ─────────────────────────── │
  │                           │ status → PICKED_UP             │                        │
  │                           │ pickedUpAt = NOW()             │                        │
  │<── FCM (pedido recogido) ─│                                │                        │
  │                           │                                │                        │
  │                           │<── POST /orders/:id/at-customer ───────────────────────│
  │                           │ status → AT_CUSTOMER           │                        │
  │<── FCM (driver llegó) ────│                                │                        │
  │                           │                                │                        │
  │                           │<── POST /orders/:id/complete ──────────────────────────│
  │                           │ status → DELIVERED             │                        │
  │                           │ deliveredAt = NOW()            │                        │
  │                           │ Calcula comisión driver        │                        │
  │                           │ Actualiza wallet si aplica     │                        │
  │<── FCM (entregado) ───────│                                │                        │
  │<── Prompt rating ─────────│                                │                        │
  │                           │── Telegram alert ─────────────>│ (entregado)            │
  │                           │                                │                        │
```

### Estados completos del Order (enum `OrderStatus`)

```
PENDING
  └── CONFIRMED ──────────────────────────────────────── (local acepta)
        └── WAITING_DRIVER ──────────────────────────── (esperando motorizado)
              └── DRIVER_ASSIGNED ─────────────────────  (asignado automático)
                    └── ACCEPTED ──────────────────────  (driver acepta)
                          └── AT_STORE ────────────────  (driver en local)
                                └── PICKED_UP ──────── (recogido)
                                      └── AT_CUSTOMER  (en destino)
                                            └── DELIVERED ✓

Flujo legacy (aún en uso en web_store):
  CONFIRMED → PREPARING → READY → ON_THE_WAY → DELIVERED

Terminal negativo:
  CANCELLED (desde PENDING por cliente; resto por admin/support)
  COMPLETED (alias de DELIVERED en algunos contextos)
```

### Modos de Asignación de Driver

| Modo | Lógica |
|---|---|
| `AUTO_PROXIMITY` (Voladera) | `AssignmentService` busca driver online + libre + región coincidente, envía FCM broadcast |
| `ADMIN_MANUAL` | Admin usa `POST /orders/:id/assign-driver` desde web_admin |

El modo activo se guarda en la entidad `Setting` y se puede cambiar desde Telegram con `/voladera_on` / `/voladera_off`.

---

## 3. Estructura de Carpetas y Responsabilidades

### Monorepo raíz

```
musuq_v2/
├── backend/                  # API NestJS — única fuente de verdad de negocio
├── mobile_customer/          # App Flutter — clientes finales
├── mobile_driver/            # App Flutter — motorizados
├── mobile_store/             # App Flutter — gestión de local (vendor)
├── web_customer/             # React SPA — clientes vía web
├── web_admin/                # React SPA — panel de administración
├── web_store/                # React SPA — panel de local vía web
└── whatsapp-delivery-bot-local/  # Bot WhatsApp para órdenes B2B (local dev)
```

---

### Backend `/backend/src/`

```
src/
├── main.ts                   # Bootstrap: CORS, ValidationPipe, Pino, timezone Lima
├── app.module.ts             # Raíz: importa todos los módulos
│
├── auth/                     # JWT auth, OTP SMS/WhatsApp, device ban, roles guard
│   ├── entities/
│   │   ├── verification-code.entity.ts
│   │   ├── banned-device.entity.ts
│   │   └── wa-otp-session.entity.ts
│   ├── security.service.ts   # Límite cuentas/dispositivo, ban check
│   ├── auth.service.ts       # registerManual, login, OTP verify
│   └── jwt-auth.guard.ts
│
├── users/                    # CRUD usuarios, wallet, deuda, referidos
│   └── entities/user.entity.ts  # Roles: CLIENT, DRIVER, VENDOR, ADMIN, SUPPORT
│
├── drivers/                  # Perfil driver, rank, comisión, stats, historial
│   ├── entities/driver.entity.ts  # Rank: BRONZE→LEGEND, financialStatus
│   ├── drivers.service.ts    # update() acepta name/imageUrl para parchear user
│   └── drivers.controller.ts # GET /:id/orders-by-day, GET /:id/stats
│
├── stores/                   # CRUD tiendas, sucursales, horarios, alertas
│   ├── entities/store.entity.ts   # telegramChatIds[], waAlertNumbers[], categoryOrder
│   └── entities/store-location.entity.ts  # Sucursales físicas
│
├── products/                 # CRUD productos, variantes, adicionales
│
├── orders/                   # Núcleo del negocio
│   ├── entities/
│   │   ├── order.entity.ts       # 40+ campos, status enum completo
│   │   ├── order-item.entity.ts  # Snapshot inmutable de producto
│   │   └── order-event.entity.ts # Audit trail de transiciones
│   ├── orders.service.ts     # 65KB — todo el lifecycle
│   ├── assignment.service.ts # Lógica de asignación AUTO/MANUAL
│   ├── order-audit.service.ts
│   ├── order.gateway.ts      # WebSocket /orders namespace, rooms por rol
│   └── orders.controller.ts  # ~30 endpoints REST
│
├── notifications/            # FCM push via Firebase Admin
│   ├── entities/device-token.entity.ts
│   └── notifications.service.ts  # sendToUser, sendToDriversByRegion, broadcast
│
├── telegram-alerts/          # 2 bots Telegram + alertas automáticas
│   └── telegram-alerts.service.ts  # Cron jobs: slow store/delivery, orphaned orders
│
├── payments/                 # Registro de pagos, liquidaciones semanales
├── wallets/                  # Billeteras: saldos, movimientos, tickets de depósito
├── coupons/                  # Códigos de descuento (FREE_DELIVERY, PERCENT, FIXED)
├── couriers/                 # Servicio courier puro (no comida) + zonas TurfJS
├── deposits/                 # Recargas de wallet
├── addresses/                # Direcciones guardadas del cliente
├── favorites/                # Favoritos (tiendas/productos)
├── ratings/                  # Calificaciones post-entrega
├── chat/                     # Mensajería driver-cliente
├── offers/                   # Ofertas promocionales
├── reports/                  # SystemEvent, analytics
├── predictive-analytics/     # Predicción de demanda
├── store-suggestions/        # Recomendaciones de tiendas
├── store-verticals/          # Categorías/verticales (RESTAURANT, etc.)
├── version-check/            # Control de versión mínima de apps
├── control/                  # Endpoints de control admin
├── settings/                 # Setting global (AssignmentMode), CommissionConfig,
│                             # DeliveryFeeRule, PromoBanner
└── config/                   # Firebase init, geo (Country/Region/City),
                              # PricingMap/Sector, ZonePricingRule, uploads (Cloudinary)
```

---

### Mobile Customer `/mobile_customer/lib/`

```
lib/
├── main.dart                     # Entry point, GoRouter, Riverpod, Firebase init
├── core/
│   ├── services/api_service.dart # Dio client, JWT interceptor, todos los endpoints
│   ├── models/                   # Order, Product, CartItem, UserAddress, etc.
│   ├── providers/                # Riverpod global providers
│   └── utils/                    # Formatters, helpers
└── features/                     # Módulos de feature (screens + state local)
    ├── auth/                     # Login (PIN), Register, OTP (SMS + WhatsApp Reverse)
    ├── home/                     # Explorar tiendas, banner promos
    ├── store_detail/             # Detalle tienda, productos por categoría
    ├── cart/                     # Carrito, cupón, fee, dirección, pago
    │   ├── providers/cart_provider.dart
    │   ├── screens/cart_screen.dart   # Recalcula cupón al cambiar dirección
    │   └── state/cart_state.dart
    ├── orders/                   # Historial, detalle, tracking, cancelar, WA tienda
    ├── payment/                  # Métodos de pago (YAPE, PLIN, CASH)
    ├── account/                  # Perfil, wallet, deuda, referidos
    ├── addresses/                # CRUD direcciones guardadas
    ├── chat/                     # Mensajería con driver
    ├── notifications/            # FCM local notifications handler
    └── ...                       # (53 carpetas de features en total)
```

---

### Mobile Driver `/mobile_driver/lib/`

```
lib/
├── main.dart
├── providers.dart                # Todos los Riverpod providers centralizados
│   ├── authProvider             # AuthNotifier (login, logout, JWT)
│   ├── driverOnlineProvider     # Toggle online/offline
│   ├── availableOrdersProvider  # Pedidos en WAITING_DRIVER para su región
│   ├── myActiveOrdersProvider   # Pedidos asignados al driver
│   ├── driverProfileProvider    # Rank, score, comisión, financialStatus
│   └── myWalletProvider         # Billetera (saldo + movimientos)
├── core/
│   └── services/
│       ├── api_service.dart      # Dio client + todos los endpoints del driver
│       └── notification_service.dart  # Firebase Messaging + local notifications
└── screens/
    ├── home_screen.dart          # IndexedStack con 4 tabs:
    │   ├── Tab 0: Pedidos disponibles (WAITING_DRIVER)
    │   ├── Tab 1: Mis entregas activas (polling 30s + socket)
    │   ├── Tab 2: Ganancias / Billetera
    │   └── Tab 3: Perfil (foto, rank, score, estado financiero)
    ├── active_delivery_screen.dart   # Pantalla de entrega activa con mapa
    ├── active_orders_screen.dart     # Lista de órdenes activas (multi-orden)
    ├── order_detail_screen.dart      # Detalle + acciones (accept, arrive, pickup, complete)
    ├── login_screen.dart
    └── chat_screen.dart
```

**Sockets del driver:**
- `order.accepted` → invalida `availableOrdersProvider`
- `order.driver_assigned` → invalida `availableOrdersProvider`
- `order.taken` → elimina orden de lista + SnackBar aviso
- `order.location_corrected` → invalida `myActiveOrdersProvider` + SnackBar sucursal nueva

---

### Mobile Store `/mobile_store/lib/`

```
lib/
├── main.dart
├── providers.dart
├── api/api_service.dart          # Dio client para vendor endpoints
├── config/
└── screens/
    └── dashboard/
        ├── home_screen.dart          # Tabs: Pedidos | Productos | Billetera | Perfil
        ├── order_detail_screen.dart  # Confirmar, rechazar, cambiar estado, corregir sucursal
        │                             # (botón solo si hay ≥2 sucursales activas)
        ├── edit_order_screen.dart
        └── chat_screen.dart
```

---

### Web Apps `/web_*/src/`

```
src/
├── main.tsx              # Entry point React + Vite
├── App.tsx               # Componente raíz (monolítico en admin/store)
├── index.css             # Variables CSS globales, responsive mobile
├── api.ts                # fetch wrapper con JWT + BASE_URL
├── socket.ts             # socket.io-client singleton
├── firebase.ts           # Firebase config (auth + messaging)
├── useNotifications.ts   # Hook FCM (permisos + token registration)
├── deviceFingerprint.ts  # Fingerprint para detección de fraude (web_customer)
├── components/           # Componentes reutilizables
├── pages/                # Páginas (web_customer usa React Router)
└── layouts/              # Layouts (web_customer)
```

**web_admin** es completamente monolítico: todo el estado, lógica y UI vive en `App.tsx`. Secciones principales: Pedidos, Drivers, Tiendas, Usuarios, Productos, Cupones, Banners, Configuración, Analytics, Finanzas (Billeteras).

---

## 4. Integraciones Clave

| Servicio | Módulo backend | Propósito |
|---|---|---|
| **Firebase FCM** | `notifications/` | Push a clientes, drivers, vendors (por userId, región, broadcast) |
| **Firebase Auth** | `auth/` | Verificación OTP por teléfono (mobile_customer) |
| **Cloudinary** | `config/uploads` | Almacenamiento de imágenes (tiendas, productos, avatars) |
| **Telegram Bot (alertas)** | `telegram-alerts/` | Alertas operativas: lentitud, cancelaciones, bloqueos, modo asignación |
| **Telegram Bot (órdenes)** | `telegram-alerts/` | Notifica nuevas órdenes a tiendas manuales |
| **Socket.io Gateway** | `orders/order.gateway.ts` | Real-time: actualizaciones de pedido, chat, asignación de driver |
| **TurfJS** | `couriers/` + `config/` | Cálculo de tarifas por zonas geográficas (booleanPointInPolygon) |
| **Google Maps** | mobile apps | Visualización de mapa, tracking driver en tiempo real |
| **WhatsApp (bot local)** | `whatsapp-delivery-bot-local/` | Órdenes B2B vía WhatsApp (`POST /orders/b2b/dispatch`) |

---

## 5. Seguridad y Control de Acceso

| Mecanismo | Implementación |
|---|---|
| Auth | JWT firmado con `JWT_SECRET`, validado en `JwtAuthGuard` |
| Roles | `@Roles(UserRole.X)` + `RolesGuard` en todos los endpoints sensibles |
| Device ban | `SecurityService.checkDeviceBanned(deviceId)` en registro |
| Cuenta limit | Máximo 5 cuentas por `deviceId` (`MAX_ACCOUNTS_PER_DEVICE = 5`) |
| Rate limiting | Throttler: 60 req/60s por defecto |
| Headers | Helmet (CSP, HSTS, XSS protection) |
| CORS | `CORS_ORIGINS` configurable por entorno |
| Fingerprint web | `deviceFingerprint.ts` en web_customer para detección de fraude |

---

## 6. Flujo de Datos Financiero

```
Pedido entregado
      │
      ▼
Calcula comisión driver (deliveryFee × commissionRate%)
      │
      ├── StoreType OFFICIAL_10  → Tienda paga 10% de subtotal a Musuq
      ├── StoreType OFFICIAL_5   → Tienda paga 5% + tarifa de envío con feeMultiplier
      └── StoreType NON_OFFICIAL → Precio con markup 10%, Musuq retiene diferencia
      │
      ▼
FinancialLedger (registro contable)
      │
      ▼
Wallet / WalletMovement / DepositTicket (billeteras — reemplazó WeeklySettlement)
      │
      ├── financialStatus → PENDING_PAYMENT si hay saldo
      ├── financialStatus → BLOCKED si deuda supera umbral
      └── Telegram alert automática a admin
```

---

*Documento generado automáticamente a partir del código fuente de musuq_v2.*  
*Fecha de análisis: 2026-04-10*
