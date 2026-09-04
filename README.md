# Predicción de la potencia de un molino de bolas

Código, metodología y resultados computacionales del Trabajo Fin de Máster:

**Predicción de la potencia consumida por un molino de bolas en la industria minera mediante modelos de series temporales**

**Autor:** Jordan Gómez Espinoza
**Programa:** Máster Universitario en Ciencia de Datos y Big Data
**Universidad:** Universidad Internacional de Valencia (VIU)
**Año:** 2026

## Descripción

Este repositorio contiene la implementación y evaluación de tres modelos para el pronóstico multihorizonte de la potencia eléctrica consumida por el motor principal de un molino de bolas:

* ARIMAX.
* eXtreme Gradient Boosting (XGBoost).
* Temporal Fusion Transformer (TFT).

Los modelos utilizan 30 minutos de información histórica para pronosticar la trayectoria de la potencia durante los siguientes 15 minutos.

## Objetivo

Desarrollar y comparar modelos estadísticos, de aprendizaje automático y aprendizaje profundo para anticipar el comportamiento de la potencia del molino, evaluando su desempeño frente a un benchmark de persistencia mediante un protocolo temporal común.

## Variables utilizadas

| Variable | Descripción                                                 |       Unidad | Función              |
| -------- | ----------------------------------------------------------- | -----------: | -------------------- |
| `FECHA`  | Marca temporal de la observación                            | Fecha y hora | Índice temporal      |
| `POT`    | Potencia activa consumida por el motor principal del molino |           MW | Variable objetivo    |
| `F80`    | Tamaño característico de la alimentación                    |           µm | Covariable histórica |
| `T`      | Tonelaje de alimentación                                    |          t/h | Covariable histórica |
| `AA`     | Flujo de agua de alimentación                               |        L/min | Covariable histórica |
| `AD`     | Flujo de agua de dilución                                   |        L/min | Covariable histórica |

No se utilizaron covariables futuras conocidas. En cada origen de pronóstico, los modelos emplearon únicamente información disponible hasta ese instante.

## Diseño experimental

* Frecuencia original de adquisición: 5 segundos.
* Frecuencia de modelamiento: 15 segundos.
* Ventana histórica: 30 minutos, equivalente a 120 observaciones.
* Horizonte predictivo: 15 minutos, equivalente a 60 pasos.
* Entrenamiento: 113.633 observaciones.
* Validación: 17.280 observaciones, equivalentes a 3 días.
* Prueba: 17.280 observaciones, equivalentes a 3 días.
* Evaluación mediante *rolling origin* con trayectorias consecutivas y no solapadas.
* Benchmark: persistencia.
* Semilla de reproducibilidad: 42.

La base auditada contenía 444.580 mediciones originales. Después de agregar grupos completos de tres mediciones, se obtuvieron 148.193 observaciones reales con una frecuencia de 15 segundos.

El conjunto de prueba permaneció aislado durante la configuración y el entrenamiento de los modelos y se utilizó únicamente para la evaluación final.

## Notebooks

| Contenido                      | Notebook                                    |
| ------------------------------ | ------------------------------------------- |
| Análisis exploratorio de datos | `EDA_FINAL_TESIS_MOLINO_1D_15MIN.ipynb`     |
| Modelo ARIMAX                  | `ARIMAX_FINAL_TESIS_MOLINO_1D_15MIN.ipynb`  |
| Modelo XGBoost                 | `XGBOOST_FINAL_TESIS_MOLINO_1D_15MIN.ipynb` |
| Temporal Fusion Transformer    | `TFT_FINAL_TESIS_MOLINO_1D_15MIN.ipynb`     |

El notebook de análisis exploratorio contiene:

* Auditoría de integridad y continuidad temporal.
* Estadísticos descriptivos.
* Análisis de rangos y distribuciones.
* Evolución temporal de las variables.
* Análisis ACF y PACF de POT.
* Documentación de la división cronológica.

Los notebooks de modelamiento contienen:

* Carga y validación del conjunto de datos.
* Preprocesamiento temporal.
* División cronológica en entrenamiento, validación y prueba.
* Construcción de ventanas históricas.
* Entrenamiento y validación de los modelos.
* Evaluación mediante *rolling origin*.
* Comparación frente al benchmark de persistencia.
* Métricas globales y por horizonte.
* Evaluación probabilística mediante intervalos P10–P90.
* Generación de tablas y figuras.
* Exportación de predicciones, métricas y configuraciones.

## Configuración de los modelos

### ARIMAX

* Orden final: ARIMAX(3,0,1).
* Variables exógenas: F80, T, AA y AD.
* Selección del orden de diferenciación mediante la prueba ADF.
* Análisis preliminar mediante ACF y PACF.
* Selección de \(p\) y \(q\) mediante AIC entre los modelos convergentes.
* Pronóstico de las covariables mediante persistencia de su último valor conocido.

### XGBoost

* Estrategia directa multihorizonte con 60 regresores independientes.
* Un regresor para cada paso futuro de 15 segundos.
* 600 características retardadas por muestra.
* Configuración regularizada fijada previamente.
* Parada temprana de 50 rondas con un máximo de 1.200 árboles.

No se realizó una búsqueda sistemática de hiperparámetros para XGBoost. La parada temprana determinó el número efectivo de árboles de cada regresor mediante el conjunto de validación.

### Temporal Fusion Transformer

* `input_chunk_length = 120`.
* `output_chunk_length = 60`.
* Cuantiles estimados: P10, P50 y P90.
* Parada temprana mediante la pérdida de validación.
* Selección y recuperación del mejor *checkpoint*.
* Pronóstico conjunto de los 60 pasos futuros.

