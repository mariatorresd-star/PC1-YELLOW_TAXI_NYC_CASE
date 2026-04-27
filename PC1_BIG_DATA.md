# PC1 BIG DATA & DATA ANALYTICS

**Universidad de Ingeniería y Tecnología — UTEC**

---

**Nombre del caso:**
Optimización de la demanda y precios en movilidad urbana mediante análisis de datos.

---

## Integrantes

- Maria Fernanda Torres
- Carla Quispe
- Isabella Romero
- Jorge Peña

## Roles

- **Líder de proyecto (Isabella):** Coordinación general y cumplimiento de entregables.
- **Analista de Datos (Carla):** Limpieza, procesamiento y exploración de datos.
- **Ingeniero de Datos (Mafer):** Integración de fuentes y manejo de datasets.
- **Analista de negocio (Jorge):** Interpretación de resultados y generación de insights.

---

## 1. Descripción del problema de negocio

El problema de negocio se centra en la variabilidad de la demanda y los precios en servicios de transporte urbano, tomando como referencia plataformas similares a Uber o Lyft en la ciudad de New York. Estas empresas enfrentan dificultades para anticipar con precisión los cambios en la demanda, lo que genera insuficiencias como escasez de conductores en horas pico o exceso de oferta en momentos de baja demanda.

Este problema es relevante porque impacta directamente en la experiencia del usuario y en la rentabilidad de la plataforma. Una mala predicción puede provocar tiempos de espera elevados, precios poco competitivos o desaprovechamiento de recursos. Además, factores externos como el clima, la hora, el día y la ocurrencia de eventos masivos influyen significativamente en el comportamiento de los usuarios, pero no siempre son incorporados de manera efectiva en la toma de decisiones.

En este contexto, analizar cómo estas variables afectan la demanda y los precios permite mejorar la asignación de conductores y optimizar los modelos de pricing dinámico. Esto no solo incrementa la eficiencia operativa, sino que también mejora la satisfacción del cliente y maximiza los ingresos de la empresa.

---

## 2. Descripción de los datos

**Fuente principal:** El dataset principal corresponde a NYC Yellow Taxi Trips, disponible en plataformas como NYC Open Data y Kaggle. Este conjunto de datos contiene información detallada sobre viajes de taxi en Nueva York, incluyendo fecha y hora, ubicación de inicio y fin, distancia recorrida, duración del viaje y tarifa pagada.

**Fuentes de enriquecimiento:**

- Datos sobre eventos públicos en Nueva York de la NYC Events API.
- Información referencial de precios de plataformas como Uber o Lyft.

**Número de registros:** El dataset cuenta con millones de registros, lo que permite realizar análisis robustos y detectar patrones significativos en el comportamiento de la demanda.

**Variables clave:**

| Variable | Descripción |
|---|---|
| Fecha y hora de viaje | Permite analizar patrones temporales como horas pico o días de la semana. |
| Ubicación | Identifica las zonas de alta y baja demanda. |
| Distancia y duración | Relacionadas con el costo y la eficiencia del servicio. |
| Tarifa total | Variable clave para el análisis de precios. |
| Condiciones climáticas | Factor externo que influye en la demanda. |
| Eventos en la ciudad | Incrementan la demanda en zonas específicas. |

---

## 3. EDA Inicial

> El código completo y los gráficos interactivos se encuentran en el notebook adjunto: **PC1 BD y DA .ipynb**

### a. Visualización 1: Distribución de viajes por hora del día

La demanda de taxis alcanza su punto más bajo entre las 3 y 6 am, incrementándose progresivamente hasta el pico entre las 18h y 20h. Esto indica que las plataformas deben concentrar su flota en horario vespertino y reducir operaciones en la madrugada.

### b. Visualización 2: Distribución de tarifas

La tarifa promedio es de **$13.72 USD**. El 50% de los viajes cuesta menos de $9.50, confirmando que dominan los trayectos cortos dentro de Manhattan. Existe una cola de outliers hasta $52.00 (percentil 99), probablemente viajes al aeropuerto JFK o LaGuardia.

### c. Visualización 3: Tipos de eventos públicos

Los eventos deportivos juveniles y especiales dominan el calendario de NYC en marzo de 2022, con más de 15,000 y 17,000 registros respectivamente. Los fines de semana con competencias deportivas son momentos clave de potencial incremento en la demanda de transporte.

### d. Visualización 4: Mapa de calor de correlaciones

La duración del viaje (`duracion_minutos`) tiene una correlación alta con la tarifa (**0.86**), siendo el principal determinante del precio. Sorprendentemente, la distancia (`trip_distance`) tiene una correlación baja (**0.14**), posiblemente por la congestión vehicular en Manhattan.

### e. Visualización 5: Evolución diaria de la demanda

La demanda oscila entre 90,000 y 120,000 viajes diarios en marzo 2022, con caídas pronunciadas cada 7 días coincidiendo con los domingos. Los picos más altos ocurren martes y jueves, lo que permite planificar la disponibilidad de conductores con anticipación.

### f. Visualización 6: Correlación taxis vs. eventos

El coeficiente de Pearson de **-0.38** indica una correlación negativa moderada. Los días con más eventos no generan más viajes de taxi, posiblemente porque los eventos deportivos en parques atraen público que llega caminando o en metro. Este hallazgo será profundizado en la Etapa 2.

