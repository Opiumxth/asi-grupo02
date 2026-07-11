# 07 — Conceptos que Más se Confunden

> Cada par se presenta lado a lado, con la pregunta exacta que los distingue y un ejemplo de SCEV. Este archivo es el "examen contra ti mismo" antes del examen real.

---

## 1. Actor de Negocio vs. Actor del Sistema

| | Actor de Negocio | Actor del Sistema |
|---|---|---|
| Interactúa con | La empresa/negocio completo | El software específicamente |
| ¿Puede ser interno? | No (siempre externo al negocio) | Sí, puede ser un Worker de Negocio que usa el software |
| Ejemplo SCEV | El "Cliente" que llega con su carro (nunca toca el software) | `Trabajador`, que usa la aplicación para registrar el ingreso |

**Pregunta que los distingue**: ¿esta entidad interactúa directamente con el software, o solo con la empresa/proceso en general?

---

## 2. `«include»` vs. `«extend»`

| | Include | Extend |
|---|---|---|
| ¿Se ejecuta siempre? | Sí | No, depende de una condición |
| Dirección de la flecha | Base → Incluido | **Extendido → Base** |
| ¿El comportamiento tiene sentido solo? | No (ej. "Validar Cliente Registrado" nunca se pide solo) | A veces sí puede tener sentido parcial, pero solo se activa dentro de otro CU |

**Pregunta que los distingue**: ¿este comportamiento SIEMPRE ocurre (include) o SOLO A VECES bajo una condición (extend)?

**Truco de memoria para la flecha**: "lo que se agrega, apunta hacia lo que recibe". El CU extendido (lo opcional) apunta hacia el CU base.

---

## 3. Atributo vs. Clase Conceptual

| | Atributo | Clase conceptual |
|---|---|---|
| ¿Tiene identidad propia? | No | Sí |
| ¿Es tipo de dato simple? | Sí (texto, número, fecha, booleano) | No |
| Ejemplo SCEV | `placa: String` (atributo de `Vehículo`) | `Sector` (tiene nombre, piso, capacidad — no es solo un texto) |

**Pregunta que los distingue**: ¿en el mundo real, esto ocupa espacio, tiene su propio ciclo de vida, o es una entidad reconocible por sí misma? Si sí → clase. Ante la duda → clase.

---

## 4. Agregación vs. Composición

| | Agregación (◇) | Composición (◆) |
|---|---|---|
| ¿La parte sobrevive sin el todo? | Sí | No |
| Multiplicidad típica en el extremo del todo | Puede ser > 1 | Siempre `1` |
| Ejemplo SCEV | `CatálogoDeTarifas ◇— TipoTarifa` (discutible, ver nota abajo) | `Estacionamiento ◆— Sector` (si se elimina el estacionamiento, sus sectores no tienen sentido) |

**Pregunta que los distingue**: si destruyo el "todo" en este instante, ¿la "parte" sigue teniendo sentido de existir por sí sola?

> ⚠️ **Nota honesta**: la frontera entre agregación y composición es la más debatida de todo el curso — incluso profesionales experimentados discrepan en casos límite. En examen, lo que importa no es "adivinar" la respuesta exacta que el profesor tiene en mente, sino **justificar tu elección con la pregunta de supervivencia de la parte**. Si tu justificación es coherente, es una respuesta defendible.

---

## 5. Diagrama de Clases (Conceptual) vs. Diagrama de Clases de Diseño

