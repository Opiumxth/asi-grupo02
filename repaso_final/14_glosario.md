# 14 — Glosario Maestro

> **Propósito**: Definición precisa de cada término del curso con referencia cruzada al archivo donde se profundiza.

---

## A

| Término | Definición | Archivo |
|---------|-----------|---------|
| **Actor de Negocio** | Entidad EXTERNA al negocio que interactúa con él (cliente, proveedor, regulador). | 🔗 [04](04_modelo_negocio.md) |
| **Actor del Sistema** | Entidad EXTERNA al software que interactúa con él (persona, otro sistema, dispositivo). | 🔗 [07](07_casos_uso.md) |
| **Agregación** | Relación todo-parte donde la parte puede existir independientemente. Rombo hueco (◇). | 🔗 [08](08_modelo_conceptual.md) |
| **Análisis** | Etapa que investiga el dominio del PROBLEMA. Produce modelo conceptual, DSS, contratos. | 🔗 [10](10_analisis_diseno.md) |
| **AOO** | Análisis Orientado a Objetos. Identificar conceptos del dominio y documentarlos. | 🔗 [08](08_modelo_conceptual.md) |
| **Artefacto** | Cualquier producto de trabajo en RUP: documento, modelo, código, ejecutable. | 🔗 [03](03_rup.md) |
| **Asociación** | Relación semántica entre dos clases que indica conexión significativa. | 🔗 [08](08_modelo_conceptual.md) |
| **Atributo** | Valor de datos lógico de un objeto. En modelo de dominio: solo tipos simples. | 🔗 [08](08_modelo_conceptual.md) |

## B

| Término | Definición | Archivo |
|---------|-----------|---------|
| **Boundary** | Estereotipo UML para clases de interfaz (formularios, pantallas). | 🔗 [12](12_colaboracion.md) |
| **Business Entity** | Ver: Entidad de Negocio. | 🔗 [04](04_modelo_negocio.md) |
| **Business Use Case** | Ver: Caso de Uso de Negocio. | 🔗 [04](04_modelo_negocio.md) |

## C

| Término | Definición | Archivo |
|---------|-----------|---------|
| **Caso de Uso de Negocio (CUN)** | Proceso del negocio que produce un resultado de valor observable para un Actor de Negocio. | 🔗 [04](04_modelo_negocio.md) |
| **Caso de Uso del Sistema (CUS)** | Descripción narrativa de un proceso donde un actor interactúa con el sistema para lograr un objetivo. | 🔗 [07](07_casos_uso.md) |
| **Ciclo de Vida** | Secuencia de etapas que atraviesa un producto software desde su concepción hasta su retiro. | 🔗 [02](02_sistemas_informacion.md) |
| **Clase Abstracta** | Clase que no puede tener instancias directas; toda instancia debe ser de alguna subclase. Cursiva en UML. | 🔗 [08](08_modelo_conceptual.md) |
| **Clase Asociación** | Clase cuyos atributos pertenecen a una asociación entre dos clases, no a ninguna de las dos. | 🔗 [09](09_clases_objetos.md) |
| **Clase Conceptual** | Idea, cosa u objeto del dominio del problema. Tiene símbolo, definición y extensión. | 🔗 [08](08_modelo_conceptual.md) |
| **Composición** | Relación todo-parte donde la parte NO existe sin el todo. Rombo relleno (◆). Multiplicidad del todo = 1. | 🔗 [08](08_modelo_conceptual.md) |
| **Contrato de Operación** | Documento que describe los cambios de estado al ejecutar una operación del sistema. | 🔗 [11](11_secuencia_contratos.md) |
| **Control** | Estereotipo UML para clases coordinadoras de procesos. | 🔗 [12](12_colaboracion.md) |
| **Crisis del Software** | Conjunto de problemas (sobrecostos, retrasos, fallos) que evidenciaron la necesidad de procesos de desarrollo. | 🔗 [02](02_sistemas_informacion.md) |