### Hallazgos clave del EDA

- El ciclo semanal es el patrón más robusto de la demanda.
- La tarifa está determinada principalmente por la duración, no la distancia.
- 117,814 filas tenían valores nulos antes de la limpieza (3.2% del total).
- La relación entre eventos y demanda de taxis es más compleja de lo esperado.

---

## 4. Hipótesis del negocio

### Hipótesis A

- **H₀:** El tipo de evento público no tiene efecto sobre la demanda diaria de taxis.
- **H₁:** Los días con mayor concentración de eventos deportivos juveniles presentan menor demanda de taxis, porque este tipo de evento moviliza grupos familiares que prefieren vehículo propio sobre tarifas cortas de taxi.

### Hipótesis B

- **H₀:** El día de la semana no influye significativamente en la demanda diaria de taxis.
- **H₁:** Los días martes y jueves concentran consistentemente los picos de demanda (>115k viajes), mientras que sábado y domingo generan los valles (<93k), siendo el ciclo semanal el predictor más robusto de demanda.

### Hipótesis C

- **H₀:** El tipo de evento público no diferencia el comportamiento de la demanda de taxis.
- **H₁:** Los días con mayor proporción de Special Events sobre eventos deportivos presentan mayor demanda de taxis (+5% sobre la media), porque su público objetivo es adulto, urbano y dependiente de transporte por aplicación.

---

## 5. Etapa 2 — Plan de trabajo (Semanas 7–14)

| Semana | Maria Fernanda  | Isabella  | Mateo | Carla  |
|---|---|---|---|---|
| **Semana 7** | Integrar definitivamente las dos fuentes (NYC Taxi + NYC Events). Verificar joins y calidad de datos combinados. | Coordinar validación de las 3 hipótesis del PC1. Documentar decisiones metodológicas en el notebook. | Redactar conclusiones de cada hipótesis (aceptada / rechazada) en lenguaje de negocio. | Ejecutar análisis bivariado: cruces de día de semana × demanda, tipo evento × viajes, hora × tarifa. |
| **Semana 8** | Crear variables derivadas: franjas horarias, flag "evento deportivo", segmentos de zona (aeropuerto vs Manhattan). | Revisar avance del análisis y alinear con rúbrica PC2 (Nivel 2). Actualizar bitácora de contribución. | Redactar los primeros insights de negocio: qué segmentos tienen más demanda y por qué. | Comparar segmentos: zonas alta vs baja demanda, días laborables vs fines de semana, eventos deportivos vs especiales. |
| **Semana 9** | Preparar dataset limpio para el modelo. Aplicar clustering K-Means sobre zonas/franjas horarias. Calcular silhouette score. | Validar que el modelo responda a una hipótesis del PC1. Alinear con el equipo el enfoque de la presentación. | Traducir los clusters a lenguaje de negocio: "Cluster A = zona aeropuerto, hora pico, tarifa alta". Proponer nombres descriptivos. | Construir regresión lineal simple: predecir tarifa a partir de duración + franja + tipo de evento. Reportar R². |
| **Semana 10** | Exportar datos procesados y preparar conexión a Looker Studio o Power BI. Configurar fuente de datos. | Definir las 4+ preguntas de negocio que cada visual debe responder. Revisar diseño del dashboard. | Redactar los títulos del dashboard como preguntas de negocio (ej. "¿Cuándo concentrar la flota?"). | Construir visuals 1 y 2: demanda por hora/día (heatmap semanal) y mapa de zonas de alta demanda. |
| **Semana 11** | Conectar los resultados del modelo al dashboard. Añadir filtros interactivos (zona, franja horaria, tipo de evento). | Revisar el dashboard completo con ojos de audiencia no técnica. Validar que cada visual tenga conclusión explícita. | Revisar consistencia visual del dashboard (paleta, títulos, etiquetas). Agregar anotaciones clave. | Construir visuals 3 y 4: distribución de tarifas por cluster y correlación eventos vs demanda. |
| **Semana 12** | Cuantificar los 2 escenarios "¿qué pasaría si…?". Ej: "Si se aumenta la flota un 15% los martes, se reduce el tiempo de espera X min". | Redactar borrador del informe final .MD: secciones 1-3 (problema, datos, análisis). | Formular 3 recomendaciones estratégicas priorizadas, con acción, responsable e indicador de éxito. | Completar y comentar el notebook completo. Asegurarse de que sea reproducible de inicio a fin. |
| **Semana 13** | Revisar que el notebook esté limpio, comentado en español y organizado por nivel analítico. | Completar informe .MD (secciones 4-5: insights y recomendaciones). Coordinar ensayo de presentación. | Escribir Reflexión Individual (200–400 palabras) y subir a GraderPortal antes de la presentación. | Escribir Reflexión Individual (200–400 palabras) y subir a GraderPortal antes de la presentación. |
| **Semana 14** | Exponer sección de datos y modelado (slides 3 + hallazgo 2). Responder preguntas técnicas del docente. | Exponer problema de negocio y cierre / próximos pasos (slides 2 + 12). Moderar el turno de Q&A. | Exponer escenarios y recomendaciones estratégicas (slides 8-11). Liderar el Q&A de negocio. | Exponer hallazgos del EDA y del análisis diagnóstico (slides 4-7). Defender los gráficos del dashboard. |
