# Esquema de Datos y Lógica de Negocio — Musuq V2
> ⚠️ **ACTUALIZACIÓN 2026-06:** las liquidaciones fueron ELIMINADAS y reemplazadas por **billeteras** (`backend/src/wallets/`). Ver `FINANZAS_BILLETERAS.md`. Las menciones a finance/settlements/deposits en este documento son históricas.


> Base de datos: **PostgreSQL** gestionada vía **TypeORM** (synchronize: true).  
> No existen triggers nativos de PostgreSQL; toda la lógica de automatización está implementada en **NestJS Services** (capa de aplicación).  
> Timezone: `America/Lima` en todo el backend.

---

## 1. Diagrama de Relaciones (ERD en texto)

```
Country ──< Region ──< City
                          │
              Store >── City (region/country/city text fields)
                │
                ├──< StoreLocation (sucursales)
                ├──< Product ──< OrderItem >── Order
                ├── PricingMap ──< ZonePricingRule >── DeliveryZone
                └──< WeeklySettlement (store settlements)

User ──< DeviceToken
  │
  ├──1:1── Driver ──< Order (driverId)
  │              └──< WeeklySettlement (driver settlements)
  ├──< Order (customerId)
  ├──< UserAddress
  ├──< Deposit
  ├──< Coupon (owner, nullable)
  └──< Favorite

Order ──< OrderItem
      ──< OrderEvent (audit)
      ──< Payment
      ──< FinancialLedger
      ── StoreLocation (storeLocationId, nullable)

CommissionConfig (city o global NULL)
DeliveryFeeRule (city o global NULL)
Setting (singleton global)
BannedDevice / VerificationCode / WaOtpSession (auth)
```

---

## 2. Diccionario de Datos — Tablas Principales

### 2.1 `users`

| Columna | Tipo | Nullable | Default | Descripción |
|---|---|---|---|---|
| id | int (PK) | NO | autoincrement | Identificador único |
| phone | varchar(20) | NO | — | Teléfono E.164 (único) |
| name | varchar(100) | SÍ | NULL | Nombre completo |
| email | varchar(100) | SÍ | NULL | Email (único) |
| role | enum | NO | CLIENT | CLIENT, DRIVER, VENDOR, ADMIN, SUPPORT |
| image_url | text | SÍ | NULL | Foto de perfil (Cloudinary) |
| is_verified | boolean | NO | false | — |
| wallet_balance | decimal(10,2) | NO | 0 | Saldo prepagado |
| referral_code | varchar(20) | SÍ | NULL | Código de invitación propio (único) |
| referred_by_code | varchar(20) | SÍ | NULL | Código del usuario que invitó (inmutable) |
| referral_reward_claimed | boolean | NO | false | Recompensa ya acreditada |
| debt | decimal(10,2) | NO | 0 | Deuda por cancelaciones con penalidad |
| is_active | boolean | NO | true | — |
| document_id | varchar(20) | SÍ | NULL | DNI u otro doc |
| dni | varchar(8) | SÍ | NULL | DNI peruano (8 dígitos) |
| address | text | SÍ | NULL | Dirección textual |
| birthdate | date | SÍ | NULL | Fecha de nacimiento |
| password | varchar | SÍ | NULL | Hash de PIN (select: false) |
| device_id | text | SÍ | NULL | Fingerprint del dispositivo |
| country | varchar(50) | SÍ | NULL | País actual |
| region | varchar(100) | SÍ | NULL | Departamento/Región |
| city | varchar(100) | SÍ | NULL | Ciudad |
| latitude | decimal(10,7) | SÍ | NULL | Posición actual |
| longitude | decimal(10,7) | SÍ | NULL | Posición actual |
| registration_source | varchar(20) | SÍ | NULL | APP, WEB, ADMIN |
| created_at | timestamptz | NO | NOW() | — |

**Relaciones:** `OneToMany → device_tokens`

---

### 2.2 `drivers`

