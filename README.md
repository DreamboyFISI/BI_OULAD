# Proyecto de Inteligencia de Negocios — OULAD

![Arquitectura de Inteligencia de Negocios](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSvJws50emve1mfFJLvwRgkALEFbEzRaA-zLL8oGhoFkw&s=10)

Solución académica de Inteligencia de Negocios desarrollada a partir del conjunto de datos **Open University Learning Analytics Dataset (OULAD)**. El proyecto implementa un proceso ETL, un Data Warehouse en Azure SQL Database, modelos analíticos tabulares y reportes de Power BI orientados al análisis del rendimiento, la participación y las evaluaciones de los estudiantes.

## Objetivo

Construir una solución de BI que permita analizar el comportamiento académico de los estudiantes mediante tres DataMarts:

1. **Rendimiento académico**: matrículas, bajas, resultados, evaluaciones y actividad general.
2. **Participación en el VLE**: interacciones de los estudiantes con los recursos del entorno virtual.
3. **Evaluaciones**: entregas, calificaciones, retrasos y resultados de las evaluaciones.

## Arquitectura

```text
Dataset OULAD
      │
      ▼
Base operacional en Azure SQL
      │
      ▼
SSIS — Extracción, transformación y carga
      │
      ▼
Data Warehouse en Azure SQL
      │
      ├── DataMart 1: Rendimiento académico
      ├── DataMart 2: Participación en el VLE
      └── DataMart 3: Evaluaciones
      │
      ├── SSAS Tabular — Modelos analíticos
      └── Power BI — Reportes
```

Los resultados del ETL permanecen almacenados en Azure SQL. Por ello, los reportes de Power BI pueden consultar el Data Warehouse sin ejecutar Visual Studio, SSIS o SSAS. SSIS solo debe ejecutarse cuando sea necesario reconstruir o actualizar la información del almacén.

## Tecnologías utilizadas

- Azure SQL Database
- SQL Server Management Studio
- Microsoft Visual Studio
- SQL Server Integration Services (SSIS)
- SQL Server Analysis Services Tabular (SSAS)
- Microsoft Power BI Desktop
- Git y GitHub

## Estructura del repositorio

```text
Proyecto_final_OLAD/
├── README.md
├── Reportes/
│   ├── DM1_Rendimiento.pbix
│   ├── DM2_Participacion.pbix
│   └── DM3_Evaluaciones.pbix
├── SSIS/
│   └── OULAD_DM1_ETL/
│       └── DM1_RendimientoAcad/
└── SSAS/
    ├── OULAD_DM1_Modelo/
    ├── OULAD_DM2_Modelo/
    └── OULAD_DM3_Modelo/
```

> Algunos archivos y proyectos pueden encontrarse todavía en desarrollo. La sección de estado muestra el avance actual.

## Modelo dimensional

El Data Warehouse utiliza una base de datos central con dimensiones conformadas y esquemas separados para identificar cada DataMart.

### Dimensiones comunes

| Tabla | Descripción |
|---|---|
| `comun.DIM_ESTUDIANTE` | Información demográfica y versiones históricas del estudiante. |
| `comun.DIM_CURSO` | Módulo, presentación, convocatoria y duración del curso. |
| `comun.DIM_TIEMPO` | Tiempo relativo a las presentaciones académicas. |

### DataMart 1 — Rendimiento académico

| Tabla | Tipo |
|---|---|
| `dm1.FACT_RESULTADO_ACADEMICO` | Hechos |
| `dm1.DIM_RESULTADO` | Dimensión |
| `dm1.DIM_TIEMPO_MATRICULA` | Vista de rol temporal |
| `dm1.DIM_TIEMPO_BAJA` | Vista de rol temporal |

Este DataMart permite analizar resultados finales, matrículas, bajas, créditos, evaluaciones, actividad y participación acumulada.

### DataMart 2 — Participación en el VLE

| Tabla | Tipo |
|---|---|
| `dm2.FACT_INTERACCION_VLE` | Hechos |
| `dm2.DIM_SITIO_VLE` | Dimensión |

Este DataMart permite analizar clics, eventos e interacciones con los recursos del entorno virtual de aprendizaje.

### DataMart 3 — Evaluaciones

| Tabla | Tipo |
|---|---|
| `dm3.FACT_ENTREGA_EVALUACION` | Hechos |
| `dm3.DIM_EVALUACION` | Dimensión |
| `dm3.DIM_ESTADO_ENTREGA` | Dimensión |
| `dm3.DIM_RESULTADO_EVALUACION` | Dimensión |

Este DataMart permite analizar entregas, notas, ponderaciones, retrasos y resultados de las evaluaciones.

## Proyecto SSIS

El proceso ETL está organizado en un proyecto unificado con los siguientes paquetes:

| Orden | Paquete | Función |
|---:|---|---|
| 00 | `00_Orquestador.dtsx` | Coordina la ejecución completa del ETL. |
| 01 | `01_DimensionComunes.dtsx` | Carga las dimensiones conformadas. |
| 10 | `10_DM1_Rendimiento.dtsx` | Carga el DataMart de rendimiento académico. |
| 20 | `20_DM2_Participacion.dtsx` | Carga el DataMart de participación en el VLE. |
| 30 | `30_DM3_Evaluaciones.dtsx` | Carga el DataMart de evaluaciones. |

