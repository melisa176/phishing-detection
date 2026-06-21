# Phishing Detection con mBERT/RoBERTa

Pipeline de extracción, limpieza, traducción y entrenamiento de un
clasificador de phishing basado en modelos transformer, con evaluación
cross-lingual entre inglés y español.

## Estado del proyecto

| Notebook | Estado | Descripcion |
|---|---|---|
| 01_extraccion.ipynb | Completo | Extrae texto de archivos .eml (phishing_pot), .mbox (Nazario) y carpetas de correo (Enron). Unifica las tres fuentes bajo el esquema text/label. |
| 02_eda.ipynb | Completo | Analisis exploratorio de datos: distribucion de longitud, idiomas presentes, tecnicas de ofuscacion HTML, contenido duplicado. No modifica los datos. |
| 03_limpieza.ipynb | Completo | Limpieza del corpus: correccion de codificacion, normalizacion de homoglifos, eliminacion de HTML y CSS residual, deduplicacion exacta y por plantilla, filtro de longitud minima, balanceo de clases. |
| 04_traduccion.ipynb | En curso | Traduccion del conjunto de entrenamiento de ingles a espanol mediante NLLB-200, para la segunda fase del experimento. |
| 05_entrenamiento.ipynb | Pendiente | Ajuste fino de mBERT y RoBERTa sobre ambos conjuntos (ingles y espanol traducido). |
| 06_evaluacion_ood.ipynb | Pendiente | Evaluacion fuera de distribucion contra los conjuntos SpaPhish y SpearPhishMX, en espanol nativo. |

## Fuentes de datos

- phishing_pot: github.com/rf-peixoto/phishing_pot. Correos de phishing recolectados mediante honeypots, con datos sensibles anonimizados.
- Nazario Phishing Corpus: monkey.org/~jose/phishing. Corpus de referencia en la literatura de deteccion de phishing, distribuido en archivos .mbox por anio (2015-2025 utilizados en este trabajo).
- Enron Email Dataset: cs.cmu.edu/~enron. Correspondencia corporativa utilizada como clase legitima.
- SpaPhish y SpearPhishMX: conjuntos de evaluacion en espanol nativo, reservados exclusivamente para la evaluacion final fuera de distribucion. No se utilizan en ninguna etapa de entrenamiento ni validacion.

## Diseno experimental

Primera fase: entrenamiento en ingles, evaluacion fuera de distribucion en espanol.
Conjunto de entrenamiento: Enron, Nazario y phishing_pot en ingles.
Evaluacion: SpaPhish y SpearPhishMX.

Segunda fase: traduccion del conjunto de entrenamiento a espanol mediante NLLB-200, entrenamiento y evaluacion en espanol sobre los mismos conjuntos de prueba.

## Decisiones metodologicas

Orden de procesamiento. Las tres fuentes se unifican antes de aplicar la limpieza, siguiendo el procedimiento descrito en MeAJOR Corpus (Mendes et al., 2025), en lugar de limpiar cada fuente de forma independiente antes de la union.

Balanceo de clases. El numero de correos legitimos se ajusta al numero final de correos de phishing disponible despues de cada etapa de limpieza que reduce el numero de filas (deduplicacion, filtrado de idioma, filtrado por longitud). Este procedimiento sigue el criterio empleado en Salloum et al. (2022) y Afane et al. (2024), en lugar de fijar el balance de clases sobre el conteo inicial sin procesar.

Tratamiento de URLs. Las direcciones URL se conservan sin modificacion en el texto, dado que el tokenizador de subpalabras utilizado por BERT y RoBERTa las descompone de forma adecuada, y su presencia constituye una senal relevante para la deteccion de phishing, conforme a lo reportado en MeAJOR Corpus.

Filtro de longitud minima. Se establece un minimo de 64 tokens por muestra, siguiendo el criterio de PEEK (Wang et al., 2024). No se establece un maximo explicito: se permite que el tokenizador trunque automaticamente las secuencias durante el entrenamiento, conforme a la practica habitual reportada en la literatura sobre deteccion de phishing con modelos BERT y RoBERTa.

Seleccion del modelo de traduccion. Se utiliza NLLB-200 (Meta AI) en lugar de Google Translate, debido a las limitaciones de tasa de uso y la ausencia de respaldo academico verificable de este ultimo, y en lugar de Helsinki-NLP/OPUS-MT, que produjo resultados de menor calidad en una comparacion directa sobre una muestra del conjunto de datos. NLLB-200 reporta una mejora promedio del 44% sobre el estado del arte previo en los benchmarks descritos por sus autores (NLLB Team, 2022).

## Limitaciones conocidas

La normalizacion de escrituras no latinas (cirilico, griego, armenio) se realiza mediante transliteracion fonetica, lo cual puede producir residuos menores en casos extremos de ofuscacion de caracteres.

El filtrado por idioma aplicado en la primera fase excluye correos de phishing escritos en idiomas distintos al ingles (principalmente aleman, portugues, frances y neerlandes), concentrados sobre todo en la fuente phishing_pot.

Una proporcion reducida de los registros conserva fragmentos de codigo CSS mal formado que no pudo eliminarse mediante expresiones regulares debido a la ausencia de una sintaxis consistente.

## Estructura del repositorio
