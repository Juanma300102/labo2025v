# Entrega final
 
> **Estudiante:** Pedrozo, Juan Martin \
> **Modalidad:** Senior \
> **Año:** 2025 \
> **Materia:** Laboratorio 1, comisión Virtual \
> **Universidad Austral**

## Experimentos

Luego de haber terminado la entrega de los experimentos colaborativos, procedi a trabajar sobre el script que mi grupo definio como la recomendación concreta, siendo este el workflow `src/workflows/experimentos_colab/ex4-950_WorkFlow_01_senior.ipynb.ipynb` ubicado en este repositorio.

Partiendo de esta base, comence a leer lo expuesto por mis compañeros en el [PPT de experimentos colaborativos](https://docs.google.com/presentation/d/1OfmXUu1QJ4IlposW_9rFo4jSJMD0gshT3o-_kEcy4vw/edit?slide=id.g2249244281f_0_126#slide=id.g2249244281f_0_126). En base a las recomendaciones de las cuales la mayoria era dejar el workflow como esta, modifiqué nuestro experimento base, convirtiendolo al workflow `final_submit/9510_WorkFlow_01_senior_suggestions.ipynb`.

Este experimento implementa especialmente la sugerencia del **Problema #05 grupo A**, cuya recomendacion concreta es:

- Incluir las variables generadas por el RF en el Feature Engineering Histórico
- Utilizar el método de ratios sobre las Exponential Moving Average de las variables
- Enlace al código correspondiente (sección 9.3.1.5 FEhist):  https://github.com/orcorsetti/labo2025v/blob/main/src/workflows/910_WorkFlow_01_junior_exp03_100193.ipynb

Dado que este experimento agregaba una **GRAN CANTIDAD DE COLUMNAS**, luego de unas 15 HS de entrenamiento, el experimento no habia culminado dicha generacion de columnas, con lo cual procedi a **descartar** este experimento por las limitaciones de tiempo dada la modalidad en la que curso.

Luego procedi a experimentar con un exp clave que nos quedo pendiente a mi equipo de experimentos colaborativos el cual consta de agregar a nuestra estrategia de pesos discretos por año, una consideración por los meses de pandemia los cuales representan una irregularidad en la actividad. Durante nuestros experimentos, observamos que eliminar dichos meses ayudaba a reducir la variación sin perder ganancia significativamente. [Click para ir al Slide del PPT de experimentos colaborativo relevante](https://docs.google.com/presentation/d/1OfmXUu1QJ4IlposW_9rFo4jSJMD0gshT3o-_kEcy4vw/edit?slide=id.g3a84afc2582_5_39#slide=id.g3a84afc2582_5_39).

Para esto, genere el script `final_submit/950_WorkFlow_01_senior_ex4-covid-1.ipynb` el cual implementa la estrategia de pesos de la siguiente manera

```R
PARAM$trainingstrategy$training <- c(
  201901, 201902, 201903, 201904, 201905, 201906,
  201907, 201908, 201909, 201910, 201911, 201912,
  202001, 202002, 202003, 202004, 202005, 202006,
  202007, 202008, 202009, 202010, 202011, 202012,
  202101, 202102, 202103, 202104, 202105
)

PARAM$trainingstrategy$covid <- c(
  202003, 202004, 202005, 202006,
  202007, 202008, 202009, 202010, 202011
)

# ========================================
# ESTRATEGIA DE PESOS POR AÑO
# ========================================
# Extraer el año de foto_mes
dataset[, anio := as.integer(substr(as.character(foto_mes), 1, 4))]

# Obtener años únicos ordenados (del más viejo al más reciente)
anios_unicos <- sort(unique(dataset$anio))

# Asignar pesos: año más viejo = 1, siguiente = 2, etc.
dataset[, peso_norm := match(anio, anios_unicos)]

# ========================================
# ESTRATEGIA DE PESOS POR MESES DE COVID
# ========================================
# Reducir el peso de los meses COVID (darles MENOR importancia)
# dataset[foto_mes %in% PARAM$trainingstrategy$covid, peso_norm := peso_norm * 0.5]
# O directamente un peso muy bajo:
dataset[foto_mes %in% PARAM$trainingstrategy$covid, peso_norm := 0.2]
```

El resultado de esta implementacion fue erroneo ya que al asignar el peso en `0.2`, cuando luego se verificaba la distribución de pesos, estos aparecian como `0`.

Por lo cual procedi a modificar esta implementacion, creando el notebook `950_WorkFlow_01_senior_ex4-covid-2.ipynb` donde los pesos se asignan de la siguiente manera

```R
PARAM$trainingstrategy$training <- c(
  201901, 201902, 201903, 201904, 201905, 201906,
  201907, 201908, 201909, 201910, 201911, 201912,
  202001, 202002, 202003, 202004, 202005, 202006,
  202007, 202008, 202009, 202010, 202011, 202012,
  202101, 202102, 202103, 202104, 202105
)

PARAM$trainingstrategy$covid <- c(
  202003, 202004, 202005, 202006,
  202007, 202008, 202009, 202010, 202011
)

# ========================================
# ESTRATEGIA DE PESOS POR AÑO
# ========================================
# Extraer el año de foto_mes
dataset[, anio := as.integer(substr(as.character(foto_mes), 1, 4))]

# Obtener años únicos ordenados (del más viejo al más reciente)
anios_unicos <- sort(unique(dataset$anio))

# Asignar pesos: año más viejo = 1, siguiente = 2, etc.
dataset[, peso_norm := match(anio, anios_unicos)]

# ========================================
# ESTRATEGIA DE PESOS POR MESES DE COVID
# ========================================
# Reducir el peso de los meses COVID (darles MENOR importancia)
# dataset[foto_mes %in% PARAM$trainingstrategy$covid, peso_norm := peso_norm * 0.5]
# O directamente un peso muy bajo:

# Pesos originales: 1, 2, 3 para años 2019, 2020, 2021
# Multiplicar todos por 10 para tener más granularidad
dataset[, peso_norm := peso_norm * 10]  # Ahora: 10, 20, 30

dataset[foto_mes %in% PARAM$trainingstrategy$covid, peso_norm := 5]
```

En esta variacion, hice dos experimentos donde el primero, constaba de asignar a los meses de covid el peso `1`. Este experimento obtuvo una reduccion significativa de la ganancia. Con lo que el experimento final fue correrlo con un peso de `5`.

De esta manera produje el final submit que he elegido.

**Aclaracion:** Los scripts fueron movidos de la carpeta `src` a la carpeta `final_submit` para la entrega, y renombrados a sus nombres actuales. Ademas, ciertas modificaciones fueron realizadas en el mismo archivo y mantenidas a traves de commits de GIT.

En esta carpeta se encuentra el archivo `exp-summary.csv` el cual recopila los resultados de los experimentos:

- Implementación de pesos discretos base `KA9500-ex4-pesos_s40`
- Implementación de pesos discretos + meses de covid con peso = 1 `KA9500-ex4-pesos-covid_s40`
- Implementación de pesos discretos + meses de covid con peso = 5 `KA9500-ex4-pesos-covid-2_s40`

## Conclusión

En el archivo `exp-summary.csv` se aprecia que el ultimo experimento que aplica un peso especial para los meses de covid fue el que mayor ganancia promedio obtuvo y con la menor desviación estándar.

Para intentar escapar de la tenebrosa maldición del ganador he elegido el submit mas cercano al promedio de ganancia (`63673.67`) y a su vez cercano a la mayor cantidad de envios el cual corresponde al submit:

- **KA9500-ex4-pesos-covid-2_s40_10750.csv** del script `final_submit/950_WorkFlow_01_senior_ex4-covid-2.ipynb`

## Instructivo para correr

El script final `950_WorkFlow_01_senior_ex4-covid-2.ipynb`  tiene una duración de aproximadamente 12h en una instance de **8vCPU y 128gb ram no Spot.**

La unica modificación significativa es que al momento de crear la instancia desde el template, **se cambió el tipo de provisionamiento de Spot a Standard** para no tener inconvenientes durante los ultimos entrenamientos debido al corto tiempo disponible.