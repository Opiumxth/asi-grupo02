# 08 — Mapa Conceptual Completo del Rango de Examen

> El flujo entero, de "Final de Sistema" a "Modelo de Análisis completo", visto desde distintos ángulos.

---

## 1. Vista de artefactos (qué se produce en cada paso)

```mermaid
flowchart TD
    subgraph SISTEMA["📦 CIERRE DE 'SISTEMA' (Tema 1)"]
        direction TB
        A1["Actores del Sistema"]
        A2["Especificación de CUS<br/>(alto nivel + expandido)"]
        A3["Relaciones entre CUS<br/>(include / extend / generalización)"]
        A4["Diagrama de Actividades del CUS"]
        A5["Prototipo de interfaz"]
    end

    subgraph ANALISIS["🔬 MODELO DE ANÁLISIS COMPLETO (Temas 2-5)"]
        direction TB
        B1["Modelo Conceptual / Dominio<br/>(Tema 2)"]
        B2["Diagrama de Secuencia<br/>del Sistema — DSS<br/>(Tema 3)"]
        B3["Contratos de<br/>Operaciones<br/>(Tema 4)"]
        B4["Realización del CU en Análisis<br/>(Tema 5)"]
        B1 --> B4
        B2 --> B4
        B3 --> B4
    end

    A2 -->|"sustantivos del flujo"| B1
    A2 -->|"acciones actor→sistema"| B2
    B1 -.->|"vocabulario"| B3
    B2 -.->|"firmas de operación"| B3

    SISTEMA --> ANALISIS
    ANALISIS --> DISEÑO["🏗️ Modelo de Diseño<br/>(fuera del examen)"]

    style SISTEMA fill:#3498db22
    style ANALISIS fill:#2ecc7122
    style DISEÑO fill:#7f8c8d33
```

---

## 2. Vista de preguntas (qué pregunta responde cada tema)

```mermaid
flowchart LR
    Q1["¿Qué funcionalidades<br/>ofrece el sistema<br/>y a quién?"] --> T1["Tema 1<br/>CUS"]
    Q2["¿Qué conceptos<br/>del dominio existen<br/>y cómo se relacionan?"] --> T2["Tema 2<br/>Modelo Conceptual"]
    Q3["¿Qué eventos<br/>llegan al sistema<br/>y en qué orden?"] --> T3["Tema 3<br/>DSS"]
    Q4["¿Qué cambia de estado<br/>con cada evento?"] --> T4["Tema 4<br/>Contratos"]
    Q5["¿Cómo se ve todo<br/>junto, por caso de uso?"] --> T5["Tema 5<br/>Realización de Análisis"]

    T1 --> T2 --> T3 --> T4 --> T5

    style T1 fill:#3498db,color:#fff
    style T2 fill:#2ecc71,color:#fff
    style T3 fill:#9b59b6,color:#fff
    style T4 fill:#f39c12,color:#fff
    style T5 fill:#e74c3c,color:#fff
```

---

## 3. Vista de trazabilidad (cómo un solo dato viaja por todos los temas)

Sigue el dato **"placa del vehículo"** a través de todo el rango, en el caso SCEV:

```mermaid
flowchart TD
    ORIGEN["Requisito R13:<br/>'registrar... placa del vehículo'"] --> T1["Tema 1<br/>Aparece en el flujo básico<br/>de 'Gestionar Parqueo':<br/>'el Trabajador ingresa... la placa'"]
    T1 --> T2["Tema 2<br/>Se convierte en atributo<br/>Vehiculo.placa (tipo texto)"]
    T1 --> T3["Tema 3<br/>Se convierte en parámetro:<br/>registrarIngresoVehiculo(..., placa, ...)"]
    T2 --> T4["Tema 4<br/>Aparece en postcondición:<br/>'se asignó v.placa = placa'"]
    T3 --> T4
    T4 --> T5["Tema 5<br/>Aparece en el Diagrama<br/>de Comunicación de Análisis:<br/>v1:Vehiculo con placa='ABC-123'"]

    style ORIGEN fill:#7f8c8d,color:#fff
    style T1 fill:#3498db,color:#fff
    style T2 fill:#2ecc71,color:#fff
    style T3 fill:#9b59b6,color:#fff
    style T4 fill:#f39c12,color:#fff
    style T5 fill:#e74c3c,color:#fff
```