| Columna | Tipo | Nullable | Default | Descripción |
|---|---|---|---|---|
| id | int (PK) | NO | autoincrement | — |
| user_id | int (FK → users) | NO | — | OneToOne |
| is_online | boolean | NO | false | Disponible para recibir órdenes |
| current_lat | decimal(10,7) | SÍ | NULL | GPS en tiempo real |
| current_lng | decimal(10,7) | SÍ | NULL | GPS en tiempo real |
| vehicle_details | text | SÍ | NULL | Placa, marca, modelo |
| is_busy | boolean | NO | false | Tiene orden activa |
| rank | enum | NO | BRONZE | BRONZE, SILVER, GOLD, DIAMOND, LEGEND |
| max_concurrent_orders | int | NO | 1 | Sincronizado con rank |
| month_bad_orders | int | NO | 0 | Órdenes tardías/canceladas en el mes |
| offered_orders | int | NO | 0 | Total ofertas recibidas |
| accepted_orders | int | NO | 0 | Total aceptadas |
| cancelled_orders | int | NO | 0 | Total canceladas |
| performance_score | decimal(5,2) | NO | 100.00 | Score 0-100 |
| commission_rate | decimal(5,2) | NO | 5.00 | % sobre delivery fee |
| financial_status | varchar(20) | NO | CLEAR | CLEAR, PENDING_PAYMENT, BLOCKED |
| blocked_at | timestamptz | SÍ | NULL | Fecha de bloqueo |
| blocked_reason | text | SÍ | NULL | Motivo |
| alerted_financial_block | boolean | NO | false | Flag dedup para alerta Telegram |

**Rank → maxConcurrentOrders:** BRONZE=1, SILVER=2, GOLD=3, DIAMOND=4, LEGEND=5

---

### 2.3 `stores`

| Columna | Tipo | Nullable | Default | Descripción |
|---|---|---|---|---|
| id | int (PK) | NO | autoincrement | — |
| name | varchar | NO | — | Nombre del local |
| description | text | SÍ | NULL | — |
| address | varchar | NO | — | Dirección textual |
| region | varchar | SÍ | NULL | Lima Norte, etc. |
| country | varchar(50) | SÍ | NULL | PE, CO, MX |
| city | varchar(100) | SÍ | NULL | Lima, Arequipa, etc. |
| phone | varchar | SÍ | NULL | Teléfono de contacto |
| latitude | decimal(10,7) | SÍ | NULL | Coordenadas del local |
| longitude | decimal(10,7) | SÍ | NULL | — |
| image_url | varchar | SÍ | NULL | Logo (Cloudinary) |
| qr_code_url | varchar | SÍ | NULL | QR para pagos |
| bank_accounts | text | SÍ | NULL | Datos bancarios |
| vertical | varchar | NO | RESTAURANT | Vertical de negocio |
| delivery_fee | decimal(10,2) | NO | 5.00 | Tarifa base de envío |
| average_prep_time_minutes | smallint | NO | 15 | Tiempo promedio de preparación |
| rating | decimal(3,1) | SÍ | NULL | Rating promedio |
| review_count | int | NO | 0 | — |
| is_rating_hidden | boolean | NO | false | Oculta rating en UI |
| is_open | boolean | NO | true | Estado actual |
| is_hidden | boolean | NO | false | Oculto por admin |
| is_official | boolean | NO | false | — |
| store_type | varchar(20) | NO | OFFICIAL_10 | OFFICIAL_10, OFFICIAL_5, NON_OFFICIAL |
| commission_percentage | decimal(5,2) | NO | 10 | % comisión aplicado |
| schedule | json | SÍ | NULL | `{lun:{open:"08:00",close:"22:00",isOpen:true},...}` |
| owner_id | int (FK → users) | SÍ | NULL | Dueño del local |
| pricing_map_id | uuid | SÍ | NULL | Mapa de precios por zona |
| financial_status | varchar(20) | NO | CLEAR | CLEAR, PENDING_PAYMENT, BLOCKED |
| blocked_at | timestamptz | SÍ | NULL | — |
| blocked_reason | text | SÍ | NULL | — |
| slug | varchar(200) | SÍ | NULL | URL SEO: `/tiendas/pizza-express` |
| driver_pays_commission | boolean | NO | false | Driver asume la comisión |
| fee_multiplier | decimal(5,2) | NO | 1.0 | Multiplicador de tarifa base |
| telegram_order_alerts | boolean | NO | false | — |
| telegram_chat_ids | json | SÍ | NULL | Array hasta 5 chat IDs |
| wa_alert_numbers | json | SÍ | NULL | Array hasta 5 números WA |
| category_order | json | SÍ | NULL | Orden personalizado de categorías |
| alerted_financial_block | boolean | NO | false | Flag dedup Telegram |

---

### 2.4 `store_locations` (Sucursales)

| Columna | Tipo | Nullable | Default | Descripción |
|---|---|---|---|---|
| id | int (PK) | NO | autoincrement | — |
| store_id | int (FK → stores) | NO | — | CASCADE delete |
| name | varchar(120) | NO | — | "Sucursal Miraflores" |
| address | varchar(255) | SÍ | NULL | Dirección de la sucursal |
| latitude | decimal(10,7) | SÍ | NULL | — |
| longitude | decimal(10,7) | SÍ | NULL | — |
| is_active | boolean | NO | true | — |
| created_at | timestamptz | NO | NOW() | — |
| updated_at | timestamptz | NO | NOW() | — |

