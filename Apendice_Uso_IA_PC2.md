# Apéndice de Uso de IA — PC2 Big Data & Data Analytics
## AD3005 · UTEC · Ciclo 2026-1
**Equipo:** Maria Fernanda Torres · Carla Quispe · Isabella Romero · Jorge Peña  
**Caso:** Movilidad Urbana — NYC Yellow Taxi 2022 + NYC Events API

---

> **Declaración de uso responsable**  
> El equipo utilizó inteligencia artificial generativa como herramienta de apoyo técnico en
> cinco momentos puntuales del proyecto. En todos los casos, el equipo verificó los outputs
> antes de incorporarlos al entregable, adaptó el código a las condiciones reales del entorno
> Databricks Serverless, y asumió la responsabilidad sobre el análisis, las interpretaciones y
> las recomendaciones estratégicas. Las conclusiones del informe son propias del equipo.
> Ningún fragmento del informe fue generado íntegramente por IA sin revisión y edición crítica.

---

## Formato de registro

Cada entrada documenta: herramienta utilizada · prompt enviado · propósito · cómo se validó el output.

---

## Prompt 1 — Diseño del pipeline ETL en PySpark sobre Databricks Serverless

**Herramienta:** Claude Sonnet 4.6 (claude.ai)  
**Integrante que lo ejecutó:** Maria Fernanda Torres (Ingeniera de Datos)

**Prompt enviado:**

> "Necesito construir un pipeline ETL completo en PySpark sobre Databricks Serverless para
> procesar los 8 archivos parquet mensuales del NYC Yellow Taxi 2022 (enero a agosto). El pipeline
> debe: (1) descargar los archivos desde la URL pública de NYC TLC si no existen en el volumen,
> (2) renombrar las columnas al español, (3) calcular duracion_minutos, fecha_exacta y
> hora_recogida como columnas derivadas, (4) filtrar outliers con rangos razonables para distancia,
> duración, pasajeros y tarifa, y (5) clasificar cada viaje en un bloque horario (Madrugada, Mañana,
> Tarde, Noche). El entorno es Databricks Serverless con PySpark — no hay acceso a librerías de
> ML de Spark por restricciones del whitelist. Genera el código limpio y sin comentarios."

**Propósito:**  
Acelerar la escritura del código base del ETL de taxis, evitando errores de sintaxis en la API de
PySpark y asegurando que la lógica de filtrado fuera coherente con los rangos operativos reales
de NYC (distancias < 100 millas, duraciones < 180 minutos, tarifas > 0).

**Cómo se validó el output:**  
- El código fue ejecutado celda por celda en Databricks. Se verificó el conteo de registros antes
  y después de cada filtro para confirmar que los outliers eliminados coincidían con lo observado
  en el EDA de PC1 (aprox. 120,000 registros nulos).
- Se comprobó que `bloque_horario` asignaba correctamente las franjas revisando 20 registros
  aleatorios con `df_taxis_clean.select("hora_recogida","bloque_horario").show(20)`.
- Se detectó que la IA usó el nombre `"Mañana"` con tilde, lo que generó inconsistencias en
  filtros posteriores. El equipo lo corrigió manualmente a `"Manana"` en todo el notebook para
  evitar problemas de encoding en Databricks Serverless.

---

## Prompt 2 — Corrección del error Py4JSecurityException en Nivel 2

**Herramienta:** Claude Sonnet 4.6 (claude.ai)  
**Integrante que lo ejecutó:** Carla Quispe (Analista de Datos)

**Prompt enviado:**

> "En la celda de correlaciones del Nivel 2 me sale este error en Databricks Serverless:
> Py4JError: Constructor public org.apache.spark.ml.feature.VectorAssembler is not whitelisted.
> El código usa VectorAssembler + Correlation.corr() de spark.ml.stat para calcular la matriz de
> correlacion de Pearson sobre 7 variables numericas del dataframe de taxis. Necesito un
> reemplazo que: (1) no use ninguna clase de spark.ml.feature, (2) sea estadisticamente
> equivalente, (3) funcione sobre el 100% de los datos sin cargar todo en memoria. Sugiere la
> mejor alternativa y explica por que funciona en Serverless."

**Propósito:**  
Resolver un bloqueo técnico crítico causado por las restricciones del whitelist de constructores
Java en Databricks Serverless, que impide instanciar `VectorAssembler` directamente desde Python.
El equipo necesitaba mantener la matriz de correlación completa para sustentar el hallazgo de que
`duracion_minutos` explica mejor la tarifa que `distancia_millas`.

**Cómo se validó el output:**  
- La IA propuso usar `.sample(fraction=0.05, seed=42).toPandas().corr()`. El equipo evaluó
  críticamente si el 5% de muestra era estadisticamente suficiente para 3.3 millones de registros.
- Se calculó el intervalo de confianza del coeficiente r con n=165,000 (5% de 3.3M) y se
  confirmó que el error estándar de r era menor a 0.003, lo que hace la muestra representativa.
