# Phishing Detection con mBERT, XLM-RoBERTa y BETO

Pipeline de extraccion, limpieza, traduccion y entrenamiento de un
clasificador de phishing basado en modelos transformer, con evaluacion
cross-lingual entre ingles y espanol.

## Estado del proyecto

| Notebook | Estado | Descripcion |
|---|---|---|
| 01_extraccion.ipynb | Completo | Extrae texto de archivos .eml (phishing_pot), .mbox (Nazario) y carpetas de correo (Enron). Unifica las tres fuentes bajo el esquema text/label. |
| 02_eda.ipynb | Completo | Analisis exploratorio de datos: distribucion de longitud, idiomas presentes, tecnicas de ofuscacion HTML, contenido duplicado. No modifica los datos. |
| 03_limpieza.ipynb | Completo | Limpieza del corpus: correccion de codificacion, normalizacion de homoglifos, eliminacion de HTML y CSS residual, deduplicacion exacta y por plantilla, filtro de longitud minima, balanceo de clases. |
| 04_traduccion.ipynb | En curso | Traduccion del conjunto de entrenamiento de ingles a espanol mediante NLLB-200, para la segunda fase del experimento. |
| 05_entrenamiento.ipynb | Pendiente | Ajuste fino de mBERT y XLM-RoBERTa sobre el conjunto en ingles. |
| 06_entrenamiento_es.ipynb | Pendiente | Ajuste fino de mBERT, XLM-RoBERTa y BETO sobre el conjunto traducido al espanol. |
| 07_evaluacion_ood.ipynb | Pendiente | Evaluacion fuera de distribucion contra los conjuntos SpaPhish y SpearPhishMX, en espanol nativo. |

## Fuentes de datos

- phishing_pot: github.com/rf-peixoto/phishing_pot. Correos de phishing recolectados mediante honeypots, con datos sensibles anonimizados.
- Nazario Phishing Corpus: monkey.org/~jose/phishing. Corpus de referencia en la literatura de deteccion de phishing, distribuido en archivos .mbox por anio (2015-2025 utilizados en este trabajo).
- Enron Email Dataset: cs.cmu.edu/~enron. Correspondencia corporativa utilizada como clase legitima. El corpus completo contiene aproximadamente 500,000 mensajes (Klimt y Yang, 2004); se extrae una submuestra aleatoria mediante random subsampling para equilibrar su aporte frente a las demas fuentes.
- SpaPhish: data.mendeley.com/datasets/hz2d6gz7pc/5 (codigo en github.com/lbustio/spa_phish). Corpus de mensajes de phishing en espanol nativo, con anotacion de senales de persuasion (autoridad, prueba social, engano por similitud, compromiso y reciprocidad, distraccion). Reservado para la evaluacion fuera de distribucion.
- SpearPhishMX: data.mendeley.com/datasets/h4bxjk84jb/2. Diaz, Martinez Cruz y Bustio Martinez (2026), "Spear-Phishing en espanol: dataset de correos dirigidos con senales de personalizacion (SpearPhishMX)", Mendeley Data, V2, doi: 10.17632/h4bxjk84jb.2. Corpus de 3,006 correos de spear phishing corporativo en espanol latinoamericano, publicado por el Instituto Nacional de Astrofisica, Optica y Electronica (INAOE), Mexico. Reservado para la evaluacion fuera de distribucion, sin uso en ninguna etapa de entrenamiento ni validacion.

## Modelos

- mBERT (bert-base-multilingual-cased)
- XLM-RoBERTa (xlm-roberta-base)
- BETO (dccuchile/bert-base-spanish-wwm-cased), utilizado unicamente en la segunda fase, dado que es un modelo monolingue en espanol.

## Diseno experimental

Primera fase: entrenamiento en ingles, evaluacion fuera de distribucion en espanol.
Conjunto de entrenamiento: Enron, Nazario y phishing_pot en ingles, balanceado.
Modelos evaluados: mBERT y XLM-RoBERTa.
Conjuntos de evaluacion: SpaPhish y SpearPhishMX.