---

### 2.5 `orders`

| Columna | Tipo | Nullable | Default | Descripción |
|---|---|---|---|---|
| id | int (PK) | NO | autoincrement | — |
| customer_id | int | SÍ | NULL | FK a users |
| customer_name | varchar | SÍ | NULL | Snapshot del nombre |
| customer_phone | varchar | SÍ | NULL | Snapshot del teléfono |
| store_id | int (FK → stores) | SÍ | NULL | — |
| driver_id | int (FK → drivers) | SÍ | NULL | Asignado al aceptar |
| order_source | varchar(20) | NO | APP_NATIVE | APP_NATIVE, B2B_EXTERNAL |
| type | varchar | NO | FOOD | FOOD, COURIER |
| status | varchar | NO | PENDING | Ver estados abajo |
| store_location_id | int (FK) | SÍ | NULL | Sucursal asignada |
| pickup_address | text | SÍ | NULL | Para tipo COURIER |
| pickup_latitude | decimal(10,7) | SÍ | NULL | — |
| pickup_longitude | decimal(10,7) | SÍ | NULL | — |
| description | text | SÍ | NULL | Para tipo COURIER |
| delivery_address | text | NO | — | Dirección de entrega |
| delivery_latitude | decimal(10,7) | SÍ | NULL | — |
| delivery_longitude | decimal(10,7) | SÍ | NULL | — |
| delivery_instructions | text | SÍ | NULL | Instrucciones al driver |
| total_amount | decimal(10,2) | NO | — | subtotal + deliveryFee - discount + tip |
| delivery_fee | decimal(10,2) | NO | — | Tarifa calculada |
| subtotal | decimal(10,2) | NO | — | Suma de ítems |
| coupon_code | varchar(50) | SÍ | NULL | Código aplicado |
| discount_amount | decimal(10,2) | NO | 0 | Descuento calculado |
| tip_amount | decimal(10,2) | NO | 0 | Propina |
| is_paid | boolean | NO | false | — |
| payment_method | varchar | SÍ | NULL | CASH, CARD, WALLET |
| wallet_amount_used | decimal(10,2) | NO | 0 | Saldo wallet descontado |
| past_debt | decimal(10,2) | NO | 0 | Deuda previa cobrada en este pedido |
| country | varchar(50) | SÍ | NULL | Snapshot país |
| region | varchar(100) | SÍ | NULL | Snapshot región |
| cancellation_reason | text | SÍ | NULL | — |
| cancellation_type | varchar(50) | SÍ | NULL | Ver tipos abajo |
| cancellation_resolved_at | timestamptz | SÍ | NULL | — |
| assignment_mode_snapshot | varchar(30) | SÍ | NULL | AUTO_PROXIMITY o ADMIN_MANUAL |
| total_to_collect | decimal(10,2) | SÍ | NULL | B2B: monto a cobrar al cliente final |
| b2b_flat_fee | decimal(10,2) | SÍ | NULL | B2B: tarifa fija de flota |
| estimated_prep_time | int | SÍ | NULL | Minutos estimados de preparación |
| confirmed_at | timestamptz | SÍ | NULL | — |
| assigned_at | timestamptz | SÍ | NULL | — |
| accepted_at | timestamptz | SÍ | NULL | — |
| arrived_at | timestamptz | SÍ | NULL | Driver llegó al local |
| picked_up_at | timestamptz | SÍ | NULL | Recogido |
| at_customer_at | timestamptz | SÍ | NULL | Driver llegó al cliente |
| is_near_customer_notified | boolean | NO | false | Push "driver cerca" enviado |
| delivered_at | timestamptz | SÍ | NULL | Entregado |
| created_at | timestamptz | NO | NOW() | — |
| updated_at | timestamptz | NO | NOW() | — |
| alerted_slow_store | boolean | NO | false | Dedup alerta Telegram |
| alerted_slow_delivery | boolean | NO | false | Dedup alerta Telegram |
| alerted_cancellation | boolean | NO | false | Dedup alerta Telegram |

---

### 2.6 `order_items`