- Se compararon los coeficientes r obtenidos con la muestra versus los reportados en el EDA
  de PC1 (calculados sobre marzo solamente). Los valores fueron consistentes: r(duracion, tarifa)
  cambió de 0.70 a 0.86 al incorporar los 8 meses, lo que el equipo interpretó correctamente
  como un efecto del mayor rango de distancias en el dataset completo versus solo marzo.
- Los resultados finales del informe usan los valores del dataset completo, con la nota metodológica
  de que se calcularon sobre muestra del 5% por restricciones del entorno.

---

## Prompt 3 — Diseño metodologico para la validacion de hipotesis H1, H2 y H3

**Herramienta:** Claude Sonnet 4.6 (claude.ai)  
**Integrante que lo ejecutó:** Jorge Peña (Analista de Negocio)

**Prompt enviado:**

> "Tenemos tres hipotesis del PC1 que debemos validar o rechazar con los datos de 8 meses:
> H1: Los dias con mayor concentracion de eventos deportivos juveniles tienen menor demanda
> de taxis (correlacion negativa esperada).
> H2: Los martes y jueves concentran picos de demanda superiores a 115k viajes, sabados y
> domingos valles inferiores a 93k — el ciclo semanal es el predictor mas robusto.
> H3: Los dias con mayor proporcion de Special Events tienen mayor demanda de taxis (+5%
> sobre la media).
> El entorno es Databricks Serverless. No tenemos scipy disponible de forma garantizada.
> Propone el codigo de validacion usando solo pandas y numpy, con el estadistico correcto para
> cada hipotesis y una conclusion clara VALIDADA / NO VALIDADA con el criterio de decision."

**Propósito:**  
Definir la metodología estadística correcta para cada hipótesis y generar código ejecutable sin
dependencias externas al stack pandas/numpy, que son las únicas librerías garantizadas en
Databricks Serverless además de PySpark.

**Cómo se validó el output:**  
- La IA propuso `numpy.corrcoef` para H1 y H3, y comparación de medianas para H2. El equipo
  revisó que `corrcoef` calcula el mismo coeficiente r de Pearson que `scipy.stats.pearsonr`
  pero sin el p-valor asociado. Se aceptó esta limitación explícitamente en el notebook.
- Para H2, la IA sugirió comparar medianas en lugar de medias porque el equipo había aplicado
  filtrado IQR previo. El equipo validó este criterio consultando el apunte de la Semana 8 del
  curso y confirmó que la mediana es más robusta ante el filtro de outliers.
- Los valores obtenidos (r_H1 = -0.613, r_H3 = +0.320, diferencia H2 = +12.5%) fueron
  contrastados contra los rangos esperados del PC1 y resultaron coherentes con los patrones
  visibles en los graficos exploratorios. El equipo tomo la decision final de VALIDAR las tres
  hipotesis con base en la interpretacion propia de los resultados, no en el criterio automatico
  de la IA.

---

## Prompt 4 — Diseño de visualizaciones D3.js con principios de Data Storytelling

**Herramienta:** Claude Sonnet 4.6 (claude.ai)  
**Integrante que lo ejecutó:** Isabella Romero (Lider de Proyecto)

**Prompt enviado:**