Orden general de ejecución:

```text
00_Orquestador
    └── 01_DimensionComunes
          └── 10_DM1_Rendimiento
                └── 20_DM2_Participacion
                      └── 30_DM3_Evaluaciones
```

## Proyectos SSAS

Los modelos tabulares se organizan por DataMart:

- `OULAD_DM1_Modelo`: rendimiento académico.
- `OULAD_DM2_Modelo`: participación en el VLE.
- `OULAD_DM3_Modelo`: evaluaciones.

Los modelos incluyen relaciones, jerarquías y medidas DAX para facilitar el análisis de los indicadores de cada área.

## Reportes Power BI

Cada DataMart tendrá un reporte independiente:

| Archivo | Contenido |
|---|---|
| `DM1_Rendimiento.pbix` | Indicadores de rendimiento académico. |
| `DM2_Participacion.pbix` | Indicadores de participación en el VLE. |
| `DM3_Evaluaciones.pbix` | Indicadores de evaluaciones y entregas. |

Los reportes colaborativos se conectan directamente a Azure SQL en modo **Importar**. Cada PBIX debe contener:

- Las tablas correspondientes a su DataMart.
- Las relaciones del modelo estrella.
- Las medidas DAX necesarias.
- Las jerarquías de navegación.
- Los datos importados y las páginas del reporte.

Los datos importados permiten abrir, filtrar y modificar el reporte sin tener SSAS activo. Solo se necesita acceso a Azure SQL para ejecutar una actualización.

## Configuración local

### Requisitos

- Visual Studio con las extensiones de SSIS y Analysis Services Projects.
- SQL Server Analysis Services en modo tabular, si se trabajará con SSAS local.
- SQL Server Management Studio.
- Power BI Desktop.
- Git.

### Conexión a Azure SQL

Las credenciales no se almacenan en el repositorio. Cada integrante debe utilizar una cuenta autorizada, preferiblemente con permisos de solo lectura para Power BI.

Datos requeridos:

```text
Servidor: proporcionado de manera privada
Base de datos: proporcionada de manera privada
Usuario: proporcionado de manera privada
Contraseña: nunca debe almacenarse en Git
```

Para configurar una conexión en Power BI:

1. Abrir el archivo PBIX correspondiente.
2. Ir a **Archivo → Opciones y configuración → Configuración de origen de datos**.
3. Seleccionar la conexión de Azure SQL.
4. Introducir las credenciales asignadas.
5. Pulsar **Actualizar**.

## Colaboración con Git

Los archivos PBIX son binarios y Git no puede fusionar automáticamente dos versiones del mismo archivo. Para evitar conflictos, cada integrante debe trabajar únicamente sobre el reporte que tenga asignado.

Flujo recomendado:

1. Clonar el repositorio.
2. Crear una rama personal o asociada al DataMart.
3. Trabajar únicamente en el PBIX asignado.
4. Guardar y comprobar el reporte.
5. Crear un commit descriptivo.
6. Enviar la rama al repositorio remoto.
7. Crear un Pull Request para revisión.

Ejemplos de ramas:

```text
feature/dm1-rendimiento
feature/dm2-participacion
feature/dm3-evaluaciones
```

Si un archivo PBIX supera el límite de tamaño permitido por GitHub, deberá administrarse mediante Git LFS.

## Seguridad

No deben publicarse en GitHub:

- Contraseñas o credenciales cifradas.
- Tokens, claves o cadenas de conexión con secretos.
- Usuarios administradores de Azure SQL.
- Archivos de configuración personal de Visual Studio.
- Archivos compilados o temporales.

Las conexiones de SSIS deben utilizar parámetros y el nivel de protección `DontSaveSensitive`, de modo que las contraseñas sean proporcionadas en el entorno de ejecución y no queden almacenadas en los paquetes.

Como mínimo, deben excluirse del repositorio:

```gitignore
.vs/
bin/
obj/
*.user
*.suo
*.bim_USER.settings
*.ispac
```

## Fuente de datos

El proyecto utiliza el conjunto de datos **Open University Learning Analytics Dataset (OULAD)**:

- Kaggle: <https://www.kaggle.com/datasets/rocki37/open-university-learning-analytics-dataset>
- Sitio del proyecto OULAD: <https://analyse.kmi.open.ac.uk/open_dataset>

## Estado del proyecto

- [x] Importación del dataset a Azure SQL.
- [x] Diseño del Data Warehouse.
- [x] Desarrollo del proceso ETL.
- [x] Creación de los tres DataMarts.
- [x] Modelo SSAS del DataMart 1.
- [x] Modelo SSAS del DataMart 2.
- [x] Modelo SSAS del DataMart 3.
- [x] Reporte Power BI del DataMart 1.
- [x] Reporte Power BI del DataMart 2.
- [x] Reporte Power BI del DataMart 3.
- [ ] Documentación final.

## Equipo

| Integrante | Responsabilidad |
|---|---|
| Iris y Juan Carlos | Coordinación, ETL y Data Warehouse |
| Michael | Reporte del DataMart 1 |
| Bryan | Reporte del DataMart 2 |
| Álvaro | Reporte del DataMart 3 |

## Uso académico

Proyecto desarrollado con fines académicos para el curso de Inteligencia de Negocios.