| Columna | Tipo | Nullable | Default | Descripción |
|---|---|---|---|---|
| id | int (PK) | NO | autoincrement | — |
| order_id | int (FK → orders) | NO | — | — |
| product_id | int (FK → products) | NO | — | — |
| quantity | int | NO | — | — |
| unit_price | decimal(10,2) | NO | — | Precio base al momento del pedido |
| subtotal | decimal(10,2) | NO | — | unit_price × quantity |
| product_name | varchar | NO | — | Snapshot inmutable del nombre |
| notes | text | SÍ | NULL | Instrucciones (sin sal, bien cocido…) |
| variant_id | varchar | SÍ | NULL | UUID de la variante elegida |
| variant_label | varchar | SÍ | NULL | Snapshot del nombre de variante |
| variant_price | decimal(10,2) | SÍ | NULL | Sobreescribe unit_price si existe |

---

### 2.7 `order_events` (Audit Log)

| Columna | Tipo | Nullable | Default | Descripción |
|---|---|---|---|---|
| id | int (PK) | NO | autoincrement | — |
| order_id | int (FK) | NO | — | @Index |
| previous_status | varchar(30) | SÍ | NULL | Estado anterior |
| new_status | varchar(30) | NO | — | Estado nuevo |
| changed_by_user_id | int | SÍ | NULL | — |
| role | varchar(20) | SÍ | NULL | VENDOR, DRIVER, ADMIN, SYSTEM |
| timestamp | timestamptz | NO | NOW() | @Index |
| metadata | jsonb | SÍ | NULL | Datos extra (locationName, prevLocationId…) |

---

### 2.8 `products`

| Columna | Tipo | Nullable | Default | Descripción |
|---|---|---|---|---|
| id | int (PK) | NO | autoincrement | — |
| name | varchar | NO | — | — |
| description | text | SÍ | NULL | — |
| price | decimal(10,2) | NO | — | Precio base |
| category_name | varchar | NO | — | "Pizzas", "Bebidas"… |
| image_url | varchar | SÍ | NULL | — |
| is_available | boolean | NO | true | Stock disponible |
| is_visible | boolean | NO | true | Visible en el menú |
| store_id | int (FK → stores) | SÍ | NULL | CASCADE delete |
| variants | json | SÍ | NULL | `[{id,label,price,isActive}]` |
| additionals | json | SÍ | NULL | `[{id,name,price,isActive}]` |

---

### 2.9 `payments`

| Columna | Tipo | Nullable | Default | Descripción |
|---|---|---|---|---|
| id | int (PK) | NO | autoincrement | — |
| amount | decimal(10,2) | NO | — | — |
| method | varchar | NO | — | CASH, CARD, WALLET |
| status | varchar | NO | — | PENDING, COMPLETED, FAILED |
| transaction_id | varchar | SÍ | NULL | Referencia externa |
| order_id | int (FK → orders) | NO | — | — |
| user_id | int (FK → users) | NO | — | — |
| created_at | timestamptz | NO | NOW() | — |

---

### 2.10 `coupons`

| Columna | Tipo | Nullable | Default | Descripción |
|---|---|---|---|---|
| id | int (PK) | NO | autoincrement | — |
| code | varchar | NO | — | Único, mayúsculas |
| type | enum | NO | — | FREE_DELIVERY, FIXED_DISCOUNT, PERCENTAGE_DISCOUNT, FIRST_ORDER |
| value | decimal(10,2) | NO | 0 | % o monto fijo según type |
| min_order_amount | decimal(10,2) | NO | 0 | Subtotal mínimo para aplicar |
| start_date | date | SÍ | NULL | Fecha inicio de vigencia |
| end_date | date | SÍ | NULL | Fecha fin de vigencia |
| start_time | varchar | SÍ | NULL | HH:mm — ventana horaria |
| end_time | varchar | SÍ | NULL | HH:mm — soporta cruce de medianoche |
| max_uses | int | NO | 0 | 0 = sin límite global |
| max_uses_per_user | int | NO | 0 | 0 = sin límite por usuario |
| used_count | int | NO | 0 | Contador global de usos |
| is_active | boolean | NO | true | — |
| is_referral | boolean | NO | false | Cupón de programa de referidos |
| country | varchar(100) | SÍ | NULL | Restricción geográfica |
| region | varchar(100) | SÍ | NULL | — |
| city | varchar(100) | SÍ | NULL | — |
| owner_id | int (FK → users) | SÍ | NULL | onDelete: SET NULL |
| created_at | timestamptz | NO | NOW() | — |
| updated_at | timestamptz | NO | NOW() | — |

---

### 2.11 `financial_ledger`