> 🔑 Esta es la mejor forma de estudiar para el examen: elige UN dato cualquiera de tu propio caso de estudio y sigue su rastro por los 5 temas. Si puedes hacerlo sin trabarte, entendiste la trazabilidad completa del rango.

---

## 4. Vista de "quién produce, quién consume" (tabla resumen)

| Tema | Insumo (qué necesita) | Producto (qué entrega) | Consumidor directo |
|---|---|---|---|
| 1. CUS | Requisitos + Actores del Sistema | Especificación CUS + Diagrama de CUS + Actividades + Prototipo | Temas 2 y 3 |
| 2. Modelo Conceptual | Flujo del CUS (sustantivos) | Clases, atributos, asociaciones | Temas 4 y 5 |
| 3. DSS | Flujo del CUS (verbos actor→sistema) | Lista de operaciones del sistema | Tema 4 |
| 4. Contratos | Operaciones (Tema 3) + vocabulario (Tema 2) | Precondiciones/postcondiciones por operación | Tema 5 |
| 5. Realización de Análisis | Temas 2, 3 y 4 integrados | Modelo de Análisis completo verificado | Diseño (fuera del examen) |

---

## 5. Vista de "todo el sistema SCEV" — mapa conceptual final

Panorama completo de SCEV, uniendo el Modelo Conceptual completo (Tema 2) con las operaciones (Tema 3) que actúan sobre él:

```mermaid
flowchart TB
    subgraph ACTORES["Actores del Sistema"]
        ADMIN["Administrador"]
        TRAB["Trabajador"]
        SENS["Sensor"]
    end

    subgraph CUS_SET["Casos de Uso del Sistema"]
        CUS1["Gestionar Trabajadores"]
        CUS2["Gestionar Catálogo de Tarifas"]
        CUS3["Gestionar Estacionamientos"]
        CUS4["Registrar Clientes"]
        CUS5["Gestionar Parqueo ⭐"]
        CUS6["Registrar Pagos"]
        CUS7["Operaciones del Sensor"]
    end

    subgraph MODELO["Modelo Conceptual"]
        M1["Estacionamiento"]
        M2["Sector / Espacio"]
        M3["Vehiculo"]
        M4["Cliente"]
        M5["Parqueo"]
        M6["Ticket"]
        M7["CatalogoDeTarifas / TipoTarifa"]
        M8["Pago (+ subclases)"]
    end

    ADMIN --> CUS1 & CUS2 & CUS3
    TRAB --> CUS4 & CUS5 & CUS6
    SENS --> CUS7

    CUS5 -.->|"produce operaciones sobre"| M5
    CUS5 -.-> M3
    CUS5 -.-> M2
    CUS5 -.-> M6
    CUS6 -.-> M8
    CUS4 -.-> M4
    CUS2 -.-> M7
    CUS3 -.-> M1

    style ACTORES fill:#3498db22
    style CUS_SET fill:#9b59b622
    style MODELO fill:#2ecc7122
    style CUS5 fill:#e74c3c,color:#fff
```

---

## 6. Los 3 niveles de abstracción que NUNCA hay que mezclar

```mermaid
flowchart TD
    N1["Nivel 1: NEGOCIO<br/>(fuera de este examen)<br/>Actor de Negocio, CUN, Worker"] --> N2
    N2["Nivel 2: SISTEMA / REQUISITOS<br/>(Tema 1 — 'Final de Sistema')<br/>Actor del Sistema, CUS"] --> N3
    N3["Nivel 3: ANÁLISIS<br/>(Temas 2-5 — este examen)<br/>Modelo Conceptual, DSS, Contratos,<br/>Realización — objetos SIN métodos"] --> N4
    N4["Nivel 4: DISEÑO<br/>(fuera de este examen)<br/>Clases de software CON métodos,<br/>Colaboración de diseño, MVC"]

    style N1 fill:#7f8c8d,color:#fff
    style N2 fill:#3498db,color:#fff
    style N3 fill:#2ecc71,color:#fff
    style N4 fill:#7f8c8d,color:#fff
```

> 🔑 El examen vive completo dentro del **Nivel 2 (el final) + Nivel 3 (completo)**. Cualquier pregunta que te obligue a usar vocabulario del Nivel 1 (Workers, CUN) o del Nivel 4 (métodos, BD, MVC) probablemente es, o una pregunta de contexto/comparación, o una señal de que te saliste del alcance.