## D

| Término | Definición | Archivo |
|---------|-----------|---------|
| **Diagrama de Actividades** | Diagrama UML que modela el flujo de un proceso paso a paso (nodos de acción, decisiones, forks). | 🔗 [05](05_uml.md) |
| **Diagrama de Clases** | Diagrama UML que muestra clases, atributos, métodos y relaciones. Usado en análisis (conceptual) y diseño. | 🔗 [09](09_clases_objetos.md) |
| **Diagrama de Colaboración** | Diagrama UML que muestra interacciones entre objetos internos mediante mensajes numerados. | 🔗 [12](12_colaboracion.md) |
| **Diagrama de Objetos** | Diagrama UML que muestra instancias concretas de clases con valores específicos en un momento dado. | 🔗 [09](09_clases_objetos.md) |
| **Diagrama de Secuencia del Sistema (DSS)** | Diagrama que muestra eventos del actor al sistema (caja negra). Fase de análisis. | 🔗 [11](11_secuencia_contratos.md) |
| **Diseño** | Etapa que crea la SOLUCIÓN. Produce diagramas de colaboración, clases de diseño, esquema BD. | 🔗 [10](10_analisis_diseno.md) |

## E

| Término | Definición | Archivo |
|---------|-----------|---------|
| **Elaboración** | Segunda fase de RUP. Se establece la arquitectura base y se mitigan riesgos técnicos. | 🔗 [03](03_rup.md) |
| **Entidad de Negocio** | Objeto de información que el negocio crea, gestiona o referencia (Factura, Producto, Pedido). | 🔗 [04](04_modelo_negocio.md) |
| **Entity** | Estereotipo UML para clases de datos persistentes. | 🔗 [12](12_colaboracion.md) |
| **Especificación/Descripción** | Clase que describe la información de otra clase, independiente de sus instancias (EspecificaciónDelProducto). | 🔗 [08](08_modelo_conceptual.md) |
| **Extend** | Relación entre CUS donde el CU base puede o NO ejecutar al CU extendido (condicional). | 🔗 [07](07_casos_uso.md) |

## F-G

| Término | Definición | Archivo |
|---------|-----------|---------|
| **Flujo Básico** | Secuencia principal de pasos en un CUS (escenario de éxito). | 🔗 [07](07_casos_uso.md) |
| **Flujo Alternativo** | Variantes o excepciones del flujo básico. | 🔗 [07](07_casos_uso.md) |
| **FURPS+** | Modelo de clasificación: Functionality, Usability, Reliability, Performance, Supportability + restricciones. | 🔗 [06](06_requerimientos.md) |
| **Generalización** | Relación "es-un-tipo-de" entre superclase y subclase. Flecha con triángulo hueco (△). | 🔗 [08](08_modelo_conceptual.md) |
| **Glosario** | Diccionario de términos del dominio del problema. Artefacto de RUP. | 🔗 [04](04_modelo_negocio.md) |

## H-I

| Término | Definición | Archivo |
|---------|-----------|---------|
| **Heurística de Abbott** | Técnica de 1983 para identificar clases mediante análisis de frases nominales en texto. | 🔗 [08](08_modelo_conceptual.md) |
| **Include** | Relación entre CUS donde el CU base SIEMPRE ejecuta al CU incluido. | 🔗 [07](07_casos_uso.md) |
| **Inicio** | Primera fase de RUP. Definir alcance, viabilidad, riesgos principales. | 🔗 [03](03_rup.md) |
| **Iterativo** | Enfoque donde se repiten fases para refinar el producto progresivamente. | 🔗 [02](02_sistemas_informacion.md) |

## L-M