| Columna | Tipo | Nullable | Default | Descripción |
|---|---|---|---|---|
| id | int (PK) | NO | autoincrement | — |
| user_id | int | NO | — | Store owner, Driver, o 0 = Platform |
| role | varchar(20) | NO | — | STORE, DRIVER, PLATFORM |
| order_id | int (FK) | SÍ | NULL | onDelete: SET NULL |
| amount | decimal(12,2) | NO | — | Positivo = ingreso, Negativo = cargo |
| type | varchar(30) | NO | — | Ver tipos abajo |
| week_number | smallint | NO | — | Semana ISO (1-53) |
| year | smallint | NO | — | — |
| metadata | jsonb | SÍ | NULL | Porcentajes aplicados, storeType, etc. |
| created_at | timestamptz | NO | NOW() | — |

**Índices:** `(user_id, year, week_number)`, `(order_id)`, `(role, year, week_number)`

**Tipos de entrada (`type`):**

| Tipo | Descripción |
|---|---|
| PLATFORM_COMMISSION | Comisión Musuq sobre subtotal de tienda |
| DRIVER_COMMISSION | Comisión driver sobre delivery fee |
| STORE_REVENUE | Ingresos tienda (solo pagos CARD) |
| DELIVERY_FEE | Ingresos driver por entrega |
| CARD_PROCESSING_FEE | Fee procesador de tarjeta |
| ADJUSTMENT | Corrección manual |
| B2B_FLEET_FEE | Tarifa de flota B2B (aislado) |

---

### 2.12 `weekly_settlements_v2`

| Columna | Tipo | Nullable | Default | Descripción |
|---|---|---|---|---|
| id | int (PK) | NO | autoincrement | — |
| user_id | int | NO | — | Store owner o Driver |
| role | varchar(20) | NO | — | STORE, DRIVER |
| week_number | smallint | NO | — | Semana ISO |
| year | smallint | NO | — | — |
| total_gross | decimal(12,2) | NO | — | Suma entradas positivas |
| total_commissions | decimal(12,2) | NO | — | Suma comisiones |
| total_card_fees | decimal(12,2) | NO | — | — |
| total_adjustments | decimal(12,2) | NO | — | — |
| net_amount | decimal(12,2) | NO | — | gross - commissions - fees + adjustments |
| status | varchar(20) | NO | — | PENDING, APPROVED, PAID, OBSERVED |
| approved_by_admin_id | int | SÍ | NULL | — |
| approved_at | timestamptz | SÍ | NULL | — |
| paid_at | timestamptz | SÍ | NULL | — |
| is_negative | boolean | NO | false | Saldo negativo: tienda/driver debe a Musuq |
| week_label | varchar(10) | SÍ | NULL | "2026-S01" |
| observed_reason | text | SÍ | NULL | — |
| created_at | timestamptz | NO | NOW() | — |
| updated_at | timestamptz | NO | NOW() | — |

**Índice único:** `(user_id, year, week_number)`

---

### 2.13 `delivery_fee_rules`

| Columna | Tipo | Nullable | Default | Descripción |
|---|---|---|---|---|
| id | int (PK) | NO | autoincrement | — |
| name | varchar(120) | NO | — | "Recargo nocturno Lima" |
| city | varchar(100) | SÍ | NULL | NULL = todas las ciudades |
| type | varchar(30) | NO | — | SURCHARGE_FIXED, SURCHARGE_PERCENTAGE, MINIMUM_FEE, MULTIPLIER, BASE_FEE |
| value | decimal(8,2) | NO | — | Valor según type |
| time_from | varchar(5) | SÍ | NULL | HH:mm inicio ventana |
| time_to | varchar(5) | SÍ | NULL | HH:mm fin (soporta cruce 00:00) |
| timezone | varchar(60) | NO | America/Lima | — |
| days_of_week | json | SÍ | NULL | `[0,1,2,3,4,5,6]` Dom=0 |
| cart_notice | varchar(500) | SÍ | NULL | Aviso mostrado al cliente en el carrito |
| priority | int | NO | 10 | Menor número = se aplica primero |
| is_active | boolean | NO | true | — |
| created_at | timestamptz | NO | NOW() | — |
| updated_at | timestamptz | NO | NOW() | — |

---

### 2.14 `delivery_zones`

| Columna | Tipo | Nullable | Default | Descripción |
|---|---|---|---|---|
| id | int (PK) | NO | autoincrement | — |
| name | varchar(150) | NO | — | "Centro Histórico" |
| slug | varchar(100) | NO | — | Único: "centro" |
| city | varchar(100) | NO | — | — |
| geometry | jsonb | NO | — | GeoJSON Polygon/MultiPolygon |
| tarifa_base | decimal(10,2) | NO | 5.00 | Tarifa base S/ |
| tarifa_km_extra | decimal(10,2) | NO | 1.50 | S/ extra por km excedente |
| tarifa_minima | decimal(10,2) | NO | 5.00 | Mínimo cobrable |
| tarifa_maxima | decimal(10,2) | NO | 25.00 | Máximo cobrable |
| is_active | boolean | NO | true | — |
| created_at | timestamptz | NO | NOW() | — |
| updated_at | timestamptz | NO | NOW() | — |

