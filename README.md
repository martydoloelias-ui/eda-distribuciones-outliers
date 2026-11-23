# Análisis visual de distribuciones y outliers: producción del campo Volve

Notebook de Jupyter que caracteriza las variables de un dataset real de
producción petrolera: forma de las distribuciones (asimetría y curtosis),
comparación entre pozos con gráficos de barras, boxplots y violin plots,
y detección de outliers con la regla del rango intercuartílico (IQR).

## Origen de los datos

Producción diaria del campo petrolero Volve (Mar del Norte), publicada
por Equinor. Se usó la copia disponible en
[Kaggle](https://www.kaggle.com/datasets/nazarmahadialseied/volve-field-production-dataset-oil-and-gas)
y se guardó en `data/volve_produccion.csv` (la fecha se convirtió a
formato `AAAA-MM-DD`; no se modificó ningún otro valor). Revisá los
términos de uso en el sitio de Equinor antes de reutilizar los datos.

- 15.634 registros diarios de 7 pozos, entre 2007 y 2016, y 16 columnas.
- Variables categóricas: `NPD_WELL_BORE_NAME` (pozo) y `FLOW_KIND`
  (`production` o `injection`).
- Variables numéricas: petróleo, gas y agua producidos, horas de
  operación, presiones y temperaturas.
- El análisis se centra en las 9.161 filas de producción, porque los pozos
  inyectores no producen petróleo.

## Requisitos

- Python 3.9 o superior.
- Librerías: pandas, seaborn, matplotlib y notebook (`requirements.txt`).

## Cómo ejecutar el notebook

1. Clonar el repositorio y entrar a la carpeta:

   ```bash
   git clone <url-del-repo>
   cd <nombre-del-repo>
   ```

2. Instalar las dependencias:

   ```bash
   python -m pip install -r requirements.txt
   ```

3. Abrir Jupyter:

   ```bash
   python -m notebook
   ```

4. En el navegador, abrir `analisis_distribuciones_outliers.ipynb` y
   ejecutar todas las celdas (menú **Kernel > Restart & Run All**).

## Qué contiene el notebook

1. Carga y revisión de nulos.
2. Gráficos de barras de las variables categóricas (registros por pozo y
   tipo de flujo, y petróleo promedio por pozo).
3. Asimetría y curtosis de cinco variables numéricas, con histograma.
4. Boxplot y violin plot del petróleo diario por pozo.
5. Detección de outliers con IQR, de forma global y por pozo, y listado de
   los registros extremos.
6. Revisión de valores imposibles que el IQR no detecta.

## Resultados principales

**Forma de las distribuciones** (asimetría positiva = cola a la derecha):

| Variable | Asimetría | Curtosis (exceso) | Forma |
|---|---|---|---|
| Petróleo diario | 1,67 | 2,00 | cola a la derecha |
| Agua diaria | 0,52 | -1,19 | cola a la derecha (leve) |
| Horas de operación | -1,87 | 1,69 | cola a la izquierda |
| Presión en cabeza de pozo | 0,91 | 0,96 | cola a la derecha |
| Presión de fondo | -0,95 | -0,86 | cola a la izquierda |

**Outliers del petróleo diario (regla IQR):**
- Con un límite global (calculado con todos los pozos juntos): 986
  outliers (10,8%), todos de los pozos `15/9-F-12` y `15/9-F-14`.
- Con un límite por pozo: 105 outliers (1,1%), de los cuales 71 son picos
  altos y 34 son valores bajos. Los 34 bajos son todos del pozo
  `15/9-F-5` y 31 son días con producción 0.

## Decisiones sobre los outliers

| Hallazgo | ¿Es un error? | Decisión |
|---|---|---|
| Picos de petróleo (986 con límite global, 71 altos con límite por pozo) | No: son producciones altas reales, sobre todo de 2009 | Conservarlos. Detectarlos por pozo y no con un límite global. Si el modelo es sensible a la asimetría, transformar la variable (logaritmo) en vez de eliminar datos. |
| Más de 24 horas de operación (13 registros, todos en el último domingo de octubre) | No: ese día cambia la hora y dura 25 horas | Conservar y documentar, o limitar a 24 si el modelo asume un máximo diario. |
| Agua negativa (4 registros, hasta -458) | Sí: es físicamente imposible, y el IQR no lo detecta | Reemplazar por nulo e imputar (o por 0 si el valor es cercano a 0). |
| Días con 0 horas de operación (12,5% de los registros) | No: el pozo estaba cerrado | No eliminar. Marcarlos con una columna indicadora o analizar aparte los días operando. |

**Idea central:** un outlier no es automáticamente un dato malo. Antes de
eliminar un valor conviene preguntar si tiene una explicación física o de
negocio, y si el límite se calculó sobre un grupo homogéneo. Segmentar por
pozo bajó los outliers de 986 a 105 y mostró un patrón distinto (los días
en 0 de `15/9-F-5`).

## Estructura

```
.
├── analisis_distribuciones_outliers.ipynb   # análisis completo
├── data/volve_produccion.csv                # dataset
├── requirements.txt
├── .gitignore
└── README.md
```
