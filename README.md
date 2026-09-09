# Workshop-001
Estructura del repositorio
.
├── database/
│   └── candidates.csv                  # Datos de origen
├── ETL - Notebook/
│   └── ETL_Workshop01.ipynb            # Colab del proceso de Extract, Transform, Load al DW
├── SQL_Queries_KPIs_DW/
│   ├── SQL_KPIs.ipynb                  # SQL Queries + gráficos de los KPIs
├── Star_Schema_Diagram/
│   ├── Esquema_Estrella.png            # Diagrama del modelo dimensional
│   └── Explanation_Star.txt            # Por qué se diseñó así
├── README.md
└── .gitignore

Cómo ejecutarlo

Los notebooks son para correr en Google Colab, cargando los datos desde Google Drive:

Subir el database candidates.csv a Drive, en Mi unidad > Colab Notebooks > workshop01.
Abre ETL_Workshop01.ipynb en Colab y corre todas las celdas en orden. Esto monta en Drive, limpia los datos, aplica la regla de HIRED y genera candidates_dw.db en esa misma carpeta.
Abre SQL_KPIs.ipynb y corre todas las celdas: consulta el DW que se generó en el paso anterior y genera los 4 gráficos de KPIs.

No se necesita instalar un motor de base de datos aparte: se usó SQLite como Data Warehouse.

Regla de negocio: HIRED
Un candidato se considera contratado (HIRED) cuando:
Code Challenge Score >= 7  Y  Technical Interview >= 7

Modelo dimensional (Esquema Estrella)
Ver Star_Schema_Diagram/Esquema_Estrella.png y la explicación completa en Star_Schema_Diagram/Explanation_Star.txt.

factApplication: una fila por candidato/aplicación. Contiene las métricas (yoe, code_challenge_score, technical_interview, hired) y las llaves foráneas hacia cada dimensión.
dimCandidate: datos de identidad del candidato — first_name, last_name, email, country.
dimSeniority: catálogo de niveles de seniority.
dimTechnology: catálogo de tecnologías postuladas.
dimApplicationDate: catálogo de fechas de aplicación (year, month, day).

Por qué no hay una dim_country aparte: el país es un dato muy básico (solo el nombre, sin ciudad ni dirección, ni código postal ni nada más), así que separarlo en su propia tabla agregaba una tabla y un JOIN extra sin aportar nada — por eso quedó directamente en dimCandidate.

Proceso ETL
Extract (ETL_Workshop01.ipynb): se lee candidates.csv con pandas.read_csv.
Transform:
Limpieza: nombres de columnas normalizados, tipos corregidos, duplicados eliminados, nulos rellenados ("Unknown" para categóricas, 0 para numéricas, filas sin fecha se descartan).
Regla de negocio: se agrega la columna hired.
Load: se crean las tablas del esquema estrella en SQLite (candidates_dw.db) y se cargan con DataFrame.to_sql.

KPIs y visualizaciones (consultando el DW)
Todo esto está en SQL_Queries_KPIs_DW/SQL_KPIs.ipynb.

KPI	Tipo de gráfico
Contrataciones por tecnología	Pie chart
Contrataciones por año	Barra horizontal
Contrataciones por seniority	Barra
Contrataciones por país (USA, Brazil, Colombia, Ecuador) por año	Multilínea

Decisiones y supuestos
Se usó SQLite como Data Warehouse por simplicidad: no requiere servidor ni configuración adicional y pandas/sqlite3 lo soportan de forma nativa.
Los registros sin Application Date se descartaron porque no se pueden ubicar en la dimensión de tiempo, pues si lo reemplazamos por un "Unknown" ahí no serviría para ningún KPI
Los valores nulos en Seniority, Country o Technology se reemplazaron por "Unknown" en vez de eliminarse, para no perder información de los puntajes de esos candidatos.
Los puntajes nulos (Code Challenge Score / Technical Interview) se llenaron con 0, lo que automáticamente los excluye de la regla HIRED.
country se incluyó dentro de dimCandidate en vez de crear una dim_country aparte (ver justificación completa en Explanation_Star.txt).