---

### 2.15 `commission_configs`

| Columna | Tipo | Nullable | Default | Descripción |
|---|---|---|---|---|
| id | int (PK) | NO | autoincrement | — |
| city | varchar(100) | SÍ | NULL | NULL = config global por defecto |
| store_commission_pct | decimal(5,2) | NO | 10 | % cobrado a tiendas |
| driver_commission_pct | decimal(5,2) | NO | 5 | % del delivery fee al driver |
| created_at | timestamptz | NO | NOW() | — |
| updated_at | timestamptz | NO | NOW() | — |

---

### 2.16 `settings` (Singleton)

| Columna | Tipo | Default | Descripción |
|---|---|---|---|
| assignment_mode | enum | AUTO_PROXIMITY | AUTO_PROXIMITY (Voladera) o ADMIN_MANUAL |
| rain_mode | boolean | false | Activa recargo por lluvia |
| demand_factor | decimal | 5.00 | Factor de demanda para pricing dinámico |
| fee_mode | varchar | zone | `zone` (polígonos TurfJS) o `distance` (lineal) |
| distance_price_per_km | decimal | — | S/ por km en modo distance |
| distance_min_fee | decimal | — | Mínimo en modo distance |
| distance_max_fee | decimal | — | Máximo en modo distance |
| driver_commission_percentage | decimal | 5 | Default global comisión driver |
| weekly_cut_day | int | 1 | Día de corte semanal (1=Lun, 7=Dom) |
| settlement_grace_hours | int | 24 | Horas de gracia post-corte |
| block_if_negative | boolean | — | Bloquear tienda/driver si settlement negativo |
| referral_enabled | boolean | — | — |
| referral_reward_amount | decimal | — | S/ acreditado al referidor |
| referral_discount_amount | decimal | — | S/ acreditado al nuevo usuario |
| support_phone | varchar | — | Número de soporte para WA |
| high_demand_threshold | int | — | Umbral para alertas de alta demanda |
| delayed_order_mins | int | — | Minutos para alerta de pedido lento |

---

## 3. Estados del Pedido

### 3.1 Flujo Principal

```
                              ┌─────────────────────────────────────────────┐
                              │              CICLO DE VIDA                  │
                              └─────────────────────────────────────────────┘

  PENDING ──────────────────> CONFIRMED ──────────────> WAITING_DRIVER
  (cliente crea)              (local confirma +          (buscando driver;
                               storeLocationId +          auto-asignación
                               estimatedPrepTime)         o manual)
                                    │
                                    │ [local rechaza]
                                    ▼
                               CANCELLED ◄─────────────── (cliente, admin, support)

  WAITING_DRIVER ────────────> DRIVER_ASSIGNED ─────────> ACCEPTED
  (broadcast FCM)              (modo AUTO)                 (driver acepta)

  ACCEPTED ──────────────────> AT_STORE ─────────────────> PICKED_UP
  (driver en camino)           (driver llegó al local)      (recogió el pedido)

  PICKED_UP ─────────────────> AT_CUSTOMER ───────────────> DELIVERED ✓
  (en camino al cliente)       (llegó al destino)            (entregado)
```

### 3.2 Tabla de Estados

| Estado | Quién lo activa | Endpoint | Timestamp grabado |
|---|---|---|---|
| PENDING | Cliente / Admin | `POST /orders` | `created_at` |
| CONFIRMED | Vendor / Admin | `PATCH /orders/:id/status` | `confirmed_at` |
| WAITING_DRIVER | Sistema (auto) / Admin | interno | — |
| DRIVER_ASSIGNED | Sistema (auto) | `AssignmentService` | `assigned_at` |
| ACCEPTED | Driver | `POST /orders/:id/accept` | `accepted_at` |
| AT_STORE | Driver | `POST /orders/:id/arrive-store` | `arrived_at` |
| PICKED_UP | Driver | `POST /orders/:id/pickup` | `picked_up_at` |
| AT_CUSTOMER | Driver | `POST /orders/:id/at-customer` | `at_customer_at` |
| DELIVERED | Driver | `POST /orders/:id/complete` | `delivered_at` |
| CANCELLED | Cliente/Admin/Support | varios endpoints | — |

### 3.3 Estados Legacy (web_store)

Aún usados en algunos flujos del panel de tienda:  
`PREPARING → READY → ON_THE_WAY → COMPLETED`