| | Conceptual (Tema 2, SÍ entra) | De Diseño (fuera del examen) |
|---|---|---|
| ¿Tiene métodos? | No | Sí |
| ¿Tiene tipos de datos concretos? | No (o muy informal) | Sí (`String`, `int`, `Date`...) |
| ¿Tiene visibilidad (+/-/#)? | No | Sí |
| Representa | Conceptos del mundo real | Clases de software |

**Pregunta que los distingue**: ¿este diagrama tiene métodos y tipos de datos de programación? Si sí, ya es Diseño, no Análisis.

---

## 6. Diagrama de Secuencia del Sistema (DSS) vs. Diagrama de Colaboración/Secuencia de Diseño

| | DSS (Tema 3, SÍ entra) | De Diseño (fuera del examen) |
|---|---|---|
| Participantes | Actor + "Sistema" (una sola caja) | Actor + múltiples objetos de software internos |
| Perspectiva | Caja negra (externa) | Caja blanca (interna) |
| Mensajes | Operaciones del sistema | Llamadas a métodos entre objetos concretos |

**Pregunta que los distingue**: ¿el diagrama muestra el sistema como una sola caja, o abre esa caja y muestra los objetos de software que colaboran por dentro?

---

## 7. Precondición vs. Postcondición

| | Precondición | Postcondición |
|---|---|---|
| ¿Cuándo se cumple? | Antes de ejecutar la operación | Después de ejecutarla con éxito |
| ¿Quién debe garantizarla? | El actor / el estado previo | El sistema, como resultado |
| Tiempo verbal típico | Presente ("existe...", "el cliente está...") | Pasado ("se creó...", "se asoció...") |

**Pregunta que los distingue**: ¿esto describe una condición que debe ser cierta ANTES de empezar, o el resultado garantizado DESPUÉS de terminar?

---

## 8. Postcondición correcta (estado) vs. Postcondición incorrecta (algoritmo)

| | Correcta (estado) | Incorrecta (algoritmo) |
|---|---|---|
| Ejemplo | "Se creó una instancia de Parqueo (p)" | "El sistema crea un objeto Parqueo llamando al constructor" |
| Ejemplo | "Se asignó p.numeroCorrelativo = valor generado" | "El sistema calcula el siguiente número correlativo sumando 1 al anterior" |
| ¿Explica el CÓMO? | No | Sí (por eso está mal) |

**Pregunta que los distingue**: ¿esta frase describe un HECHO ya consumado (estado), o describe los PASOS para llegar ahí (procedimiento)?

---

## 9. Realización de CU en Análisis vs. Realización de CU en Diseño

| | En Análisis (Tema 5, SÍ entra) | En Diseño (fuera del examen) |
|---|---|---|
| Objetos que colaboran | Conceptuales (del Modelo Conceptual, Tema 2) | De software (con métodos, del Modelo de Diseño) |
| ¿Tienen responsabilidades de software? | No | Sí |
| Diagrama asociado | Diagrama de Comunicación de Análisis (informal) | Diagrama de Colaboración de Diseño (formal) |

**Pregunta que los distingue**: ¿los objetos del diagrama tienen métodos y responsabilidades de software, o son puramente conceptos del dominio relacionándose?

> Esta es, según el propio material del curso (`Lab_14 sol.pdf`, que agrupa "diagrama de secuencia/colaboración" bajo "modelo de análisis (realizaciones)"), la confusión más "oficial" del curso — incluso el material fuente no siempre es 100% explícito en separar ambos niveles. Ver Tema 5, sección "Conceptos fundamentales", punto 1.

---

## 10. Modelo Conceptual vs. Modelo de Análisis (completo)

| | Modelo Conceptual | Modelo de Análisis completo |
|---|---|---|
| ¿Qué incluye? | Solo clases, atributos, asociaciones | Modelo Conceptual + DSS + Contratos + Realización por cada CU |
| ¿Es un subconjunto de qué? | Es UNA de las piezas del Modelo de Análisis | Es el conjunto de todas las piezas |

**Pregunta que los distingue**: ¿me están preguntando solo por el vocabulario del dominio (Modelo Conceptual), o por TODO el paquete de artefactos de análisis de un caso de uso (Modelo de Análisis completo)?

---

## 11. CUS de Alto Nivel vs. CUS Expandido

| | Alto nivel | Expandido |
|---|---|---|
| Extensión | 2-4 líneas | Precondición + flujo básico numerado + alternativos + postcondición |
| ¿Cuándo se usa? | Negociación de alcance temprana | Base para Análisis (Temas 2-5) |
| ¿Alimenta el DSS/Modelo Conceptual? | No directamente | Sí, es la fuente principal |

**Pregunta que los distingue**: ¿me están pidiendo un resumen rápido para entender el alcance, o el detalle necesario para empezar a analizar?
