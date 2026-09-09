Workshop 001: ETL

## Estructura del repositorio

```text
.
├── database/
│   └── candidates.csv                  # Datos de origen 
├── ETL_notebook/
│   └── ETL_Workshop01.ipynb            # Proceso Extract, Transform, Load al DW
├── SQL_Queries_KPIs_DW/
│   ├── SQL_KPIs.ipynb                  # SQL Queries + gráficos de los KPIs
├── Star_Schema_Diagram/
│   ├── Esquema_Estrella.png            # Diagrama del modelo dimensional
│   └── Explanation_Star.txt            # Justificación del diseño arquitectónico
├── README.md
└── .gitignore

```

## Cómo ejecutarlo

Los notebooks están diseñados para ejecutarse en **Google Colab**, montando los datos directamente desde Google Drive. No se necesita instalar algo aparte, ya que se utilizó **SQLite** como Data Warehouse.

1. **Preparar los datos:** Sube el archivo `candidates.csv` a tu Google Drive, específicamente en la ruta: `Mi unidad > Colab Notebooks > workshop01`.
2. **Ejecutar ETL:** Abre el archivo `ETL_Workshop01.ipynb` en Colab y ejecuta todas las celdas en orden. Este script conectará con Drive, limpiará los datos, aplicará las reglas de negocio y generará la base de datos `candidates_dw.db` en la misma carpeta.
3. **Generar Reportes:** Abre `SQL_KPIs.ipynb` y ejecuta todas las celdas. Este notebook consulta el Data Warehouse recién creado y renderiza los gráficos de los KPIs solicitados.

## Regla de negocio: HIRED

Un candidato se considera contratado (`HIRED = 1`) única y exclusivamente cuando cumple ambas condiciones:

> **Code Challenge Score >= 7**  Y  **Technical Interview >= 7**

##  Modelo dimensional (Esquema Estrella)

El diseño prioriza la velocidad de consulta analítica. *(Ver diagrama en `Star_Schema_Diagram/Esquema_Estrella.png` y justificación completa en `Explanation_Star.txt`)*.

* **`factApplication`:** Tabla central. Una fila por aplicación. Contiene exclusivamente métricas numéricas (`yoe`, `code_challenge_score`, `technical_interview`, `hired`) y llaves foráneas.
* **`dimCandidate`:** Datos de identidad del candidato (`first_name`, `last_name`, `email`, `country`).
* **`dimSeniority`:** Catálogo de niveles de seniority.
* **`dimTechnology`:** Catálogo de tecnologías postuladas.
* **`dimApplicationDate`:** Catálogo de fechas de aplicación separadas por `year`, `month`, `day`.

** Nota sobre Country:** El país (`country`) se incluyó directamente dentro de `dimCandidate` en lugar de crear una dimensión `dimCountry` separada. Al ser un dato básico (sin ciudad, dirección ni código postal), separarlo solo agregaría un JOIN extra a las consultas sin aportar valor jerárquico.

##  Proceso ETL

1. **Extract (`ETL_Workshop01.ipynb`):** Lectura del archivo `candidates.csv` utilizando `pandas.read_csv`.
2. **Transform:**
* **Limpieza:** Normalización de nombres de columnas, corrección de tipos de datos y eliminación de duplicados.
* **Manejo de nulos:** Se rellenaron con `"Unknown"` para variables categóricas y con `0` para numéricas. Las filas sin fecha se descartaron.
* **Transformación:** Aplicación de la regla de negocio para generar la columna `hired`.


3. **Load:** Creación de las tablas del esquema estrella y volcado de datos en SQLite (`candidates_dw.db`) mediante `DataFrame.to_sql`.

## KPIs y visualizaciones (Consultando el DW)

Todo el análisis se realiza en `SQL_Queries_KPIs_DW/SQL_KPIs.ipynb`.

| KPI | Tipo de Gráfico |
| --- | --- |
| Contrataciones por tecnología | Pie chart |
| Contrataciones por año | Barra horizontal |
| Contrataciones por seniority | Gráfico de Barras |
| Contrataciones por país (USA, Brazil, Colombia, Ecuador) por año | Multilínea |

## Decisiones y Supuestos

Durante el desarrollo se tomaron las siguientes decisiones técnicas:

* **Integridad del Tiempo:** Los registros sin *Application Date* se descartaron. Un valor nulo no se puede ubicar en la dimensión de tiempo ni reemplazar por un "Unknown" útil, invalidando el registro para análisis temporales.
* **Preservación de Datos:** Los valores nulos en *Seniority*, *Country* o *Technology* se reemplazaron por `"Unknown"` en vez de eliminarse. Esto evita perder la información valiosa de los puntajes de esos candidatos.
* **Corrección de Outliers (Scores):** Los puntajes *Code Challenge* y *Technical Interview* tienen un rango lógico de 0 a 10. Se identificó un único outlier con valor `100`; se asumió como un error tipográfico ("fat finger") y se corrigió a `10` para no perder la fila. (Un puntaje negativo u otro valor irracional sin corrección obvia se trataría como dato faltante).
* **Scores Faltantes:** Los puntajes nulos se rellenaron con `0`. Esto garantiza matemáticamente que el candidato sea excluido automáticamente por la regla `HIRED`.