### 3.4 Tipos de Cancelación (`cancellation_type`)

| Tipo | Impacto financiero |
|---|---|
| CLIENT_FAULT | Penalidad S/ 5 a `user.debt`; comisiones se registran normalmente |
| STORE_FAULT_BEFORE_DRIVER_PAID | Anulación total; sin entradas en ledger |
| STORE_FAULT_AFTER_DRIVER_PAID | Driver conserva delivery fee; tienda sin comisión |
| DRIVER_FAULT | Se registran comisiones normales; driver pierde entrega |

---

## 4. Lógica de Cálculo de Tarifa de Envío

> Implementada en: `orders.service.ts → calculateFee()` y `couriers.service.ts`  
> **No hay triggers SQL.** Es lógica de aplicación en NestJS.

### 4.1 Modo ZONE (default, `feeMode = 'zone'`)

```
1. distance_km = haversine(store.lat/lng → delivery.lat/lng)

2. Para cada delivery_zone activa con geometry GeoJSON:
     Si booleanPointInPolygon(deliveryPoint, zone.geometry):
       tarifa_base = zone.tarifa_base
       km_extra    = zone.tarifa_km_extra
       fee = tarifa_base + (distance_km * km_extra)
       fee = clamp(fee, zone.tarifa_minima, zone.tarifa_maxima)
       BREAK

3. Si ninguna zona coincide (fallback):
     fee = distance_km * 2.0   (S/ 2/km mínimo)

4. Aplicar delivery_fee_rules (ordenadas por priority ASC):
     SURCHARGE_FIXED      → fee += rule.value
     SURCHARGE_PERCENTAGE → fee += fee * (rule.value / 100)
     MINIMUM_FEE          → fee = max(fee, rule.value)
     MULTIPLIER           → fee *= rule.value
     BASE_FEE             → fee = (fee - tarifa_base) + rule.value

5. Aplicar multiplicador de tienda:
     fee *= store.fee_multiplier

6. Redondeo al medio sol más cercano hacia arriba:
     fee = ceil(fee × 2) / 2   →   S/ 7.30 → S/ 7.50
```

### 4.2 Modo DISTANCE (`feeMode = 'distance'`)

```
fee = distance_km * settings.distance_price_per_km
fee = clamp(fee, settings.distance_min_fee, settings.distance_max_fee)
(+ mismas reglas de delivery_fee_rules)
```

### 4.3 Courier (tipo COURIER, `couriers.service.ts`)

```
1. Determinar zona pickup y zona delivery
2. Si ambas en misma zona: fee = zona.tarifaBase
3. Si zonas distintas: fee = max(zonaPickup.tarifaBase, zonaDelivery.tarifaBase) + distancia × kmExtra
4. Aplicar mínimos y máximos de zona
5. Fallback si no hay zonas: S/ 6.00
```

---

## 5. Lógica de Cupones

> Implementada en: `coupons.service.ts → applyCoupon()`

### 5.1 Pipeline de Validación

```
1. Existe el cupón y is_active = true?
2. used_count < max_uses (o max_uses = 0)?
3. Uso por usuario < max_uses_per_user (órdenes no canceladas con ese código)?
4. Si type = FIRST_ORDER: usuario tiene 0 órdenes entregadas?
5. subtotal >= min_order_amount?
6. now() entre start_date y end_date?
7. Hora Lima entre start_time y end_time (soporta cruce medianoche)?
8. Si type = FREE_DELIVERY: order.type ≠ COURIER?
9. Restricción geográfica (country/region/city si está definida)?
```

### 5.2 Cálculo del Descuento

| Type | Fórmula |
|---|---|
| FREE_DELIVERY | `discount = delivery_fee` (100% del envío) |
| PERCENTAGE_DISCOUNT | `discount = subtotal × (value / 100)` |
| FIXED_DISCOUNT | `discount = value` |
| FIRST_ORDER | `discount = value` (monto fijo) |

### 5.3 Recálculo Dinámico

Cuando el cliente cambia de dirección en el carrito y ya tiene un cupón aplicado, la app llama nuevamente a `applyCoupon()` con la nueva `deliveryFee`. Esto garantiza que `FREE_DELIVERY` descuente el monto correcto y no el de la dirección anterior.

---

## 6. Lógica de Comisiones

> Implementada en: `finance.service.ts → recordOrderDelivered()`

### 6.1 Jerarquía de Resolución de Tasas

```
1. Driver.commissionRate    (override individual por driver)
2. CommissionConfig.city    (override por ciudad)
3. CommissionConfig NULL     (config global — default 10%/5%)
4. Settings.driverCommissionPercentage (fallback absoluto — 5%)
```