Segunda fase: traduccion del conjunto de entrenamiento a espanol mediante NLLB-200.
Modelos evaluados: mBERT, XLM-RoBERTa y BETO.
Conjuntos de evaluacion: los mismos dos conjuntos en espanol, SpaPhish y SpearPhishMX.

## Decisiones metodologicas

Orden de procesamiento. Las tres fuentes se unifican antes de aplicar la limpieza, siguiendo el procedimiento descrito en MeAJOR Corpus (Mendes et al., 2025), en lugar de limpiar cada fuente de forma independiente antes de la union.

Balanceo de clases. El numero de correos legitimos se ajusta al numero final de correos de phishing disponible despues de cada etapa de limpieza que reduce el numero de filas (deduplicacion, filtrado de idioma, filtrado por longitud). Este procedimiento sigue el criterio empleado en Salloum et al. (2022) y Afane et al. (2024), en lugar de fijar el balance de clases sobre el conteo inicial sin procesar.

Tratamiento de URLs. Las direcciones URL se conservan sin modificacion en el texto, dado que el tokenizador de subpalabras utilizado por los modelos transformer las descompone de forma adecuada, y su presencia constituye una senal relevante para la deteccion de phishing, conforme a lo reportado en MeAJOR Corpus.

Filtro de longitud minima. Se establece un minimo de 64 tokens por muestra, siguiendo el criterio de PEEK (Wang et al., 2024). No se establece un maximo explicito: se permite que el tokenizador trunque automaticamente las secuencias durante el entrenamiento, conforme a la practica habitual reportada en la literatura sobre deteccion de phishing con modelos transformer.

Seleccion del modelo de traduccion. Se utiliza NLLB-200 (Meta AI) en lugar de Google Translate, debido a las limitaciones de tasa de uso y la ausencia de respaldo academico verificable de este ultimo, y en lugar de Helsinki-NLP/OPUS-MT, que produjo resultados de menor calidad en una comparacion directa sobre una muestra del conjunto de datos. NLLB-200 reporta una mejora promedio del 44% sobre el estado del arte previo en los benchmarks descritos por sus autores (NLLB Team, 2022).

Seleccion de modelos de clasificacion. Se incluyen mBERT y XLM-RoBERTa en ambas fases por su naturaleza multilingue, lo que permite una comparacion directa del desempeno del mismo modelo en ingles y en espanol traducido. Se incorpora BETO unicamente en la segunda fase como modelo monolingue en espanol, para contrastar el desempeno de un modelo entrenado especificamente en ese idioma frente a los modelos multilingues.

Separacion de los conjuntos de evaluacion. SpaPhish y SpearPhishMX se mantienen completamente apartados del proceso de entrenamiento y validacion en ambas fases, conforme al criterio de evaluacion fuera de distribucion descrito en Gorman y Bedrick (2019), quienes senalan que el conjunto de prueba debe ser independiente y no contaminado por el proceso de entrenamiento para que los resultados sean validos.

## Limitaciones conocidas

La normalizacion de escrituras no latinas (cirilico, griego, armenio) se realiza mediante transliteracion fonetica, lo cual puede producir residuos menores en casos extremos de ofuscacion de caracteres.

El filtrado por idioma aplicado en la primera fase excluye correos de phishing escritos en idiomas distintos al ingles (principalmente aleman, portugues, frances y neerlandes), concentrados sobre todo en la fuente phishing_pot.

Una proporcion reducida de los registros conserva fragmentos de codigo CSS mal formado que no pudo eliminarse mediante expresiones regulares debido a la ausencia de una sintaxis consistente.

## Estructura del repositorio
phishing-detection/

├── README.md

├── notebooks/

│   ├── 01_extraccion.ipynb

│   ├── 02_eda.ipynb

│   ├── 03_limpieza.ipynb

│   ├── 04_traduccion.ipynb

│   ├── 05_entrenamiento.ipynb

│   ├── 06_entrenamiento_es.ipynb

│   └── 07_evaluacion_ood.ipynb

└── data/

    ├── dataset_limpio_ingles_balanceado.csv
