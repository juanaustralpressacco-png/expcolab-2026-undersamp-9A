# expcolab-2026-undersamp-9A

TS 9A Estabilidad del modelo predictivo frente al Undersampling

Experimento colaborativo #09, Grupo A: **undersampling de CONTINUA**.
Laboratorio de Implementación I, Maestría en Ciencia de Datos, Universidad Austral, 2026.

Integrantes: Juan Pressacco · Alexandra Cremydas

> Estado: corrido. Video entregado. El notebook de este repo es el motor; no cronometra la grilla.

## Hipótesis

El recorte de CONTINUA está en el z620 solamente por velocidad. Buscar los hiperparámetros con una fracción de CONTINUA al azar, y todas las bajas, se compara contra el control (`training_pct` = 1), que es el que ve toda la CONTINUA.

## Diseño

Base: workflow `z620`, Analista Junior, Grupo A. `param_final` se arma con `modifyList(fijos, mejores_hiperparametros)`. Con `c()` se duplican num_leaves, min_data_in_leaf y feature_fraction y LightGBM se queda con el fijo.

| Aspecto | Nivel | Qué es |
|---|---|---|
| training_pct | 1 | Control. Entra toda la CONTINUA |
| | 0,5 | Esa fracción de CONTINUA al azar, y todas las bajas |
| | 0,25 | Igual, con 0,25 de la CONTINUA |
| | 0,1 | Igual, con 0,1 de la CONTINUA |

El recorte entra solo en la búsqueda (`fold_train`). Validación y entrenamiento final completos.

Períodos, iguales en los cuatro niveles: training 201901–202105, validación 202107, entrenamiento final 201901–202107, future 202109.

El grupo B repite el mismo experimento con futuro 202107 y ganancia local.

### Las 40 corridas

Cada integrante corre sus 5 semillas. Las mismas 10 en los cuatro porcentajes. La misma posición del bloque es la misma semilla.

| Porcentaje | Alexandra | Juan |
|---|---|---|
| 1 | 91xx | 96xx |
| 0,5 | 92xx | 97xx |
| 0,25 | 93xx | 98xx |
| 0,1 | 94xx | 99xx |

Semillas Juan, en orden 01–05: 810013, 433229, 920021, 300017, 666667.
Semillas Alexandra, en orden 13–17: 102191, 241109, 671749, 789047, 912239.

En cada corrida se vuelve a hacer el grid search completo del z620, los 50 modelos. No se reutiliza la receta de otro porcentaje.

Corridas extra (0,05; reloj de la grilla; más semillas) no están en este notebook y no entran al Wilcoxon de los cuatro niveles.

### Protocolo

1. Grid search completo en cada corrida: los 50 modelos del z620 (`CJ`, no bayesiana).
2. Cada integrante corre sus 5 semillas primigenias. Las mismas 10 en los cuatro porcentajes.
3. Cortes de partida 1800 a 2400 de a 100. El número de cada semilla es el **Public Score del corte donde pica**, no el promedio de esos siete. Si el máximo cae en el borde de la ventana, se sube un corte para afuera con `prediccion.txt`, sin reentrenar, hasta que la curva baje.
4. Comparación: Wilcoxon pareado, exacto, two-sided, n=10, cada recorte contra el control. No se declara ganador por el promedio de las diez.
5. Si el óptimo de una perilla queda en el borde de la grilla, se amplía el rango y se vuelve a correr esa grilla.

## Estructura del repositorio

```
(raíz)            z620_exp09_grupoA.ipynb: la copia del z620.
                  El original de la cátedra no se pisa.
                  Antes de cada corrida se cambian semilla_primigenia,
                  experimento y training_pct.
```

Este notebook no mide el wall-clock de la grilla. Hay `Sys.time()` de arranque y cierre, como en el z620; no hay `t_inicio` / `t_fin` alrededor de `Estimar_AUC_lightgbm`.

Las salidas de cada corrida quedan en la VM, en `/content/buckets/b1/exp/WF<código>/`. No se suben acá.

## Archivos para Kaggle

No se suben al repositorio. Quedan en la VM, en `/content/buckets/b1/exp/WF<código>/kaggle/`.

Nombre: `KA<código>_<envíos>.csv`, por ejemplo `KA9601_1800.csv`. En Kaggle, la descripción de cada envío dice `envios=<corte> semilla=<semilla>`. Competencia junior: `labo-1-virtual-2026-analista-jr`.

## Cómo replicar

1. Dataset junior de la competencia, en la VM del curso (`analistajr_competencia_2026.csv.gz`).
2. Abrir `z620_exp09_grupoA.ipynb`. Cambiar solo tres cosas: `PARAM$semilla_primigenia`, `PARAM$experimento` y `PARAM$trainingstrategy$training_pct`.
3. Correr de punta a punta. La grilla elige por AUC en 202107. El modelo de producción se entrena con todos, vía `modifyList`.
4. Subir los csv de la ventana. Anotar el Public del pico. Si pica el borde, un corte afuera.
5. Armar los cuatro vectores de 10 y el Wilcoxon pareado contra `training_pct` = 1.