### 6.2 Cálculo por StoreType

```
OFFICIAL_10:
  platform_comm = subtotal × 10%
  store_revenue = subtotal (solo en pagos CARD)

OFFICIAL_5:
  base = subtotal / 1.05          ← elimina el markup del 5% ya cobrado al cliente
  platform_comm = base × 10%
  store_revenue = base

NON_OFFICIAL:
  base = precio_sin_markup = subtotal / 1.10
  platform_comm = subtotal - base  ← retiene el margen del 10%
  store_revenue = base
```

### 6.3 Comisión del Driver

```
driver_comm_amount = delivery_fee × (driver.commissionRate / 100)
```

### 6.4 Modos especiales

**`driverPaysCommission = true`:**
```
→ Entradas de tienda: OMITIDAS (driver asume toda la comisión)
→ Driver paga: platform_comm + driver_comm_amount
```

**`B2B_EXTERNAL`:**
```
→ Cero comisiones de plataforma/driver nativas
→ Solo registra: B2B_FLEET_FEE (tienda paga) + DELIVERY_FEE (driver gana)
```

**Pago en EFECTIVO (CASH):**
```
→ Driver/tienda ya cobraron → ledger graba DÉBITOS (montos negativos = deben a Musuq)
```

**Pago con WALLET/CARD:**
```
→ Musuq recibió el dinero → ledger graba CRÉDITOS (Musuq debe pagar)
```

---

## 7. Sistema de Wallet y Deudas

### 7.1 Wallet

```
Al crear orden:
  walletAmountUsed = min(user.walletBalance, totalAmount)
  user.walletBalance -= walletAmountUsed
  Si walletAmountUsed >= totalAmount → is_paid = true
  Resto: paymentMethod = CASH o CARD

Al cancelar (CLIENT_FAULT con wallet ya descontado):
  user.walletBalance += walletAmountUsed  ← devolución automática
```

### 7.2 Deuda por Cancelación

```
Estado del pedido al cancelar: PREPARING / READY / DRIVER_ASSIGNED
→ user.debt += 5.00  (penalidad fija)
→ Próximo pedido del usuario BLOQUEADO hasta que admin salda la deuda
→ Si usuario tiene deuda: pastDebt en nueva orden; se cobra junto con el pedido
```

---

## 8. Sistema de Referidos

```
Al entregar pedido (status → DELIVERED/COMPLETED):

  Si order.customer tiene referredByCode:
    referrer = users WHERE referralCode = referredByCode
    Si referrer existe Y referralRewardClaimed = false:
      referrer.walletBalance += settings.referralRewardAmount
      customer.walletBalance += settings.referralDiscountAmount
      customer.referralRewardClaimed = true   ← solo una vez
      [Registro en financial_ledger tipo ADJUSTMENT]

  Si coupon.isReferral = true (cupón de referido):
    Acreditar coupon.value al dueño del cupón (owner)
```

---

## 9. Archivos SQL del Proyecto

> **No hay migrations automáticas de TypeORM.** El esquema se sincroniza vía `synchronize: true`.  
> Existen scripts manuales en `/backend/scripts/`:

| Archivo | Propósito |
|---|---|
| `migration_pricing_maps.sql` | Crea tablas `pricing_maps` y `zone_pricing_rules` |
| `reset_database.sql` | Trunca todos los datos EXCEPTO tiendas y usuario master (+51902322766) |

---

## 10. Automatizaciones y Cron Jobs

> Todos son **cron jobs de NestJS** (`@Cron`), NO triggers de PostgreSQL.

| Cron | Frecuencia | Descripción |
|---|---|---|
| Slow Store Detector | Cada 1 min | Alerta Telegram si pedido CONFIRMED sin avanzar más de X minutos |
| Slow Delivery Detector | Cada 1 min | Alerta si PICKED_UP sin entregarse en tiempo esperado |
| Cancellation Monitor | Cada 1 min | Alerta si pedido CANCELLED sin resolución (dedup con `alerted_cancellation`) |
| Orphaned Orders | Cada 1 min | Alerta si WAITING_DRIVER sin driver asignado en X minutos |
| Financial Block Monitor | Cada 30 min | Verifica `financialStatus` de tiendas/drivers; bloquea y alerta si deuda supera umbral |

**Flags de dedup** (evitan alertas duplicadas):
- `order.alerted_slow_store`
- `order.alerted_slow_delivery`
- `order.alerted_cancellation`
- `driver.alerted_financial_block`
- `store.alerted_financial_block`

---

*Documento generado a partir del código fuente de musuq_v2.*  
*Fecha de análisis: 2026-04-10*