| Término | Definición | Archivo |
|---------|-----------|---------|
| **Larman, Craig** | Autor de "Applying UML and Patterns". Referencia principal del curso para análisis y diseño OO. | Todos |
| **Modelo Conceptual** | Sinónimo de Modelo de Dominio. Diagrama de clases que muestra conceptos del mundo real. | 🔗 [08](08_modelo_conceptual.md) |
| **Modelo de Dominio** | Artefacto UML más importante del AOO. Muestra clases conceptuales, asociaciones y atributos. | 🔗 [08](08_modelo_conceptual.md) |
| **Multiplicidad** | Cuántas instancias de una clase pueden asociarse con una instancia de otra (1, *, 0..1, 1..*). | 🔗 [08](08_modelo_conceptual.md) |
| **MVC** | Patrón Modelo-Vista-Controlador que separa datos, interfaz y control. | 🔗 [10](10_analisis_diseno.md) |

## N-O

| Término | Definición | Archivo |
|---------|-----------|---------|
| **Navegabilidad** | Dirección en la que se pueden enviar mensajes en una asociación (flecha). | 🔗 [08](08_modelo_conceptual.md) |
| **Operación del Sistema** | Acción que el sistema ofrece como respuesta a un evento del actor. Se identifica en el DSS. | 🔗 [11](11_secuencia_contratos.md) |

## P-R

| Término | Definición | Archivo |
|---------|-----------|---------|
| **Paquete** | Agrupación lógica de elementos UML relacionados para mejorar comprensión. | 🔗 [08](08_modelo_conceptual.md) |
| **Postcondición** | Estado del sistema DESPUÉS de ejecutar una operación (creación, modificación, asociación). | 🔗 [11](11_secuencia_contratos.md) |
| **Precondición** | Estado requerido ANTES de ejecutar un CUS u operación. | 🔗 [07](07_casos_uso.md) |
| **Proceso de Software** | Conjunto de actividades, métodos y prácticas para desarrollar y mantener software (IEEE 12207). | 🔗 [02](02_sistemas_informacion.md) |
| **Realización de CUN** | Muestra CÓMO se ejecuta internamente un CUN: workers, entidades, flujo. | 🔗 [04](04_modelo_negocio.md) |
| **Regla del 100%** | 100% de la definición de la superclase debe aplicarse a la subclase. | 🔗 [08](08_modelo_conceptual.md) |
| **Requerimiento** | Condición o capacidad que debe poseer un sistema (IEEE 610). | 🔗 [06](06_requerimientos.md) |
| **RUP** | Rational Unified Process. Iterativo, centrado en arquitectura, dirigido por CU. | 🔗 [03](03_rup.md) |

## S-T

| Término | Definición | Archivo |
|---------|-----------|---------|
| **Salto Semántico** | Diferencia entre la representación mental del dominio y la representación en software. El modelo de dominio lo reduce. | 🔗 [08](08_modelo_conceptual.md) |
| **Sistema de Información** | Conjunto organizado de elementos que procesan datos para producir información útil para decisiones. | 🔗 [02](02_sistemas_informacion.md) |
| **SRS/ERS** | Especificación de Requisitos de Software. Documento con todos los requisitos clasificados. | 🔗 [06](06_requerimientos.md) |
| **Trazabilidad** | Capacidad de rastrear un artefacto hasta su origen (requisito → CUS → DSS → contrato → clase). | 🔗 [01](01_mapa_general.md) |

## U-W

| Término | Definición | Archivo |
|---------|-----------|---------|
| **UML** | Unified Modeling Language. Lenguaje visual estándar para modelar sistemas OO. | 🔗 [05](05_uml.md) |
| **Visión** | Documento de alto nivel que captura el problema, stakeholders y características del producto. | 🔗 [06](06_requerimientos.md) |
| **Worker de Negocio** | Rol INTERNO al negocio que participa en la ejecución de un CUN (Cajero, Almacenero). | 🔗 [04](04_modelo_negocio.md) |
| **Workflow** | Flujo de trabajo en RUP. Secuencia de actividades que produce un resultado. 9 flujos: 6 de ingeniería + 3 de soporte. | 🔗 [03](03_rup.md) |
