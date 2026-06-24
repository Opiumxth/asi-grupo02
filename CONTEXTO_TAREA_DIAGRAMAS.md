# Tarea: Modelado UML - Gestión de Pedidos (Análisis de Sistemas)

## Contexto del Proyecto
- **Curso**: Análisis de Sistemas de Información
- **Enfoque metodológico**: RUP (Rational Unified Process)
- **Herramientas**: Rational Rose (para exportar diagramas)
- **Lenguaje de modelado**: UML 2.0
- **Diagrama previo completado**: Visión de Negocio (documento "Vision de Negocio - Grupo 02.pdf") 

## Tarea Actual
Desarrollar tres diagramas UML para el proceso elegido:
1. **Diagrama de Casos de Uso**
2. **Diagrama de Actividades** 
3. **Diagrama de Objetos**

Todos basados en el proceso: **Gestión de Pedidos**

## Proceso Elegido: Gestión de Pedidos
**Definición**: Abarca desde la creación de un pedido por parte del cliente hasta su entrega final, incluyendo validación, preparación, asignación de repartidor y seguimiento del estado.

## Avance 01 Completado
Archivo: `AVANCE_DIAGRAMAS_GESTION_PEDIDOS.md`

### Contenido entregable
1. **Diagrama de Casos de Uso (UML formal)**
   - Actores: Cliente, Tienda, Repartidor, Administrador de operaciones
   - Casos: Realizar pedido, Validar pedido, Gestionar pedido, Asignar repartidor, Recoger pedido, Entregar pedido, Consultar estado, Notificar incidencia
   - Relaciones: `<<include>>` (flujos obligatorios) y `<<extend>>` (flujos opcionales/excepción)
   - Límite del sistema definido

2. **Diagrama de Actividades**
   - Flujo principal de "Gestionar pedido"
   - Decisiones clave: Validación de pedido, Disponibilidad de repartidor
   - Nodos inicial/final
   - Nota: Implementar con swimlanes por actor en Rational Rose

3. **Diagrama de Objetos (Instancias)**
   - Muestra instancias concretas: `cliente01:Cliente`, `pedido1024:Pedido`, `tiendaCentro:Tienda`, etc.
   - Relaciones entre instancias y sus líneas de vida
   - Fotografía puntual del sistema en un momento determinado

## Criterios RUP/UML a Validar
- [ ] Casos de uso con nombres en infinitivo
- [ ] Actor primario identificado para cada caso de uso
- [ ] Límite explícito del sistema
- [ ] Actividades con guardas en decisiones
- [ ] Objetos en formato `nombre:Clase` (instancia)
- [ ] Consistencia de nombres entre los tres diagramas

## Plan Próximo: Avance 02
Ampliar el documento actual con:
- Swimlanes por actor en diagrama de actividades
- Escenarios alternos (cancelación, falta de stock)
- Especificación textual de casos de uso (formato RUP):
  - Nombre
  - Actor primario
  - Precondiciones
  - Flujo básico
  - Flujos alternos
  - Postcondiciones
- Mayor detalle de máquina de estados del Pedido

## Plan de Exportación a Rational Rose
1. Crear paquete "Modelo de Casos de Uso"
2. Dibujar diagrama de casos de uso formal
3. Crear actividades con swimlanes
4. Crear diagrama de objetos con instancias
5. Revisar consistencia y trazabilidad entre artefactos

## Notas Importantes
- NO incluir tecnología o stack técnico en este nivel de análisis
- Mantener enfoque en requisitos funcionales del negocio
- Los diagramas se presentan en Mermaid como borrador/avance, pero son equivalentes UML formales
- La revisión final del profesor será en la próxima clase (fecha sin confirmar)

## Archivos Generados
- `AVANCE_DIAGRAMAS_GESTION_PEDIDOS.md` - Documento con los tres diagramas y explicación RUP
- `CONTEXTO_TAREA_DIAGRAMAS.md` - Este archivo de memoria del contexto