## Resultados principales en prueba

| Modelo       | MAE (MW) | RMSE (MW) | MAPE (%) |       R² |
| ------------ | -------: | --------: | -------: | -------: |
| TFT          | 0,004628 |  0,007281 | 0,201784 | 0,902561 |
| Persistencia | 0,004978 |  0,007862 | 0,216878 | 0,886402 |
| XGBoost      | 0,005115 |  0,008055 | 0,223616 | 0,880769 |
| ARIMAX       | 0,005337 |  0,007901 | 0,232805 | 0,885277 |

TFT obtuvo el mejor desempeño global en el conjunto de prueba. Frente a persistencia, redujo el MAE en 7,03 % y el RMSE en 7,39 %.

En el horizonte exacto de 15 minutos, TFT obtuvo un MAE de 0,007504 MW, frente a 0,008361 MW de persistencia, lo que representa una reducción de 10,26 %. XGBoost redujo el MAE de persistencia en 2,01 %, mientras que ARIMAX presentó un MAE 9,54 % superior.

El bootstrap pareado por trayectoria estimó una diferencia media de MAE de 0,000350 MW a favor de TFT, con un intervalo de confianza del 95 % de [0,000018; 0,000669] MW. Como el intervalo no incluyó cero, la mejora de TFT frente a persistencia estuvo estadísticamente respaldada durante el periodo evaluado.

Estos resultados se limitan al molino y al periodo analizados. Su estabilidad y relevancia operacional deben confirmarse con periodos independientes y mediante pruebas en modo sombra.

## Evaluación probabilística

La cobertura empírica de los intervalos P10–P90 en el conjunto de prueba fue:

| Modelo  | Método                              | Cobertura | Amplitud media |
| ------- | ----------------------------------- | --------: | -------------: |
| TFT     | Regresión cuantílica                |   80,97 % |    0,013700 MW |
| XGBoost | Cuantiles de residuos de validación |   75,29 % |    0,012787 MW |
| ARIMAX  | Intervalo gaussiano                 |   70,56 % |    0,012699 MW |

TFT presentó la cobertura global más próxima al nivel nominal del 80 %. Sin embargo, en el horizonte exacto de 15 minutos su cobertura descendió a 73,26 %, por lo que la calibración debe verificarse en periodos adicionales.

## Análisis complementario de baja potencia

Aunque el problema principal fue de regresión, se evaluó de manera complementaria el evento:

```text
POT < 2,28 MW
```

Las predicciones continuas se transformaron en una clasificación binaria para calcular precisión, recall y F1. El umbral se utilizó únicamente con fines analíticos y no representa una alarma, un interlock ni un límite de seguridad operacional.

## Requisitos

Los experimentos fueron ejecutados con:

* Python 3.8.20.
* Pandas 1.5.3.
* NumPy 1.24.4.
* Darts 0.31.0.
* PyTorch 2.4.1.
* XGBoost 2.1.4.
* Statsmodels 0.14.1.
* Scikit-learn 1.2.2.
* CUDA 12.1.
* NVIDIA Quadro RTX 5000.

Las dependencias pueden instalarse mediante:

```bash
pip install -r requirements.txt
```

## Preparación de los datos

Por motivos de confidencialidad, el archivo industrial utilizado en el estudio no se incluye en este repositorio.

Para reproducir el flujo completo con datos autorizados, el archivo debe denominarse:

```text
1D_POT_2.csv
```

El archivo debe contener las siguientes columnas:

```text
FECHA, POT, F80, T, AA, AD
```

## Ejecución

Se recomienda ejecutar los notebooks en el siguiente orden:

1. `EDA_FINAL_TESIS_MOLINO_1D_15MIN.ipynb`
2. `ARIMAX_FINAL_TESIS_MOLINO_1D_15MIN.ipynb`
3. `XGBOOST_FINAL_TESIS_MOLINO_1D_15MIN.ipynb`
4. `TFT_FINAL_TESIS_MOLINO_1D_15MIN.ipynb`

Para iniciar Jupyter Notebook:

```bash
jupyter notebook
```

Cada notebook crea sus propias carpetas de resultados y exporta las configuraciones, predicciones, métricas, figuras y modelos serializados correspondientes.

## Confidencialidad y reproducibilidad

El conjunto de datos industrial se mantiene fuera del repositorio debido a restricciones de confidencialidad. Los notebooks permiten reproducir la metodología, el preprocesamiento, el entrenamiento y la evaluación cuando se dispone de un archivo autorizado con la estructura requerida.

Los resultados publicados corresponden exclusivamente al conjunto de datos, al periodo y al equipo analizados. No deben interpretarse como límites operacionales universales ni como recomendaciones automáticas de control.

## Referencias principales

* Box, G. E. P., y Jenkins, G. M. (1976). *Time Series Analysis: Forecasting and Control*. Holden-Day.
* Chen, T., y Guestrin, C. (2016). XGBoost: A scalable tree boosting system. *Proceedings of the 22nd ACM SIGKDD International Conference on Knowledge Discovery and Data Mining*, 785–794. https://doi.org/10.1145/2939672.2939785
* Hyndman, R. J., y Athanasopoulos, G. (2021). *Forecasting: Principles and Practice* (3.ª ed.). OTexts. https://otexts.com/fpp3/
* Lim, B., Arik, S. O., Loeff, N., y Pfister, T. (2021). Temporal Fusion Transformers for interpretable multi-horizon time series forecasting. *International Journal of Forecasting, 37*(4), 1748–1764. https://doi.org/10.1016/j.ijforecast.2021.03.012
