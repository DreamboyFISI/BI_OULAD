# Reporte DM2 – Participación en el VLE

Proyecto de Power BI (formato PBIP) correspondiente al DataMart 2. Analiza la interacción de los estudiantes con los recursos del entorno virtual de aprendizaje (VLE) y se conecta al Data Warehouse `DM1_RendimientoAcad` en modo **Importar**.

**Responsable:** Bryan Ávila Jara.

## Páginas

| Página | Contenido |
|---|---|
| Resumen de participación | Indicadores generales, evolución semanal de los clics y comparación por módulo y presentación. |
| Recursos y actividades | Distribución de los clics por grupo y tipo de actividad, y los diez recursos más utilizados. |
| Perfil del estudiante | Participación según región, índice IMD, educación previa, edad y discapacidad. |

## Preguntas de negocio atendidas

Las preguntas provienen del documento de diseño del DataMart 2.

| # | Pregunta de negocio | Objeto visual que la responde |
|---:|---|---|
| 1 | ¿Cuántos clics genera cada módulo y cada convocatoria? | *Clics por estudiante según módulo*; *Clics por presentación* |
| 2 | ¿Qué tipos de actividad concentran la mayor participación? | *Clics por grupo de actividad*; matriz *Grupo → tipo de actividad* |
| 3 | ¿Cómo evoluciona la actividad semana a semana? | *Clics por semana del curso* |
| 4 | ¿Qué proporción de la actividad ocurre antes del inicio del curso? | Tarjetas *% Pre-curso* y *Clics Pre-curso* |
| 5 | ¿Qué estudiantes no registran actividad en el VLE? | Tarjetas *Estudiantes Activos* y *Matrículas Activas* |
| 6 | ¿Qué recursos son los más utilizados? | *Recursos más utilizados* (los diez primeros) |
| 7 | ¿La participación se relaciona con la aprobación? | No se responde en este reporte: el resultado final pertenece al DM1 (`dm1.DIM_RESULTADO`). Se obtiene mediante *drill-across* entre el DM1 y el DM2 a través de las dimensiones conformadas. |
| 8 | ¿Qué módulos o perfiles requieren acompañamiento por baja actividad? | Gráficos por módulo, índice IMD, educación previa, edad y discapacidad |

## Modelo semántico

- **Tablas:** `dm2 FACT_INTERACCION_VLE` y `dm2 DIM_SITIO_VLE` y las dimensiones conformadas `comun DIM_ESTUDIANTE`, `comun DIM_CURSO` y `comun DIM_TIEMPO`. El modelo respeta el esquema estrella del DataMart 2 y no incluye tablas de otros DataMarts.
- **Relaciones:** cuatro relaciones de varios a uno, desde la tabla de hechos hacia las dimensiones, con la misma estructura de claves del Data Warehouse.
- **Jerarquías:**
  - Actividad (grupo → tipo → sitio)
  - Curso (año → convocatoria → módulo → presentación)
  - Tiempo (semana → día)
  - Perfil (región → educación → IMD)

## Medidas DAX principales

| Medida | Definición | Valor total |
|---|---|---:|
| Clics Totales | `SUM(sum_click)` | 39 605 099 |
| Eventos | `SUM(num_eventos)` | 10 655 280 |
| Estudiantes Activos | `CALCULATE(DISTINCTCOUNT(id_student), tabla de hechos)` | 26 074 |
| Clics por Estudiante | Clics Totales / Estudiantes Activos | 1 518,9 |
| Matrículas Activas | Combinaciones distintas estudiante–curso con actividad | 29 228 |
| Días Activos Promedio | Combinaciones estudiante–curso–día / Matrículas Activas | 61,9 |
| % Pre-curso | Clics con `dia_offset < 0` / Clics Totales | 5,42 % |
| Clics por Matrícula | Clics Totales / Matrículas Activas | 1 355,0 |

Las medidas no aditivas (estudiantes, matrículas, días y sitios) se calculan con conteos distintos, no por suma. Todos los valores fueron contrastados con consultas SQL sobre el Data Warehouse.

## Conexión

El origen se define mediante dos parámetros de Power Query:

- `ServidorSQL`: por defecto, el servidor Azure del grupo.
- `BaseDatos`: `DM1_RendimientoAcad`.

Para trabajar con una copia local, use **Transformar datos → Editar parámetros** y cambie `ServidorSQL` a `localhost`. Las credenciales no se almacenan en el repositorio.

## Uso

Abra `DM2_Participacion.pbip` (o `Reportes/DM2_Participacion.pbix`), ingrese las credenciales de Azure SQL y pulse **Actualizar**.