> "Necesito crear visualizaciones D3.js para un notebook de Databricks que se renderizan con
> displayHTML(). El contexto es un dashboard ejecutivo para la NYC TLC con 4 paneles alineados
> a las hipotesis H1, H2 y H3 del proyecto. Los principios a seguir son los del PDF de Data
> Storytelling del Prof. Morante: titulos activos con el insight directo, prueba de 3 segundos,
> color pre-atentivo para el dato clave, arco narrativo en cada panel. La paleta debe combinar
> dark navy (#111318) con olive/mostaza (#b8a820) como acento principal. IMPORTANTE: el SVG
> no puede usar clientWidth porque devuelve 0 en displayHTML — usar anchos fijos. Sin comentarios
> en el codigo JavaScript."

**Propósito:**  
Generar el codigo base de los 4 paneles D3.js del dashboard ejecutivo aplicando los principios
de storytelling del curso, con la restriccion tecnica especifica de Databricks Serverless donde
`svg.node().clientWidth` no funciona al momento de ejecucion del script.

**Cómo se validó el output:**  
- Cada panel fue ejecutado de forma independiente en Databricks antes de integrarlos al dashboard
  final. Se verifico que los SVG se renderizaban sin desbordamiento y que los tooltips eran
  visibles con `displayHTML()`.
- El equipo reviso que los titulos propuestos por la IA fueran consistentes con los hallazgos
  reales del analisis. Por ejemplo, la IA propuso "La duracion explica la tarifa (r=0.86)" como
  titulo del panel de correlaciones — el equipo verifico que el valor 0.86 correspondia al
  resultado real de la celda N2-1 antes de aprobarlo.
- Se descartaron dos variantes de color propuestas por la IA (violeta oscuro para el panel de
  tarifas) porque el equipo considero que no seguian el criterio de consistencia cromatica del
  modelo de dashboard de referencia. Se sustituyo por el olive-dim (#6b6012) propio de la paleta.
- Se corrigieron manualmente los titulos que contenian tildes o caracteres especiales que generaban
  problemas de encoding en el string HTML de Python.

---

## Prompt 5 — Estructuracion del analisis prescriptivo con supuestos explicitos

**Herramienta:** Claude Sonnet 4.6 (claude.ai)  
**Integrante que lo ejecutó:** Jorge Peña (Analista de Negocio)

**Prompt enviado:**

> "Necesito construir el Nivel 4 Prescriptivo del analisis. Tenemos dos escenarios:
> Escenario 1: incrementar la flota +15% los martes y jueves (dias pico segun H2).
> Escenario 2: reasignar el 30% de los conductores de la franja Madrugada hacia la franja Tarde.
> Para cada escenario necesito: (1) un supuesto operativo explicito y justificable, (2) el calculo
> cuantificado del impacto usando las variables reales del dataset, y (3) un indicador de exito
> medible. El analisis prescriptivo debe tener trazabilidad hacia los niveles 1, 2 y 3 del notebook.
> Los supuestos deben ser conservadores y verificables con datos publicos de la industria del taxi
> en NYC. Genera el codigo Python y el texto narrativo para el informe."

**Propósito:**  
Estructurar los dos escenarios prescriptivos con supuestos metodologicamente solidos y calculos
reproducibles, asegurando que cada recomendacion pudiera trazarse hacia evidencia especifica
de los niveles anteriores del analisis.

**Cómo se validó el output:**  
- El supuesto de 20 viajes/conductor/dia propuesto por la IA fue verificado contra el reporte
  anual de la NYC TLC (2022 TLC Factbook), que reporta un promedio de 18-22 viajes diarios
  por conductor activo. El equipo acepto 20 como valor representativo del rango.
- El calculo de reduccion del 13% en tiempo de espera fue revisado por el equipo usando la
  formula de relacion inversa oferta-demanda (1 - 1/(1+0.15)). Se confirmo que la formula
  era correcta pero se agrego explicitamente en el informe que es una aproximacion lineal,
  no una proyeccion de simulacion, para evitar sobrevender el resultado.
- La trazabilidad hacia niveles anteriores fue revisada celda por celda: se verifico que los
  valores numericos citados en el Nivel 4 (demanda_pico = 105,719 viajes/dia, brecha = 5,778)
  correspondian exactamente a los outputs de las celdas N1-1 y N2-3 del notebook.
- El equipo decidio no incluir una tercera recomendacion sobre pricing dinamico con la precision
  cuantitativa original que sugeria la IA (+8% ingresos), ya que ese calculo requeria datos de
  elasticidad de precio que no estaban en el dataset. Se reformulo como "+5% de ingreso promedio
  por viaje" con criterio mas conservador y verificable.

---

## Resumen de uso

| # | Prompt | Herramienta | Quien lo uso | Output incorporado | Modificaciones del equipo |
|---|--------|-------------|--------------|-------------------|---------------------------|
| 1 | Pipeline ETL PySpark | Claude Sonnet 4.6 | Mafer Torres | Celdas ETL-1 a ETL-4 del notebook | Correccion de encoding en "Manana", ajuste de rangos de filtro |
| 2 | Fix VectorAssembler | Claude Sonnet 4.6 | Carla Quispe | Celda N2-1 (correlacion con sample 5%) | Verificacion estadistica de representatividad de la muestra |
| 3 | Validacion H1/H2/H3 | Claude Sonnet 4.6 | Jorge Peña | Celda N2-4 (validacion explicita) | Cambio de criterio en H2 a medianas; interpretacion final propia |
| 4 | Dashboard D3.js | Claude Sonnet 4.6 | Isabella Romero | Celdas de visualizacion D3 y dashboard ejecutivo | Correcciones de color, titulos, encoding; descarte de 2 variantes |
| 5 | Analisis Prescriptivo | Claude Sonnet 4.6 | Jorge Peña | Celdas N4-1, N4-2, N4-3 y seccion 7 del informe | Reformulacion conservadora del escenario de pricing; verificacion de supuesto operativo |

---

## Reflexion sobre el uso de IA

El equipo concluye que la IA generativa fue util principalmente en dos tipos de situaciones:
como **acelerador tecnico** para resolver errores de entorno especificos de Databricks Serverless
(Prompts 2 y 4), y como **estructurador** para organizar analisis que el equipo ya habia
conceptualizado pero necesitaba traducir en codigo ejecutable (Prompts 1, 3 y 5).

En ningun caso el equipo acepto los outputs de la IA sin revision critica. Los errores mas
frecuentes fueron: uso de tildes y caracteres especiales en strings Python que generaban
SyntaxError, valores numericos citados sin verificar contra los resultados reales del notebook,
y supuestos operativos demasiado optimistas que el equipo modero para mantener la credibilidad
del analisis ante una audiencia de negocios.

La limitacion mas importante identificada es que la IA no tiene acceso al estado real del
entorno de ejecucion ni a los datos del proyecto, por lo que cualquier valor numerico en su
output debe considerarse un placeholder hasta ser reemplazado por el resultado real del analisis.

---

*Documento generado en cumplimiento de la politica de uso de IA del curso AD3005, seccion 7.1
de la rubrica del Proyecto Final 2026-1, UTEC.*
