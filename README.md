# Predicción de la potencia de un molino de bolas

Código y resultados computacionales del Trabajo Fin de Máster:

**Predicción de la potencia consumida por un molino de bolas en la industria minera mediante modelos de series temporales**

Autor: Jordan Gómez Espinoza  
Programa: Máster Universitario en Ciencia de Datos y Big Data  
Universidad: Universidad Internacional de Valencia (VIU)  
Año: 2026

## Descripción

Este repositorio contiene la implementación y evaluación de tres modelos para el pronóstico multihorizonte de la potencia eléctrica consumida por el motor principal de un molino de bolas:

- ARIMAX.
- eXtreme Gradient Boosting (XGBoost).
- Temporal Fusion Transformer (TFT).

Los modelos utilizan 30 minutos de información histórica para pronosticar la trayectoria de la potencia durante los siguientes 15 minutos.

## Objetivo

Desarrollar y comparar modelos de series temporales y aprendizaje automático para anticipar el comportamiento de la potencia del molino, evaluando su desempeño frente a un benchmark de persistencia.

## Variables

| Variable | Descripción | Función |
|---|---|---|
| `FECHA` | Marca temporal | Índice temporal |
| `POT` | Potencia eléctrica del molino | Variable objetivo |
| `F80` | Tamaño característico de alimentación | Covariable histórica |
| `T` | Tonelaje de alimentación | Covariable histórica |
| `AA` | Agua de alimentación | Covariable histórica |
| `AD` | Agua de descarga | Covariable histórica |

## Diseño experimental

- Frecuencia original: 5 segundos.
- Frecuencia de modelamiento: 15 segundos.
- Ventana histórica: 30 minutos, equivalente a 120 observaciones.
- Horizonte predictivo: 15 minutos, equivalente a 60 pasos.
- Validación: 3 días.
- Prueba: 3 días.
- Evaluación rolling origin con trayectorias no solapadas.
- Benchmark: persistencia.
- Umbral operacional evaluado: `POT < 2,28 MW`.
- Semilla de reproducibilidad: 42.

La base auditada contiene 444.580 mediciones originales y 148.193 intervalos completos de 15 segundos. La división cronológica comprende 113.633 observaciones de entrenamiento, 17.280 de validación y 17.280 de prueba.

## Notebooks

| Modelo | Notebook |
|---|---|
| ARIMAX | `ARIMAX_FINAL_TESIS_MOLINO_1D_15MIN.ipynb` |
| XGBoost | `XGBOOST_FINAL_TESIS_MOLINO_1D_15MIN.ipynb` |
| TFT | `TFT_FINAL_TESIS_MOLINO_1D_15MIN.ipynb` |

Cada notebook contiene:

- Auditoría del conjunto de datos.
- Preprocesamiento temporal.
- División cronológica.
- Entrenamiento y validación.
- Evaluación rolling origin.
- Comparación con persistencia.
- Métricas globales y por horizonte.
- Evaluación del evento `POT < 2,28 MW`.
- Generación de tablas y figuras.
- Exportación de resultados.

## Resultados principales en prueba

| Modelo | MAE (MW) | RMSE (MW) | MAPE (%) | R² |
|---|---:|---:|---:|---:|
| TFT | 0,004972 | 0,007743 | 0,217015 | 0,889821 |
| Persistencia | 0,004978 | 0,007862 | 0,216878 | 0,886402 |
| XGBoost | 0,005115 | 0,008055 | 0,223616 | 0,880769 |
| ARIMAX | 0,005337 | 0,007901 | 0,232805 | 0,885277 |

En el horizonte exacto de 15 minutos, TFT redujo el MAE de persistencia en 4,62 %, XGBoost en 2,01 %, mientras que ARIMAX presentó un MAE 9,54 % superior.

El bootstrap pareado indicó que la ventaja global de TFT frente a persistencia fue pequeña y no concluyente. Por ello, los resultados se interpretan como evidencia de competitividad predictiva y no como demostración de superioridad estadística global.

## Requisitos

El experimento fue ejecutado utilizando:

- Python 3.8.20.
- Pandas 1.5.3.
- NumPy 1.24.4.
- Darts 0.31.0.
- PyTorch 2.4.1.
- XGBoost 2.1.4.
- Statsmodels 0.14.1.
- Scikit-learn 1.2.2.

Las dependencias pueden instalarse mediante:

```bash
pip install -r requirements.txt
